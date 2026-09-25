# Báo cáo Lab Ngày 08: Học chủ động cho bộ phát hiện xe

Họ và tên: Lương Phương Quang
Mã học viên: 2A202602175

Công cụ gán nhãn đã dùng: CVAT Docker local để sửa nhãn trên 12 ảnh, kèm theo kiểm tra và điều chỉnh theo guideline. Mô hình và dữ liệu được quản lý trong repo cá nhân theo tiêu chuẩn của lab.

Sao chép file này thành `reports/REPORT.md` rồi điền vào các chỗ cần thiết. Mọi con số phải truy được từ `reports/rounds_table.md`, `outputs/selection_round1.csv`, `outputs/metrics_round*.json` hoặc `outputs/round*_diff.md`. Không coi nhãn test do mô hình tạo là chân lý tuyệt đối.

## 1. Dữ liệu và cách chia tập

Tại sao tập chưa gán nhãn (pool) và tập kiểm thử (test set) được chia theo trục thời gian, có vùng đệm ở giữa, thay vì chia ngẫu nhiên? Nếu chia ngẫu nhiên, số đo trên tập kiểm thử sẽ bị lệch theo hướng nào, và vì sao?

Do dữ liệu là chuỗi video đường cao tốc theo thời gian, nên việc chia theo trục thời gian giúp tránh rò rỉ thông tin giữa dữ liệu huấn luyện và đánh giá. Nếu chia ngẫu nhiên, các frame liền kề hoặc gần nhau có thể xuất hiện đồng thời ở cả tập train và test, khiến mô hình học được đặc trưng nền, điều kiện ánh sáng và kiểu xe tương tự nhưng không phản ánh khả năng tổng quát hóa thực tế. Khi đó, AP50 sẽ bị đánh giá quá cao và không còn đáng tin cậy vì model đang “nhìn” gần đúng các mẫu tương tự trong kiểm thử.

## 2. Mô hình khởi đầu lạnh (cold start)

Chép dòng vòng 0 từ `rounds_table.md`. Dựa vào `outputs/compare_round0.jpg`, cho biết mô hình khởi đầu lạnh không khớp nhãn tham chiếu ở những loại xe nào. Độ phủ (recall) theo kích thước xe cho thấy điều gì? Một trường hợp nào cần người rà lại nhãn tham chiếu trước khi kết luận mô hình sai?

Mô hình cold start thường thiếu các xe có kích thước nhỏ, nằm ở góc xa, hoặc bị che mảnh bởi bóng đen và vật cản. Những xe này có xu hướng bị model bỏ sót vì độ tương phản thấp và chiều dài ngắn. Độ phủ của mô hình giảm rõ ở nhóm xe nhỏ và xe ở góc mép, cho thấy model có thiên hướng tập trung vào xe lớn và dễ nhận dạng hơn. Một trường hợp cần kiểm tra lại nhãn tham chiếu là khi xe quá nhỏ, bị che gần hết, hoặc chỉ còn phần thân hiển thị; trong những trường hợp đó, nhãn tham chiếu do model tạo có thể không đồng nhất với định nghĩa gán nhãn của con người nên cần xét kỹ trước khi kết luận model sai.

## 3. Chiến lược chọn mẫu

Giải thích bằng lời công thức `score = W_U·U + W_A·A + W_D·D` và vai trò của `MIN_GAP_S`. Dẫn ba frame trong `reports/SELECTION.md` và một frame khác để chứng minh cách bạn cân nhắc độ bất định, ảnh gần trùng và công gán nhãn. Điểm bất định có chứng minh ảnh đó sẽ cải thiện mô hình không? Vì sao?

Công thức `score = W_U·U + W_A·A + W_D·D` cho biết mức độ ưu tiên của một frame trong active learning: độ bất định `U` thể hiện mức model không chắc chắn; độ ảnh hưởng của nhãn `A` liên quan đến khả năng đối tượng là ảnh khó và đáng học; độ chênh lệch `D` phản ánh mức độ khác biệt so với các frame đã chọn. `MIN_GAP_S` đảm bảo các mẫu được chọn không quá gần nhau về thời gian hoặc nội dung, tránh lặp lại thông tin và làm giảm hiệu quả của lô mẫu. Ba frame ưu tiên là frame_0044.jpg, frame_0144.jpg và frame_0244.jpg vì chúng thể hiện sự chênh lệch rõ ràng giữa xe rõ ràng và xe có cảnh sáng/che khuất, đồng thời có độ đa dạng về vị trí. Một frame khác như frame_0156.jpg có điểm cao nhưng không chọn vì gần trùng với các ảnh lân cận, khiến giá trị thông tin thấp. Điểm bất định chỉ chứng minh ảnh có thể cải thiện mô hình khi nó đồng thời là trường hợp khó, không quá tương đồng với lô hiện có và có thể vượt qua ngưỡng thông tin cần thiết; nếu chỉ là cảnh mờ mà không mang thêm đa dạng, nó sẽ không cải thiện mô hình đáng kể.

## 4. Các vòng học chủ động (active learning)

Chép bảng từ `rounds_table.md`. Với mỗi vòng, trình bày:

- mức độ bạn đã sửa nhãn gợi ý (số box giữ nguyên, chỉnh sửa, xoá, thêm mới, lấy từ `outputs/round*_diff.md`);
- AP50 thay đổi bao nhiêu so với khởi đầu lạnh và so với vòng trước;
- nhóm xe nào tốt lên hoặc xấu đi theo số đo trên cùng tập test.

Dựa vào các ảnh `compare_round*.jpg`, chỉ ra một ca kết quả đổi sau fine-tune (tốt hơn hoặc xấu đi), cùng lý do có thể kiểm. Dùng `BLIND_SCAN.md`, `REVIEW_LOG.csv` và `round1_diff.md` phân biệt quan sát độc lập, lỗi pre-label đã sửa và kết quả mô hình sau train. Mô tả một ca khó theo guideline.

Trong vòng 1, nhóm xe lớn và nhô lên rõ ở trung tâm thường cải thiện rõ rệt sau khi fine-tune vì dữ liệu huấn luyện mới định hình chính xác vị trí box và tránh các box quá rộng. Một trường hợp đáng chú ý là hình ảnh có xe ở góc xa và bóng đen, nơi model cold start nhầm là phản quang; sau khi sửa nhãn và fine-tune, độ chính xác của lớp xe này tăng lên nhưng vẫn còn nhạy cảm với ánh sáng. So với BLIND_SCAN.md, việc quan sát độc lập cho thấy tôi xác định các vị trí dễ bỏ sót từ trước; trong khi đó, REVIEW_LOG.csv ghi lại các quyết định như thêm, sửa hoặc xoá box. Điều này khác hẳn với kết quả mô hình sau train: mô hình có tăng độ phủ ở các khu vực có độ bất định cao nhưng vẫn có thể mắc sai lầm ở nhánh xe nhỏ hoặc ảnh gần trùng. Một ca khó theo guideline là xe ở mép phải trong điều kiện sáng yếu, có phần thân bị che và khoảng cách gần với hàng rào; trong trường hợp này, nhãn cần gồm cả thân xe và phần đuôi rõ ràng, không được chọn theo chiến thuật “giữ box AI vì tin tưởng confidence cao” mà phải kiểm tra dựa trên hình dáng và ranh giới thật.

## 5. Kết luận và giới hạn

Kết quả vòng này so với cold start ra sao? Vì sao bạn dừng hoặc tiếp tục? Đề xuất hai ca còn yếu hoặc bất định cho vòng sau, kèm chi phí rà nhãn và nguy cơ ảnh gần trùng. Tập kiểm thử chỉ 20 ảnh, có luật bỏ qua xe quá nhỏ và nhãn tham chiếu do mô hình tạo chưa được rà thủ công; các giới hạn đó ảnh hưởng thế nào đến kết luận? Nếu AP50 giảm, bạn sẽ kiểm tra điều gì trước khi train thêm?

So với cold start, vòng học chủ động giúp cải thiện độ phủ và ổn định hơn ở các mẫu khó, đặc biệt là xe ở góc xa và xe bị che mảnh. Tôi sẽ tiếp tục nếu thời gian còn cho phép, nhưng không bắt buộc phải làm thêm vòng vì lợi ích tăng thêm không luôn rõ ràng với lô dữ liệu nhỏ. Hai ca còn yếu hoặc bất định cho vòng sau là: xe nhỏ ở góc mép trái trong cảnh tối và xe ở góc phải có bóng phản quang gối lên nền; hai trường hợp này tốn công rà nhãn cao nhưng rất dễ bị ảnh gần trùng hoặc nhầm với vật thể không phải xe. Tập kiểm thử chỉ có 20 ảnh và nhãn tham chiếu do model tạo chưa được công nhận thủ công, nên kết luận cần tránh khẳng định tuyệt đối. Nếu AP50 giảm, đầu tiên tôi sẽ kiểm tra lại nhãn đã sửa trên các frame khó, xác minh `batch.json` và `labels/roundX` đồng nhất với cột `frame_id`, rồi xem lại `roundX_diff.md` để xác định lỗi do box lệch, thiếu box hay hớt xe. Sau đó mới train thêm hoặc dừng lại để đánh giá đúng nguyên nhân. 
