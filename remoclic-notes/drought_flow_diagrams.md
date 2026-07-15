# Sơ đồ luồng hệ thống Dự báo Hạn hán

---

## 1. Luồng Crawl Dữ liệu (Data Ingestion)

> Tự động chạy **1 lần/ngày** (APScheduler `interval days=1`) và **1 lần ngay khi khởi động** (`date` job).

```mermaid
sequenceDiagram
    autonumber
    participant Scheduler as APScheduler<br/>(BackgroundScheduler)
    participant Ingestion as drought_ingestion_service<br/>run_drought_ingestion()
    participant RemoteServer as Remote HTTP Server<br/>(DROUGHT_DATA_URL)
    participant Parser as ingest_drought_file()<br/>+ xarray
    participant DB as PostgreSQL<br/>(drought_forecast / drought_ref_date)

    Note over Scheduler: App khởi động → lifespan()
    Scheduler->>Scheduler: add_job(interval=1 day)<br/>add_job(date=now) — chạy ngay

    loop Mỗi 24 giờ
        Scheduler->>Ingestion: Trigger run_drought_ingestion()
        Ingestion->>Ingestion: init_db() + SessionLocal()

        Ingestion->>RemoteServer: GET DROUGHT_DATA_URL<br/>(scan danh sách thư mục YYYYMM/)
        RemoteServer-->>Ingestion: HTML listing (regex → subdirs[])

        Ingestion->>DB: SELECT DISTINCT ref_date FROM drought_forecast
        DB-->>Ingestion: existing_dates (set)

        Ingestion->>Ingestion: missing_subdirs = subdirs − existing_dates

        alt Không có thư mục mới
            Ingestion->>Ingestion: Log "up-to-date", return
        else Có thư mục chưa được ingest
            loop Với mỗi subdir thiếu (YYYYMM)
                Ingestion->>Parser: ingest_drought_file(db, url, subdir)

                Parser->>RemoteServer: urllib.request.urlretrieve(url)<br/>→ download file .nc về temp file
                RemoteServer-->>Parser: NetCDF file (*.nc)

                Parser->>Parser: xr.open_dataset() → load dataset
                Parser->>Parser: parse_ref_date() từ ds.attrs hoặc subdir_name
                Parser->>Parser: Transpose dims: (timescale, lead, lat, lon)<br/>Extract: mild, mord, seve, dr_ens

                Parser->>DB: DELETE drought_forecast WHERE ref_date=X
                Parser->>DB: INSERT / upsert drought_ref_date (is_active=True)
                DB-->>Parser: OK

                Parser->>Parser: Build records[] (lọc NaN / -99.0)<br/>Batch 5.000 records/lần

                Parser->>DB: bulk_insert_mappings(DroughtForecast, batch)
                DB-->>Parser: commit OK

                Parser->>Parser: os.remove(temp_path)

                alt Lỗi bất kỳ
                    Parser-->>Ingestion: raise Exception
                    Ingestion->>DB: db.rollback()
                    Ingestion->>Ingestion: Log error, tiếp tục subdir kế tiếp
                end
            end
        end

        Ingestion->>DB: db.close()
    end
```

---

## 2. Luồng Xử lý HTTP Request (Dự báo Hạn hán)

> Frontend gọi API → FastAPI router → Service → Repository → PostgreSQL → trả JSON về Frontend.

```mermaid
sequenceDiagram
    autonumber
    participant FE as Frontend<br/>(Browser / Client)
    participant Router as FastAPI Router<br/>app/api/routes/drought.py
    participant Dep as Dependency Injection<br/>get_drought_forecast_service()
    participant Service as DroughtForecastService<br/>drought_forecast_service.py
    participant Repo as DroughtForecastRepository<br/>drought_forecast_repository.py
    participant DB as PostgreSQL<br/>(drought_forecast / drought_ref_date)

    FE->>Router: GET /drought/probability-forecast<br/>?lat=&lng=&ref_date=&timescale=

    Router->>Dep: Depends(get_drought_forecast_service)
    Dep->>Dep: SessionLocal() → new DB session
    Dep-->>Router: DroughtForecastService(repository)

    Router->>Service: service.get_probability_forecast(lat, lng, ref_date_str, timescale)

    Service->>Service: Parse / validate ref_date_str<br/>(YYYY-MM-DD hoặc YYYYMM)

    alt ref_date không truyền lên
        Service->>Repo: get_latest_ref_date()
        Repo->>DB: SELECT ref_date FROM drought_forecast<br/>JOIN drought_ref_date WHERE is_active=True<br/>ORDER BY ref_date DESC LIMIT 1
        DB-->>Repo: latest ref_date
        Repo-->>Service: ref_date
    end

    Service->>Repo: is_ref_date_active(ref_date)
    Repo->>DB: SELECT * FROM drought_ref_date WHERE ref_date=X
    DB-->>Repo: DroughtRefDate record (or None)
    Repo-->>Service: True / False

    alt ref_date không active
        Service-->>Router: raise ValueError
        Router-->>FE: HTTP 400 Bad Request
    end

    Service->>Repo: find_nearest_grid_point(lat, lng)
    Repo->>DB: SELECT lat, lon FROM drought_forecast<br/>ORDER BY ABS(lat-X) + ABS(lon-Y) LIMIT 1
    DB-->>Repo: (nearest_lat, nearest_lon)
    Repo-->>Service: coord tuple

    Service->>Repo: get_forecast_points(nearest_lat, nearest_lon, ref_date, timescale)
    Repo->>DB: SELECT * FROM drought_forecast<br/>JOIN drought_ref_date<br/>WHERE lat=X AND lon=Y AND ref_date=Z AND timescale=T<br/>AND is_active=True<br/>ORDER BY lead ASC
    DB-->>Repo: List[DroughtForecast]
    Repo-->>Service: points[]

    Service->>Service: _generate_date_labels(ref_date, leads[])<br/>_clean_vals() — làm tròn 2 chữ số, lọc None

    Service-->>Router: dict { location, ref_date, timescale, labels, data{mild,mord,seve} }

    Router-->>FE: HTTP 200 JSON Response

    Note over Router,FE: Tương tự cho GET /event-forecast (trả dr_ens)<br/>GET /ref-dates (trả danh sách ref_date active)<br/>POST /ref-dates/toggle (bật/tắt ref_date)
```

---

## Kiến trúc tổng quan

```mermaid
flowchart TB
    subgraph SCHEDULER["APScheduler - moi 24h"]
        S1["run_drought_ingestion()"]
    end

    subgraph INGESTION["Ingestion Pipeline"]
        I1["Scan remote URL\n(danh sach YYYYMM/)"]
        I2["So sanh voi DB\n(xac dinh missing)"]
        I3["Download .nc\n(urllib -> tempfile)"]
        I4["Parse NetCDF\n(xarray, transpose dims)"]
        I5["Build records\n(loc NaN va -99.0)"]
        I6["Bulk insert\n(batch 5000 rows)"]
    end

    subgraph REMOTE["Remote HTTP Server"]
        R1["HTML listing\n(cac thu muc YYYYMM)"]
        R2["NetCDF file (.nc)"]
    end

    subgraph HTTP["HTTP Request Pipeline"]
        H1["FastAPI Router\n(/drought/probability-forecast\n/drought/event-forecast)"]
        H2["Dependency Injection\n(DB Session)"]
        H3["DroughtForecastService\n(business logic)"]
        H4["DroughtForecastRepository\n(SQLAlchemy queries)"]
    end

    subgraph DB["PostgreSQL"]
        D1[("drought_forecast\nref_date, lat, lon\ntimescale, lead\nmild, mord, seve, dr_ens")]
        D2[("drought_ref_date\nref_date, is_active")]
    end

    FE["Frontend"] -->|"HTTP GET / POST"| H1

    SCHEDULER --> S1
    S1 --> I1
    I1 -->|GET html listing| R1
    R1 --> I2
    I2 -->|urlretrieve| R2
    R2 --> I3
    I3 --> I4
    I4 --> I5
    I5 --> I6
    I6 -->|"bulk insert + commit"| D1
    I6 -->|upsert| D2

    H1 --> H2
    H2 --> H3
    H3 --> H4
    H4 -->|SELECT| D1
    H4 -->|"SELECT / UPDATE"| D2
    H4 -->|List of records| H3
    H3 -->|JSON dict| H1
    H1 -->|"HTTP 200 JSON"| FE
```
