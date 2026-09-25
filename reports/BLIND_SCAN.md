# Quét độc lập trước khi xem pre-label

Frame: frame_0044.jpg

Số xe nhìn thấy bằng mắt: 3

Hai vị trí dễ bị AI bỏ sót hoặc vẽ sai, kèm mô tả xe:
- Vị trí 1: xe ở phía trước hàng xe chính, gần mép trái của khung hình; phần đầu xe hơi che bởi cây và bóng tối, rất dễ bị AI bỏ sót vì vật thể còn xuất hiện rõ thân xe nhưng không có ranh giới cạnh rõ.
- Vị trí 2: xe ở góc phải phía xa, nhỏ hơn và gần với nền, dễ bị model nhận nhầm thành đốm sáng hoặc đoạn phản quang; cần chú ý kiểm tra cả chiều dài và chiều rộng để tránh box quá ngắn.

Chạy `python3 tools/lock_blind.py` ngay sau khi điền. Sau đó giữ file này nguyên vẹn.
