# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 516; đối tượng sau duyệt: 525; ngưỡng accepted: IoU >= 0.9
- Đã **loại 30 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 403 | 403/516 = 78.1% | 516 box máy |
| edited (sửa) | 55 | 55/516 = 10.7% | 516 box máy |
| deleted (xoá) | 52 | 52/516 = 10.1% | 516 box máy |
| added (vẽ thêm) | 63 | 63/525 = 12.0% | 525 đối tượng sau duyệt |

Trong số deleted + added, **9 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 54.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 237 | 19 | 13 | 24 |
| bicycle | 6 | 0 | 2 | 1 |
| motorbike | 109 | 26 | 12 | 12 |
| car | 35 | 1 | 0 | 15 |
| bus | 2 | 0 | 1 | 2 |
| truck | 3 | 0 | 2 | 3 |
| three_wheeler | 11 | 9 | 22 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 56 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 41 | 29 | 15 | 58.6% (KTC 95%: 47–69%) | 73.2% (KTC 95%: 60–83%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 7/12 = 58.3% (KTC 95%: 32–81%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 2/3
  - wrong_class: 2/3
  - loose: 2/3
  - extra: 1/3
  - lọt: img_021.jpg (loose), img_033.jpg (extra), img_018.jpg (missing), img_025.jpg (wrong_class), img_029.jpg (extra)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
