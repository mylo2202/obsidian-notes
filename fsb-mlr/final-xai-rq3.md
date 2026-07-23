# XAI - Research Question 3

### RQ3: Làm thế nào để đánh giá định lượng và định tính tác động của các giải thích AI đối với mức độ tin cậy và hiệu quả ra quyết định của bác sĩ trong quy trình lâm sàng?

Vấn đề chính:

- Sự chấp nhận lâm sàng: Việc tạo ra giải thích là chưa đủ; hệ thống cần chứng minh được rằng những giải thích này thực sự giúp bác sĩ đưa ra quyết định tốt hơn hoặc tăng mức độ tin tưởng vào mô hình.
- Rủi ro về nhận thức: Các giải thích quá phức tạp có thể gây quá tải nhận thức, trong khi các giải thích quá đơn giản (như các mô hình surrogate) có thể làm mất đi độ chính xác của logic mô hình gốc (performance-interpretability trade-off).
- Đo lường sự tin cậy: Hiện thiếu các bộ chỉ số chuẩn để đo lường "niềm tin" của bác sĩ và tính "trung thực" (faithfulness) của giải thích so với chính sách Offline RL thực tế.

Các hướng đánh giá và tiêu chí phù hợp:

- Thử nghiệm Human-in-the-loop: So sánh hiệu quả quyết định của bác sĩ khi có và không có module XAI để đo lường sự cải thiện về độ chính xác và thời gian phản hồi.
- Chỉ số đánh giá sự hài lòng của người dùng (User Satisfaction): Sử dụng các thang đo Likert hoặc khảo sát định tính để đánh giá tính hữu ích và khả năng dự đoán hành vi mô hình của bác sĩ.
- Tính trung thực (Explanation Faithfulness): Đảm bảo giải thích phản ánh đúng trọng số và logic mà tác nhân Offline RL đã sử dụng để đưa ra hành động (ví dụ: đối chiếu Attention weights với kiến thức y khoa thực tế).
- Giải thích tương phản (Contrastive Explanation): Đánh giá liệu việc giải thích "Tại sao không chọn hành động B" có giúp bác sĩ loại bỏ các rủi ro tốt hơn giải thích "Tại sao chọn hành động A" hay không.

Ví dụ ứng dụng: Hệ thống không chỉ đưa ra lý do cho việc dùng thuốc được chỉ định mà còn cho phép bác sĩ truy vấn các tình huống giả định (counterfactuals). Hiệu quả của XAI sẽ được xác nhận nếu bác sĩ có thể nhận diện nhanh chóng các trường hợp mô hình đưa ra quyết định sai sót do nhiễu dữ liệu.

Paper:

1. [Chanda et al. - 2024 - Dermatologist-like explainable AI enhances trust and confidence in diagnosing melanoma](https://doi.org/10.1038/s41467-023-43095-4)

Đánh giá lòng tin thông qua sự tương đồng (Alignment) và nghiên cứu đa giai đoạn. Nghiên cứu này cung cấp một phương pháp định lượng mạnh mẽ để đo lường lòng tin của bác sĩ da liễu trong việc chẩn đoán khối u ác tính.

- Thiết kế nghiên cứu đa giai đoạn: Sử dụng mô hình nghiên cứu 3 giai đoạn (Không có AI -> Có AI -> Có XAI) để cô lập tác động cụ thể của các lời giải thích đối với độ chính xác, sự tự tin và lòng tin.
- Chỉ số sự tương đồng (Alignment Metrics): Sử dụng hệ số tương đồng Sørensen-Dice (DSC) để đo lường mức độ chồng lấp giữa giải thích của máy và giải thích của chuyên gia. Nghiên cứu chứng minh rằng lòng tin tăng lên tỷ lệ thuận với mức độ tương đồng giữa logic của AI và logic của bác sĩ.
- Đo lường sự tự tin: Cho thấy XAI không chỉ tăng lòng tin vào hệ thống mà còn tăng sự tự tin của chính bác sĩ vào quyết định của họ so với khi chỉ có AI thông thường.

2. [Herm - 2023 - Impact Of Explainable AI On Cognitive Load Insights From An Empirical Study](https://doi.org/10.48550/arXiv.2304.08861)

Đánh giá hiệu quả thông qua gánh nặng nhận thức (Cognitive Load). Nghiên cứu này tập trung vào khía cạnh "hiệu quả" bằng cách đo lường mức độ nỗ lực trí óc mà bác sĩ cần để hiểu lời giải thích.

- Chỉ số Hiệu quả Tâm thần (Mental Efficiency): Đề xuất một công thức kết hợp giữa nỗ lực tinh thần (mental effort), hiệu suất nhiệm vụ (performance) và thời gian thực hiện (task time) để xếp hạng các loại giải thích XAI.
- Phân loại các loại giải thích: So sánh các loại XAI khác nhau (Why, Why-Not, How, How-To, What-Else) và nhận thấy các giải thích cục bộ (local) như "Why" và "Why-Not" có hiệu quả tâm thần cao nhất, giúp bác sĩ ra quyết định nhanh hơn mà ít bị quá tải.
- Đo lường thời gian nhiệm vụ: Cung cấp dữ liệu định lượng về việc các lời giải thích XAI phù hợp có thể giảm đáng kể thời gian ra quyết định (giảm gần một nửa so với không có giải thích).

3. [Lee and Chew - 2023 - Understanding the Effect of Counterfactual Explanations on Trust and Reliance on AI for Human-AI Col](https://doi.org/10.1145/3610218)

Đánh giá lòng tin hiệu chỉnh (Calibrated Trust) và rủi ro lệ thuộc quá mức. Nghiên cứu này giải quyết vấn đề rủi ro nhận thức bằng cách sử dụng giải thích tương phản (Counterfactual) trong bối cảnh phục hồi chức năng sau đột quỵ.

- Lòng tin hiệu chỉnh (Calibrated Trust): Đánh giá liệu bác sĩ có thể nhận biết khi nào AI sai hay không. Nghiên cứu chỉ ra rằng các giải thích Counterfactual (What-if) giúp bác sĩ đánh giá độ chính xác của AI tốt hơn và giảm sự lệ thuộc mù quáng vào các đầu ra sai của mô hình.
- Giảm lệ thuộc quá mức (Overreliance): Cung cấp bằng chứng cho thấy các giải thích dựa trên tính năng (salient features) có thể gây ra lòng tin quá mức, trong khi giải thích Counterfactual giúp bác sĩ tư duy phản biện hơn, giảm 21% sự lệ thuộc vào các gợi ý sai của AI.
- Đánh giá định tính về sự hữu ích: Sử dụng các khảo sát về mức độ minh bạch, sự thất vọng và ý định sử dụng để bổ sung cho dữ liệu định lượng về hiệu suất.