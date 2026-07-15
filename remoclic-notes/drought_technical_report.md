# Báo cáo Kỹ thuật: Hệ thống Dự báo Hạn hán

---

## 1. Kiến trúc Tổng quan

Hệ thống backend được xây dựng trên **FastAPI**, áp dụng kiến trúc phân lớp rõ ràng gồm 4 thành phần chính:

| Lớp | Module | Vai trò |
|---|---|---|
| **Router** | `app/api/routes/drought.py` | Tiếp nhận HTTP request, xác thực tham số đầu vào |
| **Service** | `app/services/drought_forecast_service.py` | Xử lý logic nghiệp vụ, tổng hợp kết quả |
| **Repository** | `app/repositories/drought_forecast_repository.py` | Truy vấn CSDL qua SQLAlchemy ORM |
| **Database** | PostgreSQL | Lưu trữ dữ liệu dự báo dạng bảng quan hệ |

Song song với luồng phục vụ HTTP request, hệ thống duy trì một **pipeline ingestion** chạy nền tự động mỗi 24 giờ, được điều phối bởi **APScheduler**. Pipeline này thu thập file NetCDF từ nguồn dữ liệu từ xa, bóc tách và nạp vào PostgreSQL.

### Cơ sở dữ liệu

Hai bảng chính phục vụ cho chức năng dự báo hạn hán:

```python
# app/models/drought_forecast.py
class DroughtForecast(Base):
    __tablename__ = "drought_forecasts"

    id        = Column(Integer, primary_key=True)
    ref_date  = Column(Date, nullable=False, index=True)  # Tháng phát hành dự báo
    lat       = Column(Float, nullable=False)              # Vĩ độ điểm lưới
    lon       = Column(Float, nullable=False)              # Kinh độ điểm lưới
    timescale = Column(Float, nullable=False)              # Chuỗi thời gian (SPI-1, SPI-3, ...)
    lead      = Column(Integer, nullable=False)            # Bước dự báo (tháng tiếp theo)
    mild      = Column(Float)   # Xác suất hạn nhẹ (%)
    mord      = Column(Float)   # Xác suất hạn vừa (%)
    seve      = Column(Float)   # Xác suất hạn nặng (%)
    dr_ens    = Column(Float)   # Chỉ số tổng hợp dự báo sự kiện (z-score)

    __table_args__ = (
        Index("idx_drought_forecast_coords", "lat", "lon"),
        Index("idx_drought_forecast_query", "lat", "lon", "ref_date"),
    )
```

```python
# app/models/drought_ref_date.py
class DroughtRefDate(Base):
    __tablename__ = "drought_ref_date"

    id        = Column(Integer, primary_key=True)
    ref_date  = Column(Date, nullable=False, unique=True, index=True)
    is_active = Column(Boolean, nullable=False, default=True)  # Bật/tắt hiển thị API
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
```

> [!NOTE]
> Bảng `drought_ref_date` đóng vai trò **công tắc** — cho phép admin ẩn một tháng dữ liệu khỏi API mà không cần xóa khỏi CSDL. Nếu một `ref_date` không có bản ghi trong bảng này, mặc định nó được coi là `active`.

---

## 2. Luồng Crawl Dữ liệu (Data Ingestion)

### 2.1 Khởi động Scheduler

Khi ứng dụng FastAPI khởi động, hàm `lifespan()` trong [`main.py`](file:///home/mylo2202/remoclic-v2-be/app/main.py) đăng ký hai loại job với APScheduler:

```python
# app/main.py
@asynccontextmanager
async def lifespan(fastapi_app: FastAPI):
    init_db()  # Tạo bảng nếu chưa tồn tại

    scheduler = BackgroundScheduler()

    # Job định kỳ: chạy mỗi 24 giờ
    scheduler.add_job(run_drought_ingestion, 'interval', days=1,
                      id='drought_ingestion_task')

    # Job tức thì: chạy ngay khi app khởi động để bắt kịp dữ liệu mới nhất
    scheduler.add_job(run_drought_ingestion, 'date', run_date=datetime.now(),
                      id='drought_ingestion_startup')

    scheduler.start()
    yield
    scheduler.shutdown()
```

- `'interval', days=1` — APScheduler lập lịch tự động gọi lại hàm mỗi 24 giờ.
- `'date', run_date=datetime.now()` — đảm bảo pipeline chạy ngay khi service restart, tránh bỏ lỡ chu kỳ cập nhật.

---

### 2.2 Scan thư mục từ xa & xác định dữ liệu còn thiếu

Hàm `run_drought_ingestion()` trong [`drought_ingestion_service.py`](file:///home/mylo2202/remoclic-v2-be/app/services/drought_ingestion_service.py) thực hiện bước đầu tiên: quét danh sách thư mục trên remote HTTP server.

```python
# app/services/drought_ingestion_service.py
def run_drought_ingestion():
    db = SessionLocal()

    base_url = settings.DROUGHT_DATA_URL.rstrip('/')

    # Bước 1: GET trang listing, tìm tất cả thư mục dạng YYYYMM/
    response = requests.get(base_url, timeout=15)
    response.raise_for_status()
    all_matches = re.findall(r'(\d{6})/', response.text)
    subdirs = list(dict.fromkeys(all_matches))  # Loại bỏ trùng lặp, giữ thứ tự

    # Bước 2: Lấy danh sách ref_date đã có trong DB
    existing_dates_query = db.execute(
        select(DroughtForecast.ref_date.distinct())
    ).scalars().all()
    existing_dates = {d.strftime("%Y%m") for d in existing_dates_query}

    # Bước 3: Xác định thư mục còn thiếu
    missing_subdirs = [s for s in subdirs if s not in existing_dates]

    for subdir in sorted(missing_subdirs):
        file_url = f"{base_url}/{subdir}/{settings.DROUGHT_DATA_FILE_NAME}"
        try:
            ingest_drought_file(db, file_url, subdir)
        except Exception as e:
            db.rollback()
            logger.error("Failed to ingest %s: %s", file_url, e)
```

- `re.findall(r'(\d{6})/', response.text)` — parse HTML listing, tìm mọi chuỗi 6 chữ số (định dạng `YYYYMM`) theo sau là `/`.
- So sánh tập hợp (`set`) để xác định thư mục nào chưa được nạp, tránh download lại dữ liệu cũ.
- Nếu một `subdir` lỗi, `db.rollback()` cô lập lỗi đó, vòng lặp tiếp tục với `subdir` tiếp theo.

---

### 2.3 Download & Parse file NetCDF

Với mỗi thư mục còn thiếu, `ingest_drought_file()` thực hiện toàn bộ chu trình download → parse → insert:

```python
def ingest_drought_file(db: Session, url: str, subdir_name: str):
    # Tạo file tạm để tránh conflict khi chạy song song
    fd, temp_path = tempfile.mkstemp(suffix=".nc")
    os.close(fd)

    try:
        # Bước 1: Download file NetCDF về đĩa cục bộ
        urllib.request.urlretrieve(url, temp_path)

        # Bước 2: Mở dataset bằng xarray (decode_times=False vì NetCDF dùng
        # định dạng thời gian tùy chỉnh, không phải chuẩn CF)
        ds = xr.open_dataset(temp_path, decode_times=False)
        ds.load()   # Load toàn bộ vào memory trước khi đóng file
        ds.close()

        # Bước 3: Parse ngày tham chiếu từ metadata hoặc tên thư mục
        ref_date_attr = ds.attrs.get("ref_date")
        ref_date = parse_ref_date(ref_date_attr, subdir_name)

        # Bước 4: Xóa dữ liệu cũ của ref_date này (nếu chạy lại)
        db.execute(delete(DroughtForecast).where(DroughtForecast.ref_date == ref_date))
        db.commit()

        # Upsert trạng thái vào bảng drought_ref_date
        status = db.execute(
            select(DroughtRefDate).where(DroughtRefDate.ref_date == ref_date)
        ).scalars().one_or_none()
        if status is None:
            db.add(DroughtRefDate(ref_date=ref_date, is_active=True))
            db.commit()
    finally:
        if os.path.exists(temp_path):
            os.remove(temp_path)  # Luôn dọn file tạm dù có lỗi hay không
```

---

### 2.4 Bóc tách dữ liệu 4 chiều & Bulk Insert

File NetCDF chứa 4 biến dạng mảng 4 chiều `(timescale × lead × lat × lon)`. Hệ thống chuẩn hóa thứ tự chiều rồi duyệt toàn bộ để tạo records:

```python
        # Chuẩn hóa thứ tự chiều — tránh sai lệch nếu file NetCDF có chiều
        # không theo đúng thứ tự mặc định
        da_mild   = ds["mild"].transpose("timescale", "lead", "lat", "lon")
        da_mord   = ds["mord"].transpose("timescale", "lead", "lat", "lon")
        da_seve   = ds["seve"].transpose("timescale", "lead", "lat", "lon")
        da_dr_ens = ds["dr_ens"].transpose("timescale", "lead", "lat", "lon")

        records = []
        for t_idx, timescale in enumerate(ds["timescale"].values):
            for l_idx, lead in enumerate(ds["lead"].values):
                for lat_idx, lat in enumerate(ds["lat"].values):
                    for lon_idx, lon in enumerate(ds["lon"].values):
                        mild  = float(da_mild[t_idx, l_idx, lat_idx, lon_idx])
                        mord  = float(da_mord[t_idx, l_idx, lat_idx, lon_idx])
                        seve  = float(da_seve[t_idx, l_idx, lat_idx, lon_idx])
                        dr_ens = float(da_dr_ens[t_idx, l_idx, lat_idx, lon_idx])

                        # Lọc fill value: -99.0 là giá trị mặc định của NetCDF
                        # khi không có dữ liệu (trên biển, ngoài lãnh thổ, ...)
                        def clean(v):
                            return v if (v != -99.0 and not math.isnan(v)) else None

                        mild, mord, seve, dr_ens = clean(mild), clean(mord), clean(seve), clean(dr_ens)
                        if any(v is None for v in [mild, mord, seve, dr_ens]):
                            continue  # Bỏ qua điểm không có dữ liệu đầy đủ

                        records.append({
                            "ref_date": ref_date, "lat": float(lat), "lon": float(lon),
                            "timescale": float(timescale), "lead": int(lead),
                            "mild": mild, "mord": mord, "seve": seve, "dr_ens": dr_ens
                        })

        # Bulk insert theo batch 5.000 để cân bằng memory và số roundtrip tới DB
        batch_size = 5000
        for i in range(0, len(records), batch_size):
            db.bulk_insert_mappings(DroughtForecast, records[i:i + batch_size])
        db.commit()
```

> [!NOTE]
> File NetCDF dự báo hạn hán có thể chứa hàng trăm nghìn điểm lưới. Việc chia batch 5.000 records tránh tình trạng cạn kiệt memory và giảm thời gian lock bảng trong PostgreSQL.

---

## 3. Luồng Xử lý HTTP Request

### 3.1 Router: Tiếp nhận request

[`app/api/routes/drought.py`](file:///home/mylo2202/remoclic-v2-be/app/api/routes/drought.py) định nghĩa các endpoint. FastAPI tự động validate tham số query và truyền `service` qua Dependency Injection:

```python
# app/api/routes/drought.py
@router.get("/probability-forecast")
async def get_probability_forecast(
        lat: float = Query(...),        # Bắt buộc — vĩ độ điểm cần tra
        lng: float = Query(...),        # Bắt buộc — kinh độ điểm cần tra
        ref_date: str = Query(None),    # Tuỳ chọn — mặc định lấy tháng mới nhất
        timescale: float = Query(1.0),  # Tuỳ chọn — chuỗi thời gian SPI
        service: DroughtForecastService = Depends(get_drought_forecast_service),
):
    try:
        return service.get_probability_forecast(lat=lat, lng=lng,
                                                ref_date_str=ref_date,
                                                timescale=timescale)
    except ValueError as e:
        raise HTTPException(status_code=400, detail=str(e))
    except Exception:
        raise HTTPException(status_code=500, detail="Internal server error")
```

- `Depends(get_drought_forecast_service)` — FastAPI tự động resolve dependency chain, tạo DB session và inject vào Service trước khi hàm handler chạy.
- Phân tách rõ `ValueError` (lỗi do input không hợp lệ → HTTP 400) và `Exception` chung (lỗi hệ thống → HTTP 500).

---

### 3.2 Dependency Injection: Tạo Service

```python
# app/api/dependencies.py

# Bước 1: Tạo DB session từ connection pool
def get_drought_forecast_repository(
        db: Session = Depends(get_db)
) -> DroughtForecastRepository:
    return DroughtForecastRepository(db=db)

# Bước 2: Inject repository vào Service
def get_drought_forecast_service(
        repository: DroughtForecastRepository = Depends(get_drought_forecast_repository),
) -> DroughtForecastService:
    return DroughtForecastService(repository=repository)
```

FastAPI giải quyết chuỗi dependency theo thứ tự: `get_db` → `get_drought_forecast_repository` → `get_drought_forecast_service`. Mỗi request nhận một DB session riêng biệt, được đóng sau khi request hoàn tất.

---

### 3.3 Service: Xử lý Logic nghiệp vụ

[`DroughtForecastService`](file:///home/mylo2202/remoclic-v2-be/app/services/drought_forecast_service.py) là lớp trung tâm, thực hiện toàn bộ logic trước khi trả kết quả:

```python
# app/services/drought_forecast_service.py
class DroughtForecastService:
    def __init__(self, repository: DroughtForecastRepository):
        self.repository = repository

    def get_probability_forecast(self, lat, lng, ref_date_str=None, timescale=1.0):
        nearest_lat, nearest_lon, ref_date, labels, points = \
            self.get_drought_forecast_tuple(lat, lng, ref_date_str, timescale)

        return {
            "location": {"lat": nearest_lat, "lng": nearest_lon},
            "ref_date": ref_date,
            "timescale": timescale,
            "labels": labels,        # ["01-2026", "02-2026", ...]
            "data": {
                "mild": _clean_vals([p.mild for p in points]),
                "mord": _clean_vals([p.mord for p in points]),
                "seve": _clean_vals([p.seve for p in points]),
            },
        }
```

Hàm trung tâm `get_drought_forecast_tuple()` thực hiện theo trình tự:

```python
    def get_drought_forecast_tuple(self, lat, lng, ref_date_str=None, timescale=1.0):

        # 1. Parse hoặc lấy ref_date mặc định
        if ref_date_str:
            if "-" in ref_date_str:
                ref_date = datetime.strptime(ref_date_str, "%Y-%m-%d").date()
            else:
                ref_date = datetime.strptime(ref_date_str, "%Y%m").date()
        else:
            # Lấy tháng mới nhất đang active trong DB
            ref_date = self.repository.get_latest_ref_date()
            if not ref_date:
                raise ValueError("No forecast data available.")

        # 2. Kiểm tra ref_date có đang active không
        if not self.repository.is_ref_date_active(ref_date):
            raise ValueError(f"Reference date {ref_date} is temporarily unavailable.")

        # 3. Tìm điểm lưới gần nhất với toạ độ yêu cầu (nearest neighbor)
        coord = self.repository.find_nearest_grid_point(lat, lng)
        if not coord:
            raise ValueError("No grid coordinates found in database.")
        nearest_lat, nearest_lon = coord

        # 4. Lấy toàn bộ bước dự báo (lead) tại điểm lưới đó
        points = self.repository.get_forecast_points(
            lat=nearest_lat, lon=nearest_lon, ref_date=ref_date, timescale=timescale
        )

        # 5. Tạo nhãn ngày cho từng lead (ví dụ lead=1 → "02-2026")
        labels = _generate_date_labels(ref_date, [p.lead for p in points])
        return nearest_lat, nearest_lon, ref_date, labels, points
```

```python
def _generate_date_labels(ref_date: date, leads: list[int]) -> list[str]:
    """Tính tên tháng/năm ứng với từng bước dự báo (lead)."""
    labels = []
    for lead in leads:
        new_month = (ref_date.month + lead - 1) % 12 + 1
        new_year  = ref_date.year + (ref_date.month + lead - 1) // 12
        labels.append(f"{new_month:02d}-{new_year}")
    return labels

def _clean_vals(vals) -> list[float | None]:
    """Làm tròn 2 chữ số thập phân, chuyển fill value thành None."""
    return [round(float(v), 2) if (v is not None and v != -99.0) else None for v in vals]
```

---

### 3.4 Repository: Truy vấn CSDL

[`DroughtForecastRepository`](file:///home/mylo2202/remoclic-v2-be/app/repositories/drought_forecast_repository.py) đóng gói toàn bộ logic truy vấn SQLAlchemy, tách biệt hoàn toàn khỏi business logic:

**Lấy ref_date mới nhất đang active:**
```python
def get_latest_ref_date(self) -> Optional[date]:
    stmt = (
        select(DroughtForecast.ref_date)
        .join(DroughtRefDate, DroughtRefDate.ref_date == DroughtForecast.ref_date, isouter=True)
        .where((DroughtRefDate.is_active.is_(True)) | (DroughtRefDate.id.is_(None)))
        .distinct()
        .order_by(DroughtForecast.ref_date.desc())
        .limit(1)
    )
    return self.db.execute(stmt).scalar()
```

> `isouter=True` (LEFT JOIN) + `DroughtRefDate.id.is_(None)` — cho phép trả về `ref_date` ngay cả khi chưa có bản ghi trong `drought_ref_date`, mặc định xem là active.

**Tìm điểm lưới gần nhất (Manhattan distance):**
```python
def find_nearest_grid_point(self, lat: float, lon: float):
    stmt = (
        select(DroughtForecast.lat, DroughtForecast.lon)
        .order_by(func.abs(DroughtForecast.lat - lat) + func.abs(DroughtForecast.lon - lon))
        .limit(1)
    )
    return self.db.execute(stmt).first()
```

> Sử dụng **khoảng cách Manhattan** (`|Δlat| + |Δlon|`) thay vì Euclidean do lưới điểm dữ liệu đều và vuông góc — tính toán đơn giản hơn, hiệu quả tương đương trong phạm vi nhỏ.

**Lấy toàn bộ bước dự báo theo điểm lưới:**
```python
def get_forecast_points(self, lat, lon, ref_date, timescale=1.0):
    stmt = (
        select(DroughtForecast)
        .join(DroughtRefDate, DroughtRefDate.ref_date == DroughtForecast.ref_date, isouter=True)
        .where(
            DroughtForecast.lat == lat,
            DroughtForecast.lon == lon,
            DroughtForecast.ref_date == ref_date,
            DroughtForecast.timescale == timescale,
            (DroughtRefDate.is_active.is_(True)) | (DroughtRefDate.id.is_(None)),
        )
        .order_by(DroughtForecast.lead.asc())
    )
    return list(self.db.execute(stmt).scalars().all())
```

---

### 3.5 Response trả về Frontend

Kết quả cuối cùng của `GET /drought/probability-forecast` có dạng:

```json
{
  "location": { "lat": 15.5, "lng": 108.25 },
  "ref_date": "2026-03-01",
  "timescale": 1.0,
  "labels": ["04-2026", "05-2026", "06-2026", "07-2026", "08-2026", "09-2026"],
  "data": {
    "mild": [12.34, 15.20, 18.05, null, null, null],
    "mord": [8.10,  9.75, 11.20, null, null, null],
    "seve": [3.50,  4.10,  5.80, null, null, null]
  }
}
```

- `labels` — nhãn tháng/năm tương ứng với từng bước `lead`, được tính động từ `ref_date`.
- `null` — điểm không có dữ liệu (fill value sau khi làm sạch).
- `GET /drought/event-forecast` trả về cấu trúc tương tự nhưng trường `data` chỉ chứa `dr_ens` (chỉ số sự kiện hạn tổng hợp dạng z-score).
