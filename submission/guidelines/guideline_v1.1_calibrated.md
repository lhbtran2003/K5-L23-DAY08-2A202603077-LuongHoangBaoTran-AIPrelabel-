# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2S202603077 - Lương Hoàng Bảo Trân

> Đây là phần bổ sung và chốt các ca mơ hồ so với guideline v1.0. Mỗi ca có ảnh ví dụ và khớp với một dòng trong `submission/qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ/dày đặc gần camera: từng xe hay `ignore_region`? | Gán từng xe nếu còn nhìn thấy và tách được vật; chỉ dùng `ignore_region` cho cụm từ 5 vật trở lên khi các vật nhỏ, cao dưới 20 px hoặc không thể tách đáng tin cậy. | `img_032` | `CAL-02` |
| 4 | Người ngồi trong xe ba bánh có mui che, chỉ thấy thấp thoáng, có gán không? | Gán `person` riêng khi còn nhìn thấy khoảng từ 20% cơ thể trở lên; dưới ngưỡng đó không gán người, chỉ giữ nhãn xe nếu xe đủ rõ. | `img_013` | `CAL-03` |

## 2. Ca khó mới (từ lỗi của máy và bẫy lọt)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_007` | Xích lô/xe ba bánh bị gọi sai lớp | YOLO26 gọi `three_wheeler` là `motorbike` ở `r01`/`r02`. | Gán `three_wheeler` cho mọi xe ba bánh, gồm tuk-tuk, xích lô và xe ba gác; người lái là `person` riêng. | Bổ sung mục 2 và 4.3: ưu tiên số bánh và cấu trúc xe, không giữ nhãn detector nếu sai lớp. |
| `img_015` | Hình người trên banner bị nhận nhầm là người thật | SAM3 tạo 4 box/mask `person` trên banner quảng cáo. | Không gán hình in, ảnh trên biển quảng cáo, tượng hoặc phản chiếu thành `person`. | Nhắc lại mục 2 và 4.1.7: vật thể không có trong không gian thật không được gán. |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | — |
| v1.1 | Chốt cách xử lý cụm xe dày, người bị che trong tuk-tuk; làm rõ xe ba bánh luôn là `three_wheeler`; loại hình người trên banner khỏi `person`. | `CAL-01`, `CAL-02`, `CAL-03`, `CAL-04` |
