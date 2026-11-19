# Báo cáo Day 08 — 2S202603077 - Lương Hoàng Bảo Trân

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26-m (`r01`, `r02`) | mặc định 0.3 | `threshold_sweep` không có ngưỡng nào <= 0.5 đạt recall mục tiêu 90%; recall cao nhất chỉ 73% trên 22 vật gold, nên giữ mặc định và nêu rõ giới hạn. `r01` cho thấy conf=0.25 giữ được nhiều person nhỏ/ở xa hơn, nhưng cũng tạo false positive. |
| bicycle | YOLO26-m (`r01`, `r02`) | mặc định 0.3 | Chỉ có 1 vật bicycle trong 4 ảnh gold nên mẫu quá ít để chọn ngưỡng riêng; script yêu cầu dùng mặc định. `r01` và `r02` còn cho thấy bánh sau xích lô có thể bị gọi nhầm là bicycle. |
| motorbike | YOLO26-m (`r01`, `r02`) | 0.25 | Đạt recall 90% và precision 69% trên 10 vật gold. Chọn 0.25 vì `r01` giữ được nhiều motorbike nhỏ/ở xa hơn; `r02` ở 0.5 sạch hơn nhưng bỏ sót thêm. |
| car | YOLO26-m (`r01`, `r02`) | 0.5 | Đạt recall 90% và precision 69% trên 10 vật gold. Chọn 0.5 để giảm box thừa; tuy nhiên vẫn cần review box lỏng như xe Lead trắng trong img_030. |
| bus / truck | YOLO26-m (`r01`, `r02`) | mặc định 0.3 | Bus không có ngưỡng nào <= 0.5 đạt recall 90% và truck không có vật nào trong gold; mẫu không đủ để chọn riêng, nên dùng mặc định theo script. |
| three_wheeler | YOLOE-26s, prompt `three_wheeler` (`r03`) | 0.15 | Đạt recall 100% và precision 78% trên 7 vật gold. YOLOE tìm three_wheeler tốt hơn YOLO26, nhưng `r03` vẫn có false positive ở img_002 và img_011 nên phải review. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  ```bash
  python3 scripts/merge_prelabels.py \
    work/prelabel/yoloe_all/pre.coco.json \
    work/prelabel/y26_all/pre.coco.json \
    --thr "motorbike=0.25,car=0.5,three_wheeler=0.15" \
    --conf 0.3 \
    --iou 0.6 \
    --split "batch,gold" \
    --out work/prelabel/merged_review/pre.coco.json
  ```
- **Phiên bản SAM bạn chọn:** SAM3 (`r05`). SAM3 tạo mask trực tiếp từ prompt chữ và nhận diện person, motorbike, car khá tốt trên phần lớn pilot; tuy nhiên chạy rất chậm trên CPU, khoảng 170 giây/ảnh, và có false positive trên banner ở img_015. Vì vậy chỉ dùng để khảo sát/mask vật khó, không dùng làm pipeline pre-label chính.
- Lỗi nào **không sửa được bằng đổi ngưỡng**? YOLO26 là detector closed-set, không có lớp `three_wheeler`, nên xích lô/xe ba bánh ở img_007 bị gọi là `motorbike`; ở img_032 bánh sau xích lô bị gọi là `bicycle`. Đổi threshold chỉ bỏ hoặc giữ box, không sửa được nhãn. Cách xử lý là dùng YOLOE cho `three_wheeler` (`r03`) và review thủ công.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng**: YOLOE ở img_002 có box `three_wheeler` sai trên cụm 3 xô nước; YOLO26 ở img_016 có box `person` sai trên thân cây; SAM3 ở img_015 có 4 box/mask `person` sai trên banner. Ở chiều ngược lại, tăng threshold lên 0.5 làm mất car ở xa trong img_030 và nhiều person/motorbike nhỏ ở xa (`r02`), cho thấy box điểm thấp đôi khi vẫn đúng.
- Giới hạn: threshold được chọn và đánh giá trên cùng 4 ảnh gold, nên precision/recall trên gold có thể lạc quan. Các lớp `bicycle`, `truck` và những lớp có ít vật có độ tin cậy thấp; kết quả chỉ là ước lượng thô.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 403 | 78.1% | 516 box máy sau khi loại vùng quanh bẫy |
| edited | 55 | 10.7% | 516 box máy sau khi loại vùng quanh bẫy |
| deleted | 52 | 10.1% | 516 box máy sau khi loại vùng quanh bẫy |
| added | 63 | 12.0% | 525 đối tượng sau duyệt |

- Lớp máy tệ nhất là `three_wheeler`: có 22 box bị xoá, 9 box bị sửa và chỉ có 11 box được giữ nguyên. YOLOE còn nhầm các vật khác thành `three_wheeler` và bỏ sót một số xe ba bánh/xích lô; các lỗi này phù hợp với quan sát Part 2 ở img_002 và img_011 (`r03`).
- So với gold: precision của **máy** là 41/70 = 58.6% (KTC 95%: 47–69%), recall là 41/56 = 73.2% (KTC 95%: 60–83%) trên 4 ảnh gold và 56 đối tượng gold; nguồn: `submission/qc/review_stats.md`.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt được 7/12 bẫy = 58.3%. Theo loại: `missing` 2/3, `wrong_class` 2/3, `loose` 2/3, `extra` 1/3. Bẫy lọt: img_021 (`loose`), img_033 (`extra`), img_018 (`missing`), img_025 (`wrong_class`), img_029 (`extra`). Các bẫy lọt cho thấy mình chưa nhất quán ở bước quét ảnh để tìm vật bị bỏ sót, kiểm tra box lỏng và kiểm tra box thừa ở các vùng ít nổi bật.
- Accepted đạt 403/516 = 78.1% nhưng bắt bẫy chỉ 7/12 = 58.3%. Hai số đo khác nhau: `accepted` đo các box máy được giữ nguyên sau review, còn bẫy kiểm tra khả năng chủ động phát hiện lỗi được cài có chủ đích. Vì vậy accepted khá cao không có nghĩa review đã bắt được các lỗi tinh vi; cần ưu tiên checklist và quét ảnh độc lập trước khi nhìn nhãn máy.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay gold-manual là 60 phút/4 ảnh = 15 phút/ảnh; review batch là 90 phút/24 ảnh = 3.75 phút/ảnh. Thiên lệch: gold được gán trước khi xem pre-label, nhưng người review batch có thể bị neo bởi box máy.
- Kết luận: Có thể dùng máy như pre-label hỗ trợ, không được release tự động. Cơ sở: chỉ 403/516 = 78.1% box được giữ nguyên, phải sửa/xóa 107/516 = 20.7% box máy, cần thêm 63 đối tượng, và bắt bẫy chỉ 7/12 = 58.3%, thấp hơn mức 80%.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Mỗi ảnh phải được quét độc lập khi ẩn toàn bộ box máy trước, theo thứ tự: bỏ sót → box thừa → sai lớp → box lỏng.
  2. Không kết luận chất lượng chỉ từ `accepted`; luôn kiểm tra thêm box bị xoá/sửa, số vật phải thêm và các bẫy được cài.
  3. Với cụm vật nhỏ/dày dưới 20 px, dùng `ignore_region` theo guideline thay vì cố vẽ từng vật; sau đó kiểm tra lại vùng ignore để không che mất vật đủ lớn.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013` | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Có: mask bao phủ xe ba bánh và người lái; phần khoang/hàng hóa nhìn thấy cũng nằm trong vùng mask. | Mask bám khá sát xe; một số vùng người lái và chi tiết phía trong bị nhập chung, cần ghi nhận khi dùng mask cho vật thể có người đi kèm. | `masks/tuktuk_img013.jpg` |
| `img_032` | SAM 2.1 (`sam2.1_b.pt`) | 4 (2 dương, 2 âm) | Không thấy người lái bị nhập vào; mask chủ yếu bao phủ chiếc xích lô ở giữa ảnh. | Mask tách được xích lô khỏi người đứng bên phải và các xe phía sau; không thấy lỗi ăn sang vật lân cận đáng kể. | `masks/cyclo_img032.jpg` |
| `img_018` | SAM 2.1 (`sam2.1_b.pt`) | 4 (2 dương, 2 âm) | Không; mask chỉ bao phủ người đi bộ được chọn, không nhập sang xe máy/người khác. | Vật được chọn tách khá sạch trong cảnh nhiều xe; các xe và mặt đường xung quanh không bị nhập vào mask. | `masks/person_img018.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc giữ đối tượng theo vật thể thật giúp kiểm tra mask không ăn sang xe/người lân cận; quy tắc về vật thể có người lái hoặc hàng hóa đi kèm chưa nêu ranh giới mask thật chi tiết, nên cần ghi rõ mask có nhập người lái/hàng hóa trong từng ca.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không thực hiện; Part 5 dùng offline `sam2_points.py`. Cách offline dùng tọa độ click dương/âm và lưu ảnh kết quả trực tiếp, còn Interactor SAM thao tác tương tác trong CVAT và gắn với quy trình review của task.

## 6. Calibration (Part 6)

- Số quyết định: 4 (`calibration_decisions.csv`); quan trọng nhất: `CAL-01` chốt xích lô/xe ba bánh là `three_wheeler`, không giữ nhãn `motorbike` của YOLO26.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: làm rõ cách dùng `ignore_region`, ngưỡng nhìn thấy của người trong xe ba bánh, nhận diện xe ba bánh theo số bánh/cấu trúc xe và loại hình người trên banner.
- Nguyên nhân gốc: kết hợp lỗi máy (sai lớp, false positive, bỏ sót) và một số câu hỏi mở của guideline v1.0 chưa quy định đủ cụ thể.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ảnh ngẫu nhiên, seed 42), 110 đối tượng; người kiểm: bạn (hạn chế: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 1/110 = 0.9%; theo **ảnh**: 1/5 = 20.0% (ngưỡng Release ≤ 2%).
- Top lỗi: `sai_vi_tri_kich_thuoc` 1 lỗi, ví dụ `img_031.jpg`, object `#477` có box person quá rộng và bao gồm xe đạp/giỏ hàng phía dưới. Review batch cũng ghi nhận các nhóm bỏ sót, thừa, sai lớp và box lỏng.
- **Giới hạn mẫu nhỏ**: cận trên Wilson 95% của tỉ lệ ảnh lỗi là 62%; mẫu 5 ảnh quá nhỏ để chứng minh ngưỡng 2%.
- Script kiểm được / không kiểm được: các script kiểm được sự tồn tại, định dạng, lớp, box và thống kê; không kiểm được việc box có ôm sát vật thật, vật mà cả máy và người cùng bỏ sót, hoặc việc checklist được thực hiện đầy đủ.

**Quyết định: Release**, ngày 2026-11-19. Lý do: chấp nhận bộ nhãn hiện tại theo quyết định của người làm bài dù QC tự động gợi ý Rework vì lỗi ảnh là 1/5 = 20.0% và bắt bẫy chỉ 7/12 = 58.3%; quyết định Release có giới hạn và không nên hiểu là đạt ngưỡng chất lượng 2%.
Không có việc Rework/Hold bắt buộc theo quyết định hiện tại; nếu tiếp tục cải thiện, ưu tiên hoàn tất calibration và chạy QC mẫu trước khi sử dụng bộ nhãn cho pipeline khác.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: dùng YOLO26, YOLOE, SAM3 và SAM 2.1 để thử model, tạo pre-label/mask; đối chiếu bằng ảnh trực quan, CVAT và các script thống kê. Các quyết định cuối được kiểm tra thủ công trước khi ghi log.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không; gold được gán tay trước khi dùng pre-label để đo.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, không có GPU, chạy CPU; gặp lỗi thiếu `numpy` khi chạy `check_submission.py`, sau đó tạo env kiểm tra và cài `numpy`.
- Dùng cache `data/cache/` thay vì tự chạy? Không; các model chính được chạy theo recipe trong repo.
- Lỗi / bug script đã gặp: SAM3 chạy chậm trên CPU; một mask SAM ban đầu ở `img_018` bị dính vật lân cận và phải chọn lại vật dễ tách hơn; terminal kiểm tra ban đầu thiếu env/numpy.
- Một điều bạn làm khác lần sau: hoàn tất calibration và QC mẫu ngay sau review batch, đồng thời dùng GPU hoặc môi trường Colab cho các lượt SAM chậm.
