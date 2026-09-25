# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Bản báo cáo dưới đây là phiên bản minh họa / mô phỏng các số liệu trong quá trình học chủ động, dùng để hỗ trợ viết báo cáo khi chưa có kết quả chạy Colab thực tế. Các giá trị AP50, số lượng box và nhận xét là giả định hợp lý theo cấu trúc lab và không thay thế dữ liệu thật từ notebook.

Họ và tên: Lương Phương Quang
Mã học viên: 2A202602175

Công cụ gán nhãn đã dùng: CVAT Docker local + chỉnh sửa nhãn theo guideline, với dữ liệu minh họa dùng cho báo cáo ban đầu.

## 1. Dữ liệu và cách chia tập

Tập chưa gán nhãn (pool) và tập kiểm thử (test set) được chia theo trục thời gian, có vùng đệm ở giữa, để ngăn rò rỉ thông tin giữa các frame liên tiếp. Nếu chia ngẫu nhiên, các frame liền kề sẽ xuất hiện cả trong train và test, khiến mô hình học nhầm đặc điểm nền và ánh sáng của cùng một chuỗi video. Khi đó, AP50 sẽ bị ảo hóa ở mức cao hơn thực tế vì mô hình đang nhìn thấy các mẫu tương tự trong tập đánh giá.

Với mô phỏng này, tập pool có 268 frame, trong khi tập test gồm 20 frame được rút ra sau vùng đệm. Tỷ lệ dữ liệu phù hợp với mô hình hoạt động trên video đường cao tốc ban đêm, nơi các xe lớn nằm ở trung tâm dễ hơn để nhận dạng, còn xe nhỏ ở các góc mép bị bỏ sót nhiều hơn.

## 2. Mô hình khởi đầu lạnh (cold start)

Mô hình khởi đầu lạnh đạt AP50 khoảng 0.38 trên tập test. Đây là mức thấp do các xe nhỏ, nằm ở vị trí mép khung hình hoặc bị che bởi bóng tối thường bị bỏ sót. Độ phủ (recall) ở nhóm xe nhỏ giảm mạnh, trong khi các xe lớn và rõ hình dạng ở trung tâm vẫn được phát hiện tương đối tốt.

Trong vòng cold start, một số trường hợp đáng chú ý là xe ở góc phải xa và xe gần mép trái, nơi model không kịp nhận dạng do độ tương phản thấp. Một trường hợp cần rà lại nhãn tham chiếu trước khi kết luận mô hình sai là khi xe chỉ hiện phần thân và bị che bởi lan can hoặc bóng đen; trong trường hợp đó, nhãn tham chiếu có thể do model sinh ra không hoàn toàn đáng tin cậy.

## 3. Chiến lược chọn mẫu

Chiến lược chọn mẫu dựa trên công thức `score = W_U·U + W_A·A + W_D·D`. Ở đây, `U` là mức độ bất định của model, `A` phản ánh giá trị thông tin của frame để cải thiện học, và `D` là độ cách biệt so với các ảnh đã chọn. `MIN_GAP_S` giúp tránh chọn các frame quá gần nhau về không gian thời gian, tránh lặp lại cùng một cảnh và làm giảm hiệu quả của lô mẫu.

Trong phần mô phỏng này, 5 frame ưu tiên được chọn gồm: frame_0044.jpg, frame_0050.jpg, frame_0144.jpg, frame_0147.jpg và frame_0244.jpg. Ba trong số đó là frame_0044.jpg, frame_0144.jpg và frame_0244.jpg: các frame này chứa xe ở vị trí có độ bất định cao, đồng thời không trùng quá nhiều với các ảnh khác. Một frame có điểm cao nhưng không chọn là frame_0156.jpg vì nó gần với các frame lân cận và mang giá trị học thấp. Một frame có điểm thấp nhưng vẫn hữu ích là frame_0350.jpg vì nằm trong cảnh tối và có bóng phản quang, do đó rất phù hợp để kiểm tra sai lệch. Điểm bất định chứng minh ảnh đó hữu ích khi nó vừa khó, vừa có tính đa dạng, và vừa không trùng với dữ liệu đã chọn trước đó.

## 4. Các vòng học chủ động (active learning)

Trong vòng học chủ động, tôi ưu tiên sửa các box AI sai vị trí, xóa box giả, thêm box bị thiếu và giữ lại những box đã khớp với guideline. Mức sửa nhãn ước tính là 12 box được chỉnh sửa, 3 box bị xóa và 2 box mới được thêm. Nếu tính theo quy mô lô 12 ảnh, đây là mức thay đổi hợp lý cho phân đoạn đường cao tốc ban đêm.

Bảng số liệu mô phỏng:
- Vòng 0: AP50 = 0.38
- Vòng 1: AP50 = 0.52
- Tăng so với cold start: +0.14
- Tăng so với vòng trước: +0.14

Tăng này tập trung ở nhóm xe lớn và xe trung bình ở vùng trung tâm và gần mép, vì các lớp này có nhiều thông tin hơn và ít bị che quá mức. Một trường hợp đáng chú ý là frame_0044.jpg: trước đó model bỏ sót xe ở góc mép, nhưng sau khi model được fine-tune với nhãn sửa, xe này được phát hiện với box khớp hơn và không còn bị lệch quá nhiều. Một trường hợp khó theo guideline là xe ở mép phải trong cảnh tối, nơi phần thân xe bị che bởi bóng và nên không giữ nguyên box AI chỉ vì confidence cao.

## 5. Kết luận và giới hạn

Kết quả mô phỏng cho thấy vòng học chủ động giúp tăng AP50 từ 0.38 lên khoảng 0.52, tương ứng với mức cải thiện đáng kể theo lô dữ liệu nhỏ. Nếu còn thời gian, tôi sẽ tiếp tục chọn thêm 2–3 frame khó ở vùng sáng tối và góc mép để cải thiện độ phủ. Tuy nhiên, với tập test chỉ 20 ảnh và nhãn tham chiếu là nhãn do mô hình tạo, kết luận cần được xem là ước tính chứ không phải “chân lý tuyệt đối”.

Hai kịch bản còn yếu cho vòng sau là: xe nhỏ ở góc mép trái trong điều kiện tối và xe ở vùng phản quang, vì hai trường hợp này dễ nhầm với nền và tốn nhiều chi phí rà nhãn. Nếu AP50 giảm, điều đầu tiên cần kiểm tra là các box đã sửa trong các frame khó, xác định xem lỗi đến từ box lệch, box giả, hay thiếu box, trước khi quyết định train thêm hoặc dừng lại.

> Lưu ý: Đây là bản báo cáo mô phỏng, dùng cho mục đích viết và trình bày khi chưa có số liệu thật từ Colab. Khi đã chạy notebook thực tế, bạn nên thay thế bằng chính xác các giá trị AP50, số lượng box và nhận xét dựa trên kết quả thu được.
