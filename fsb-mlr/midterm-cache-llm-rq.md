# Research Questions

### 1. Làm thế nào Học tăng cường (Reinforcement Learning) có thể tối ưu hóa việc quản lý bộ nhớ cache một cách thích ứng để vượt trội hơn các thuật toán tĩnh truyền thống?

Trọng tâm chính trong các bài báo này là giải quyết sự thất bại của các thuật toán truyền thống như **LRU (Least Recently Used)** và **LFU (Least Frequently Used)** trong việc xử lý khối lượng công việc động hoặc các phụ thuộc phức tạp.

* **Bộ nhớ cache theo ngữ cảnh thích ứng (Adaptive Contextual Caching - ACC):** Nghiên cứu việc sử dụng Học tăng cường sâu (Deep RL - DRL) để tinh chỉnh việc thay thế bộ nhớ cache cho các LLM biên trên thiết bị di động, cân bằng giữa mức độ liên quan của ngữ cảnh và chi phí do lỗi truy cập.

* **CHROME:** Khám phá một cách tiếp cận toàn diện bằng cách tối ưu hóa đồng thời việc thay thế bộ nhớ cache, bỏ qua và tìm nạp trước bằng cách sử dụng Học tăng cường trực tuyến trong các hệ thống đa lõi.

* **Stormbird:** Trả lời câu hỏi làm thế nào Học tăng cường có thể đưa ra các chính sách thay thế tiết kiệm chi phí bằng cách phân tích **các loại truy cập bộ nhớ cache** (như ghi lại) để giảm chi phí phần cứng.
* **Học tăng cường nhận biết tính nhất quán (Coherence-Aware RL):** Hỏi liệu việc tận dụng **trạng thái nhất quán bộ nhớ cache** (ví dụ: Đã sửa đổi, Đã chia sẻ) có thể hướng dẫn các quyết định loại bỏ tốt hơn trong bộ xử lý đa lõi hay không.

* **ISCC:** Giải quyết vấn đề cải thiện việc thay thế nội dung trong IoT công nghiệp (IIoT) bằng cách tích hợp Q-learning với các cấu trúc dữ liệu hiệu quả như Cuckoo Hashing.

### 2. Làm thế nào để có thể tận dụng nhận thức ngữ nghĩa và mối quan hệ yêu cầu để cải thiện độ chính xác của bộ nhớ cache cho các dịch vụ LLM?

Một số bài báo nghiên cứu cách vượt ra ngoài các mẫu truy cập dữ liệu riêng lẻ bằng cách kết hợp "ý nghĩa" hoặc "cấu trúc" của các yêu cầu.

* **RAC (Bộ nhớ cache nhận biết mối quan hệ - Relation-Aware Cache):** Trả lời câu hỏi về các mối quan hệ có thể quan sát được trực tuyến — cụ thể là **sự lặp lại theo chủ đề** và **sự phụ thuộc cấu trúc** — để bảo toàn các "neo ngữ cảnh" quan trọng.

* **CompilerKV:** Nghiên cứu việc sử dụng **tín hiệu rủi ro cấp độ lời nhắc** (entropy chú ý và độ phức tạp cục bộ) để điều chỉnh động ngưỡng nén bộ nhớ cache KV.
* **RLKV:** Hỏi những đầu chú ý cụ thể nào là cần thiết để duy trì tính nhất quán của **lý luận chuỗi suy nghĩ (CoT)**, cho phép nén mạnh các đầu chú ý không quan trọng.

* **ISCC:** Đề xuất một mô hình chấm điểm ngữ nghĩa (miền, độ tin cậy, mức độ quan trọng) để cung cấp thông tin cho bộ nhớ đệm nhận biết ngữ cảnh trong môi trường mạng.

### 3. Chất lượng phản hồi LLM và các tiên nghiệm kiến ​​trúc có thể đóng vai trò là phản hồi để tối ưu hóa hiệu quả truy xuất và bộ nhớ như thế nào?

Các bài báo này khám phá việc sử dụng hành vi hoặc đặc điểm nội bộ của chính mô hình làm tín hiệu giám sát để tối ưu hóa.

* **DynamicRAG:** Trả lời câu hỏi làm thế nào một bộ xếp hạng lại có thể điều chỉnh động số lượng tài liệu ($k$) cho một truy vấn bằng cách sử dụng **chất lượng phản hồi LLM** làm chỉ báo trực tiếp về mức độ liên quan của tài liệu.

* **CompilerKV:** Nghiên cứu cách các **tiên nghiệm kiến ​​trúc** ổn định về độ tin cậy của đầu chú ý có thể được biên dịch ngoại tuyến thành một chính sách xác định vẫn an toàn cho các quyết định điền trước một lần.
* **RLKV:** Sử dụng RL như một công cụ để quan sát cách nén các đầu riêng lẻ ảnh hưởng đến **kết quả suy luận thực tế** (độ chính xác của câu trả lời), xác định các đầu quan trọng cho suy luận tạo sinh.

### 4. Làm thế nào RL đa tác nhân và các cơ chế dựa trên vai trò có thể phối hợp tài nguyên trong các hệ thống không đồng nhất quy mô lớn?

Chủ đề nghiên cứu này tập trung vào sự phối hợp và điều phối trong các môi trường phân tán phức tạp.

* **Điều phối tài nguyên MARL:** Trả lời câu hỏi làm thế nào **các tác nhân dựa trên vai trò không đồng nhất** (đại diện cho các nút tính toán, lưu trữ và bộ lập lịch) có thể phối hợp việc học chính sách để cải thiện việc sử dụng tài nguyên trong các cụm điện toán đám mây.

* **CHROME:** Giải quyết việc tối ưu hóa chung các kỹ thuật bộ nhớ đệm riêng biệt, sử dụng RL để giảm thiểu sự can thiệp giữa các quyết định quản lý cạnh tranh.

* **ISCC:** Khám phá sự đánh đổi giữa hiệu quả cấu trúc và học tập thích ứng trong các nút IIoT bị hạn chế bộ nhớ.