# Thông tin học viên

- Họ và tên: Lương Hoàng Bảo Trân
- MSHV: 2S202603077
- Lớp (L2 / L3 / khác): L2A
- Tài khoản GitHub: lhbtran2003
- Repo bài nộp (URL): https://github.com/lhbtran2003/K5-L23-DAY08-2A202603077-LuongHoangBaoTran-AIPrelabel-
- Commit chốt bài (hash):
- Máy: Windows 11, không có GPU
- CVAT (Docker): chưa ghi phiên bản

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 403 / 55 / 52 (mẫu số: 516 box máy); added: 63 (mẫu số: 525 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 58.6% (41/70) / 73.2% (41/56)
- Bắt bẫy: 7/12 = 58.3%
- Thời gian: gán tay 15 phút/ảnh (60 phút/4 ảnh); duyệt có pre-label 3.75 phút/ảnh (90 phút/24 ảnh)
- Lỗi QC: 1/110 = 0.9% theo đối tượng; 1/5 = 20.0% theo ảnh (mẫu 5 ảnh, seed 42); cận trên Wilson 95% theo ảnh là 62%.
- **Quyết định:** Release — chấp nhận bộ nhãn hiện tại dù QC tự động gợi ý Rework, bắt bẫy đạt 7/12 = 58.3% và recall trên gold là 41/56 = 73.2%; quyết định có giới hạn do mẫu đánh giá nhỏ.
