## 1. Slide 1: Tiêu đề và Giới thiệu tổng quan

**Lời dẫn:** Kính thưa Hội đồng và các đồng nghiệp, trong kỷ nguyên AI tạo sinh hiện nay, các hệ thống Retrieval-Augmented Generation (RAG) và Large Language Models (LLM) đang chạm phải một rào cản vật lý khắc nghiệt: "Bức tường bộ nhớ" (Memory Wall) và điểm nghẽn tính toán (Compute Bottleneck). Khi quy mô ngữ cảnh và lưu lượng truy vấn bùng nổ, tầng backend không chỉ đơn thuần là nơi lưu trữ, mà đã trở thành "tử huyệt" quyết định khả năng mở rộng và tính kinh tế của toàn bộ hệ thống. Đề tài "Ứng dụng học tăng cường cho thay thế cache động trong backend RAG/LLM" mà tôi trình bày hôm nay sẽ giải quyết bài toán tối ưu hóa hạ tầng này từ góc độ chiến lược: Chuyển dịch từ các cơ chế tĩnh sang một hệ thống tự thích nghi thông minh, đảm bảo hiệu suất tối đa trong môi trường tài nguyên hạn chế.

---

## 2. Slide 2: Bối cảnh thực tiễn và Áp lực hệ thống

**Lời dẫn:** Trong kiến trúc RAG hiện đại, khái niệm "cache" đã tiến hóa thành một hệ sinh thái dữ liệu đa tầng phức tạp, từ Embedding, Document Chunks đến KV-cache suy luận của LLM. Áp lực lên GPU và RAM là cực kỳ lớn do dung lượng bộ nhớ tỷ lệ thuận với độ dài ngữ cảnh.

Thực tế cho thấy, các thuật toán Heuristic truyền thống như LRU hay LFU, dù là tiêu chuẩn vàng từ thập niên 70, hiện đang hoàn toàn "mù lòa" trước các đặc trưng ngữ nghĩa. Trong RAG, hai truy vấn có thể khác nhau về mặt ký tự nhưng tương đồng về bản chất thông tin; việc loại bỏ một mục cache chỉ dựa trên tần suất truy cập mà bỏ qua giá trị ngữ nghĩa dẫn đến việc lãng phí tài nguyên để tái tạo dữ liệu không cần thiết. Điều này tạo ra một nghịch lý: chúng ta đang sử dụng AI tiên tiến nhất để tạo nội dung, nhưng lại dùng các quy tắc cũ kỹ để quản lý bộ nhớ của nó. Tại sao không dùng chính AI để quản lý AI? Câu trả lời nằm ở Học tăng cường tại Slide 3.

---

## 3. Slide 3: Vấn đề nghiên cứu và Sự phù hợp của Học tăng cường (RL)

**Lời dẫn:** Tại sao Reinforcement Learning (RL) lại là "vũ khí" tối ưu thay vì các mô hình học máy tĩnh? Bởi lẽ thay thế cache bản chất là một chuỗi các quyết định tuần tự, nơi một hành động loại bỏ (evict) hiện tại sẽ trực tiếp ảnh hưởng đến khả năng hit/miss trong tương lai—đây chính là đặc điểm cốt lõi của Quy trình quyết định Markov (MDP).

Học tăng cường cho phép chúng ta giải quyết bài toán đánh đổi giữa Khám phá và Khai thác (Exploration-Exploitation trade-off). Tác nhân RL có thể chấp nhận rủi ro loại bỏ một mục cache cũ để "thử nghiệm" lưu trữ một mục mới có tiềm năng mang lại giá trị lâu dài hơn. Bằng cách tích hợp "lớp tín hiệu" đa chiều bao gồm ngữ nghĩa (embedding similarity), tải hệ thống và chi phí tái tạo, RL biến bộ điều khiển cache từ một thực thể thụ động thành một hệ thống điều khiển có khả năng tự tiến hóa theo workload thực tế.

---

## 4. Slide 4: Mục tiêu nghiên cứu và Mô hình hóa MDP

**Lời dẫn:** Mục tiêu trọng tâm của nghiên cứu này là toán học hóa một vấn đề hạ tầng thành một mô hình MDP chặt chẽ thông qua 4 trụ cột:

1. **Mô hình hóa MDP:** Định nghĩa chính xác môi trường cache backend như một không gian trạng thái động.
2. **Thiết kế trạng thái (State Space):** Kết hợp linh hoạt giữa metadata truyền thống và các đặc trưng ngữ nghĩa (semantic features).
3. **Huấn luyện tác nhân:** Xây dựng chính sách quyết định (Policy) tối ưu hóa việc giữ lại hoặc loại bỏ dữ liệu.
4. **Đánh giá hiệu năng:** Chứng minh giá trị thực tiễn thông qua các workload hội thoại thực tế.

Việc coi tác nhân RL là một "bộ điều khiển trung tâm" sẽ giúp hệ thống tự thích nghi với những thay đổi bất thường của người dùng mà không cần sự can thiệp thủ công của kỹ sư hệ thống.

---

## 5. Slide 5: Câu hỏi nghiên cứu (Research Questions)

**Lời dẫn:** Để định hướng cho nghiên cứu, tôi đặt ra 4 câu hỏi then chốt nhằm đo lường sự thành công:

- **RQ1 (Hiệu quả):** RL có thực sự vượt qua các baseline công nghiệp như LRU/LFU về cả Hit rate và Latency trong môi trường RAG không?
- **RQ2 (Tầm quan trọng đặc trưng):** Tín hiệu nào—giữa recency, tương đồng ngữ nghĩa hay tải hệ thống—đóng vai trò quan trọng nhất trong việc ra quyết định?
- **RQ3 (Thiết kế hàm thưởng):** Làm thế nào để cân bằng giữa Hit rate và Latency p99? Đây là thách thức lớn nhất vì tối ưu hit rate đôi khi dẫn đến chi phí tính toán cao cho chính tác nhân RL.
- **RQ4 (Tính tổng quát):** Chính sách học được trên một miền dữ liệu có thể "sống sót" khi đối mặt với workload hoàn toàn mới không?

---

## 6. Slide 6: Tổng quan tài liệu (Literature Review Map)

**Lời dẫn:** Qua rà soát hệ thống các công trình từ 2023 đến 2026, tôi phân loại bức tranh công nghệ hiện nay thành 4 nhóm chiến lược:

|Nhóm tiếp cận|Công trình tiêu biểu|Mục tiêu chính|
|---|---|---|
|**Cache LLM/RAG Ngữ nghĩa**|ACC, DynamicRAG, RAC|Tối ưu dựa trên tương đồng ngữ nghĩa và "context anchors".|
|**Nén KV-cache LLM**|CompilerKV, HeadsMatter|Tối ưu GPU thông qua nén token và attention heads quan trọng.|
|**Cache Hệ thống**|CHROME, RL Multicore|Quản lý tầng thấp (LLC), MESI states và nạp trước.|
|**Điều phối Cloud-native**|ISCC, MARL Orchestration|Tối ưu tài nguyên đa tác nhân cho hạ tầng phân tán.|

Xu hướng rõ rệt là sự chuyển dịch từ tối ưu hóa phần cứng thuần túy sang việc nhận thức sâu sắc nội dung ngữ nghĩa của LLM.

---

## 7. Slide 7: Nhóm 1 - LLM/RAG và Cache Ngữ nghĩa (ACC, DynamicRAG, RAC)

**Lời dẫn:** Trong nhóm nghiên cứu về RAG, khung **ACC (2025)** đã đặt ra một tiêu chuẩn mới khi sử dụng Deep RL cho thiết bị biên, đạt tỷ lệ Hit rate ấn tượng trên 80%, đồng thời giảm độ trễ truy xuất **40%** và chi phí bộ nhớ **55%**. Sự đột phá này đến từ việc RL có khả năng nhận diện các "context anchors" trong văn bản—như cách tiếp cận của **RAC (2026)**—để bảo tồn các thực thể quan trọng, giúp hiệu quả vượt qua các phương pháp truyền thống từ 20% đến 30%. Điều này minh chứng rằng việc hiểu ngữ cảnh là chìa khóa để quản lý bộ nhớ hiệu quả.

---

## 8. Slide 8: Nhóm 2 - Nén và Quản lý KV-cache LLM (CompilerKV, HeadsMatter)

**Lời dẫn:** Tại tầng GPU, quản lý KV-cache là bài toán sống còn để chống lỗi tràn bộ nhớ (OOM). **CompilerKV (2026)** đã giới thiệu một hướng đi táo bạo: sử dụng **Offline RL với thuật toán Conservative Q-Learning (CQL)**. Tại sao lại là Offline RL? Bởi trong môi trường sản xuất, chúng ta không thể chấp nhận rủi ro từ việc tác nhân "thử sai" online. CQL cho phép huấn luyện chính sách an toàn từ dữ liệu lịch sử, giúp phục vụ batch-16 ổn định trong khi các phương pháp thông thường sẽ thất bại. Kết hợp với **HeadsMatter (2025)**, việc dùng RL để xác định các attention heads quan trọng cho phép nén 20-50% dữ liệu mà vẫn bảo toàn khả năng suy luận logic (CoT).

---

## 9. Slide 9: Nhóm 3 & 4 - RL cho Hệ thống và Điều phối Backend (CHROME, ISCC, MARL)

**Lời dẫn:** Ở tầng hệ thống, chúng ta kế thừa những bài học kinh điển từ **CHROME (2024)**, nơi thuật toán SARSA và mô hình **C-AMAT (Concurrent Average Memory Access Time)** được dùng để kết nối việc thay thế và nạp trước cache, tăng hiệu suất 13.7%. Trong lĩnh vực mạng IIoT, nghiên cứu **ISCC (2025)** đã chứng minh sức mạnh của nhận thức ngữ nghĩa khi tăng Hit rate thêm **28%** và giảm số lần thay thế cache **23%**. Đây là tiền đề để chúng ta mở rộng sang mô hình đa tác nhân (MARL), nơi các node cache cộng tác để đạt tỷ lệ sử dụng tài nguyên tới gần 90%.

---

## 10. Slide 10: Khoảng trống nghiên cứu (Research Gaps)

**Lời dẫn:** Dù đạt nhiều tiến bộ, nhưng bức tranh hiện tại vẫn còn những "khoảng trống" mà luận văn này sẽ lấp đầy:

- **Thiếu tính thống nhất:** Các nghiên cứu hiện tại thường chỉ tập trung vào một lớp duy nhất (như KV-cache). Luận văn của tôi đề xuất một mô hình **Cache phân cấp (Hierarchical Caching)**, quản lý đồng bộ từ Vector results, Document chunks đến Prompt prefix.
- **Thiếu mô hình End-to-end:** Chưa có một controller nào tích hợp đồng thời giữa ngữ nghĩa, chi phí tái tạo và tải hệ thống thực tế.
- **Workload thực tế:** Các phương pháp hiện có chưa được kiểm chứng mạnh mẽ trên các hội thoại dài và các truy vấn diễn đạt lại (paraphrased queries).

Đây chính là cơ hội để chúng ta xây dựng một middleware toàn diện hơn.

---

## 11. Slide 11: Hướng nghiên cứu đề xuất (Proposed Methodology)

**Lời dẫn:** Tôi đề xuất một giải pháp middleware thông minh với cấu trúc MDP được thiết kế đặc thù cho RAG. Trái tim của giải pháp là hàm thưởng đa mục tiêu tối ưu hóa toàn diện hiệu năng và chất lượng:

R = \alpha \cdot \text{Hit} - \beta \cdot \text{Latency} - \gamma \cdot \text{Memory} + \delta \cdot \text{Quality}

Trong đó, thành phần \delta \cdot \text{Quality} là một đóng góp mới, đại diện cho tính xác thực (Factuality) của câu trả lời RAG. Hệ thống sẽ không chỉ chạy đua theo tỷ lệ hit rate đơn thuần, mà phải đảm bảo rằng việc giữ lại dữ liệu trong cache thực sự đóng góp vào chất lượng đầu ra của AI. Tác nhân sẽ học cách loại bỏ (evict) hoặc bỏ qua (bypass) các mục cache dựa trên embedding truy vấn và tải trọng hệ thống thời gian thực.

---

## 12. Slide 12: Hướng triển khai và Metric đánh giá

**Lời dẫn:** Về mặt triển khai, middleware này sẽ nằm chiến lược giữa API Gateway và LLM Service. Chúng ta sẽ áp dụng quy trình: Huấn luyện Offline bằng dữ liệu trace thực tế để đảm bảo tính an toàn, sau đó mới triển khai Online với các chính sách kiểm soát rủi ro.

Bộ chỉ số đánh giá sẽ bao quát 3 khía cạnh:

1. **Hệ thống:** Hit rate ngữ nghĩa, độ trễ p95/p99 và throughput (QPS).
2. **Hạ tầng:** Số lần thay thế cache và mức tiêu thụ GPU/RAM.
3. **Chất lượng AI:** Điểm số xác thực (Factuality score) và F1 để đảm bảo tính toàn vẹn của nội dung.

---

## 13. Slide 13: Kết luận và Thông điệp chính

**Lời dẫn:** Kính thưa Hội đồng, quá trình rà soát tài liệu đã khẳng định: RL không còn là một lựa chọn, mà là hướng đi tất yếu. Chúng ta đang tiến tới kỷ nguyên **"Software 2.0"** hay **"AI-driven Infrastructure"**—nơi hạ tầng không còn được xây dựng bằng những dòng lệnh If-Else cứng nhắc, mà được vận hành bởi những bộ điều khiển thông minh biết học hỏi.

Thông điệp cuối cùng tôi muốn khẳng định: **"Thay thế cache cho RAG/LLM không còn là bài toán lưu trữ đơn thuần, mà là bài toán điều khiển thích nghi."** Đề tài này cam kết sẽ hiện thực hóa tầm nhìn đó bằng một giải pháp middleware nhẹ, nhận thức ngữ nghĩa và tối ưu đa mục tiêu.

Xin chân thành cảm ơn Hội đồng đã lắng nghe và tôi rất sẵn lòng nhận các câu hỏi thảo luận.