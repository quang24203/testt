# Vì sao chọn lô này?

Trong 50 dòng đứng đầu `outputs/selection_round1.csv`, chọn năm frame bạn sẽ ưu tiên nếu chỉ có ngân sách rà năm ảnh. Ghi tên, điểm, thời gian, thứ tự và lý do; tối thiểu một quyết định phải xét ảnh gần trùng hoặc trường hợp model không dự đoán được box: frame_0044.jpg, frame_0050.jpg, frame_0144.jpg, frame_0147.jpg, frame_0244.jpg.

Ba frame thuộc lô 12 ảnh model chọn và bằng chứng trong CSV/ảnh contact sheet:
- frame_0044.jpg: điểm cao, vùng xe xuất hiện rõ và có thể cải thiện độ phủ vì model ban đầu bỏ sót xe ở vị trí nghiêng, phù hợp để chỉnh sửa trước khi fine-tune.
- frame_0144.jpg: xe tương đối lớn, bề mặt rõ, là ví dụ phủ tốt và giúp kiểm tra nhãn có quá sát hay quá rộng không.
- frame_0244.jpg: có sự tương phản cao giữa xe và nền, cho thấy model dự đoán có tính nhất quán ở trạng thái phức tạp vừa phải.

Một frame có điểm cao nhưng không chọn hoặc một frame có điểm thấp vẫn nên xem, và lý do:
- frame_0156.jpg: có điểm tương đối cao nhưng gần trùng với các ảnh lân cận trong đoạn video, nên nếu chọn thêm sẽ tạo dư thừa thông tin và không cải thiện đáng kể độ bao phủ của mẫu.
- frame_0350.jpg: không phải điểm cao nhất nhưng là trường hợp khó với ánh sáng yếu và bóng đen, vì vậy vẫn nên xem để đánh giá một số box giả/thiếu xác thực.

Điều phép chọn này chưa chứng minh về chất lượng mô hình:
- Lần này, tôi chọn theo ba yếu tố chính: độ bất định (uncertainty), ảnh gần trùng và chi phí rà nhãn. Một số frame có điểm cao nhưng quá tương đồng với nhau không mang thêm thông tin; ngược lại, một vài frame có độ bất định lớn nhưng lại chứa vùng khó do che khuất hoặc ánh sáng yếu, rất phù hợp để xem xét. Vì vậy, lựa chọn này giúp cải thiện mẫu huấn luyện nhưng chưa thể khẳng định mô hình đã vượt trội trong mọi tình huống.
