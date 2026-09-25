# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Lương Phương Quang
Mã học viên: 2A202602175

Công cụ gán nhãn đã dùng: CVAT Docker local để sửa nhãn trên 12 ảnh theo quy định của lab; kiểm tra và chỉnh sửa dựa trên nguyên tắc của guideline, đồng thời dùng các file kết quả sinh ra từ notebook để đánh giá mô hình.

## 1. Dữ liệu và cách chia tập

Tập chưa gán nhãn (pool) và tập kiểm thử (test set) được chia theo trục thời gian có vùng đệm ở giữa để tránh rò rỉ thông tin giữa các frame liền kề. Nếu chia ngẫu nhiên, các hình ảnh tương tự về ánh sáng, cảnh nền và kiểu xe sẽ xuất hiện đồng thời ở hai tập, khiến số đo trên tập kiểm thử bị đánh giá quá cao. Điều này làm giảm độ tin cậy của AP50 vì mô hình không thực sự được kiểm nghiệm trên dữ liệu mới, chưa từng thấy trước đó.

## 2. Mô hình khởi đầu lạnh (cold start)

Theo quá trình cold start, mô hình thường bỏ sót các xe có kích thước nhỏ, nằm ở góc xa hoặc bị che mảnh bởi bóng đen, cột điện, hoặc các vật thể lân cận. Độ phủ (recall) giảm rõ ở nhóm xe nhỏ và xe ở mép khung hình, cho thấy mô hình thiên về các xe lớn, rõ nét và gần trung tâm hơn. Một trường hợp cần rà lại nhãn tham chiếu trước khi kết luận mô hình sai là khi xe chỉ hiện một phần thân, bị che gần hết hoặc nằm trong vùng sáng tối mạnh; trong những trường hợp này, nhãn tham chiếu có thể không đồng nhất với quy tắc gán nhãn của con người và cần kiểm tra kỹ trước khi chốt đánh giá.

## 3. Chiến lược chọn mẫu

Cách chọn mẫu trong active learning dựa trên ý tưởng rằng mỗi frame được ưu tiên dựa trên độ bất định, giá trị thông tin và mức độ khác biệt so với các frame đã chọn. Công thức `score = W_U·U + W_A·A + W_D·D` thể hiện rằng độ bất định `U` cho biết mô hình không chắc chắn; độ ảnh hưởng của nhãn `A` thể hiện mẫu đó có khả năng giúp học thêm gì; và độ chênh lệch `D` đảm bảo ảnh được chọn không quá giống với những frame đã có. `MIN_GAP_S` giúp tránh chọn các ảnh quá gần nhau về thời gian hoặc nội dung, duy trì tính đa dạng của lô mẫu và tăng hiệu quả học.

Ba frame quan trọng trong lựa chọn là frame_0044.jpg, frame_0144.jpg và frame_0244.jpg: chúng thể hiện các trường hợp xe xuất hiện rõ, có độ bất định đáng kể và có giá trị cải thiện mô hình. Một frame có điểm cao nhưng không chọn là frame_0156.jpg vì nó gần với các frame lân cận và không mang thêm thông tin đáng kể. Một frame có điểm thấp nhưng vẫn nên xem là frame_0350.jpg, vì nó nằm trong môi trường tối và có bóng mờ, rất dễ gây nhầm giữa xe thật và phản quang hoặc ảnh nền. Điểm bất định chỉ chứng minh ảnh đó hữu ích khi nó đồng thời là một trường hợp khó, có độ đa dạng cao và không quá lặp với các mẫu đã có.

## 4. Các vòng học chủ động (active learning)

Trong vòng học chủ động, tôi ưu tiên sửa các box AI sai vị trí, bỏ box giả, thêm các xe bị model bỏ sót và giữ lại những box đã đúng theo guideline. Cách làm này giúp loại bỏ lỗi nghiêm trọng trước khi đưa dữ liệu vào fine-tune, thay vì chỉ tin tưởng vào độ tin cậy của model. Những trường hợp sửa mạnh nhất thường là xe ở mép khung hình, giá trị tương phản thấp, và xe che bởi góc nhìn nghiêng hoặc bóng tối.

Một ca đáng chú ý sau fine-tune là trường hợp xe ở góc xa hoặc mép ảnh: dù ban đầu model bỏ sót hoặc gán box lệch, sau khi dùng nhãn đã sửa và fine-tune, mô hình có xu hướng khớp tốt hơn với hình dạng thực của xe. Ngược lại, các trường hợp có ánh sáng mạnh hoặc bóng phản quang vẫn dễ bị nhầm với vật thể không phải xe, dù đã có chỉnh sửa. Cách phân biệt giữa quan sát độc lập, lỗi pre-label sửa và kết quả sau train được ghi rõ trong BLIND_SCAN.md, REVIEW_LOG.csv và các file diff; điều này giúp tôi tách rời ba yếu tố: quan sát ban đầu, quyết định sửa nhãn và hiệu ứng thực tế của mô hình.

Một ca khó theo guideline là xe ở vị trí mép phải trong cảnh tối, bị che bởi bóng và có độ tương phản thấp. Trong trường hợp ấy, không được giữ nguyên box AI chỉ vì confidence cao; cần kiểm tra thật kỹ xem đó có phải là xe hay phản quang, đồng thời đảm bảo box bao trọn thân xe mà không kéo quá dài vào nền.

## 5. Kết luận và giới hạn

So với cold start, vòng học chủ động giúp cải thiện độ phủ và sự ổn định của mô hình ở các trường hợp khó, đặc biệt là xe ở đoạn biên hoặc trong bóng tối. Nếu còn thời gian, nên tiếp tục chọn thêm vài frame khó để giảm sai lệch, nhưng không cần làm thêm vòng không có ý nghĩa vì dữ liệu nhỏ và chi phí rà nhãn tăng lên đáng kể. Hai trường hợp còn yếu cho vòng sau là xe nhỏ ở góc mép trái trong điều kiện sáng yếu và xe nằm trong vùng phản quang, vì chúng tốn nhiều công và dễ mắc nhầm với vật thể nền hoặc ảnh gần trùng.

Tập kiểm thử chỉ có 20 ảnh nên không thể khẳng định tuyệt đối chất lượng mô hình; nhãn tham chiếu do mô hình tạo cũng chưa được rà thủ công, nên các kết luận cần xem như suy đoán dựa trên cùng tập đánh giá. Nếu AP50 giảm sau một vòng train, trước tiên tôi sẽ kiểm tra lại nhãn đã sửa ở các frame khó, xác minh các box không bị lệch, thiếu hoặc dư, rồi mới xét đến việc thêm dữ liệu hay dừng lại. Điều này giúp tránh gán nguyên nhân sai cho mô hình khi thực chất là lỗi trong dữ liệu gán nhãn.
