# Báo cáo Day 08 — 2A202603199 - Nguyen Tuan Hung

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp                                              | Model + cỡ                                              | Ngưỡng       | Vì sao (dẫn `run_id` trong `model_log.csv`)                                                                                                                                                                              |
| ------------------------------------------------ | ------------------------------------------------------- | ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| person / bicycle / motorbike / car / bus / truck | YOLO26m                                                 | mặc định 0.3 | Không lớp nào đủ dữ liệu và recall ≥90% trong vùng chọn ngưỡng; person/motorbike/car không đạt mục tiêu, bicycle/bus/truck có dưới 5 mẫu hoặc không có mẫu trong gold (`r01`, `r02`; `submission/qc/threshold_sweep.md`) |
| three_wheeler                                    | YOLOE-26s, prompt: `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15         | Recall 7/7 = 100%, precision 78%; dùng YOLOE vì YOLO26 bỏ sót hoặc nhầm xe ba bánh (`r03`; `submission/qc/threshold_sweep.md`)                                                                                           |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts\merge_prelabels.py work\prelabel\yoloe_all\pre.coco.json work\prelabel\y26_all\pre.coco.json --thr 'three_wheeler=0.15' --conf 0.3 --iou 0.6 --split 'batch,gold' --out work\prelabel\merged_review\pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: dùng SAM 2.1 cho các mask còn lại sau khi so sánh định tính với SAM 1 trên `img_013`. SAM 2.1 tạo mask khởi đầu dùng được nhưng vẫn có lỗi ở khoang mở và các vùng nhỏ; phép thử này không chứng minh SAM 2.1 tốt hơn về chất lượng hay tốc độ vì chưa đo IoU/RAM hoặc benchmark có kiểm soát.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLO26 gán tuk-tuk thành `car` ở `img_013`; đổi `--conf` không sửa được sai lớp. Đổi nhãn sang `three_wheeler` theo taxonomy hoặc dùng YOLOE rồi review thủ công.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): `img_002`, YOLOE phát hiện đúng tuk-tuk với conf 0.93 nhưng cũng tạo box thừa trên xe bán tải với conf 0.37 (`r03`, ảnh viz).
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: kết quả in-sample không đại diện độc lập; chỉ có 4 ảnh và số mẫu mỗi lớp nhỏ (`submission/qc/threshold_sweep.md`, `submission/qc/review_stats.md`).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái |  Số | Tỉ lệ | Mẫu số                  |
| ---------- | --: | ----: | ----------------------- |
| accepted   | 364 | 70.3% | 518 box máy             |
| edited     |  71 | 13.7% | 518 box máy             |
| deleted    |  80 | 15.4% | 518 box máy             |
| added      |  32 |  6.9% | 467 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? `three_wheeler`: chỉ 6 box accepted, 9 edited, 28 deleted và 6 added; YOLO26 thường nhầm tuk-tuk thành `car`/`truck`, còn YOLOE có thể tạo box thừa/lỏng (`submission/qc/review_stats.md`, ví dụ `img_002`, `img_013`).
- So với gold: precision / recall của **máy** 56.1% (37/66) / 88.1% (37/42) (mẫu số: 42 đối tượng gold; số đo trong `submission/qc/review_stats.md`).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 10/12 = 83.3%; loại nào lọt: 2 bẫy `missing` ở `img_013.jpg`, `img_028.jpg`; cần quét toàn ảnh trước khi xem nhãn máy để phát hiện vật không có box (`submission/qc/review_stats.md`).
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted 70.3% không chứng minh nhãn máy đúng: vẫn có 80/518 box bị xóa, 71/518 phải sửa và còn 2 bẫy missing lọt. Cần kiểm tra ảnh độc lập với các box máy.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 6.25 phút/ảnh (25/4) vs duyệt có pre-label 3.63 phút/ảnh (87/24). Thiên lệch (bạn gán gold trước nên đã quen cảnh): thời gian có thể bị ảnh hưởng vì đã quen với loại cảnh và nhãn trong bộ dữ liệu; cần thận trọng khi so sánh.
- Kết luận: có, dùng máy để đề xuất box, đặc biệt YOLOE cho `three_wheeler`, nhưng không auto-accept; dựa trên accepted 364/518 = 70.3%, edited 71/518 = 13.7%, deleted 80/518 = 15.4%, added 32/467 = 6.9% và precision 37/66 = 56.1% (`review_stats.md`).
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Quét toàn ảnh trước khi nhìn pre-label, ghi lại vật bị bỏ sót.
  2. Rà theo từng lớp và sửa mọi box sai lớp/lỏng; không dùng confidence làm bằng chứng nhãn đúng.
  3. Kiểm tra box thấp hơn 20 px theo guideline và hoàn thành QC mẫu trước khi quyết định release.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh     | SAM (phiên bản) |          Số click | Mask gồm người lái / hàng hoá?                                                    | Lỗi quan sát                              | File trong `masks/`      |
| ------- | --------------- | ----------------: | --------------------------------------------------------------------------------- | ----------------------------------------- | ------------------------ |
| img_013 | SAM 1           | 3 (2 dương, 1 âm) | Tuk-tuk và mui xe được mask; vùng khoang người lái cần kiểm tra/tách khỏi mask xe | Rìa khoang và phần dưới xe còn nhiễu      | `tuktuk_img013_sam1.jpg` |
| img_013 | SAM 2.1         | 3 (2 dương, 1 âm) | Mask thân xe còn lỗi quanh khoang mở và vùng người lái; chưa thể xem là đã tách người sạch | Nhiều lỗ/đốm và biên rỗ quanh khoang, cần click âm hoặc sửa tay | `tuktuk_img013_sam2.jpg` |
| img_004 | SAM 2.1         |                 3 | Mask xe ba bánh; không thấy hàng hóa rời cần gộp vào xe                           | Biên dưới bánh và vùng sát bóng chưa sạch | `tuktuk_img004_sam2.jpg` |
| img_017 | SAM 2.1         |                 3 | Mask xe; không thấy hàng hóa rời cần gộp vào xe                                   | Có lỗ/đốm nhỏ ở vùng cửa sổ và mái        | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Quy tắc mask theo phần nhìn thấy, không lấy bóng đổ và tách hai vật giúp hậu kiểm. SAM có thể lẫn người với xe hoặc tạo lỗ/đốm sai; cần click âm và sửa tay. Quy tắc rõ về kính/cửa sổ và phân tách người trong khoang được bổ sung ở v1.1.
- _(Tuỳ chọn)_ Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: Chưa thử Interactor SAM trong CVAT; các kết quả trên chạy offline.

## 6. Calibration (Part 6)

- Số quyết định: 3 (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): CAL-01 — YOLO26 nhầm tuk-tuk thành `car`, không thể sửa bằng đổi confidence.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: phân lớp theo định nghĩa taxonomy; quét toàn ảnh để tìm vật thiếu; người nhìn thấy trong khoang xe là instance riêng, kính/khung cửa thuộc mask phương tiện.
- Nguyên nhân gốc: máy (sai lớp / bỏ sót) và guideline v1.0 còn câu hỏi mở về mask; checklist quét ảnh cần được áp dụng nhất quán.

## 7. QC và quyết định (Part 6)

- Mẫu QC gồm 5 ảnh: không có ảnh `can_review`, lấy ngẫu nhiên 5 ảnh với seed 42. Tôi (Nguyen Tuan Hung) trực tiếp rà các ảnh và ghi kết quả trong `submission/qc/qc_review.csv`. Vì tôi vừa thực hiện batch review vừa chọn mẫu QC, kết quả có thể chịu ảnh hưởng chủ quan.
- Lỗi theo **đối tượng**: 1/96 = 1.0% · theo **ảnh**: 1/5 = 20.0% (ngưỡng Release ≤ 2%; `submission/qc/qc_report_auto.md`, ghi trong `submission/qc/qc_review.csv`).
- QC ghi nhận một lỗi sai vị trí/kích thước: ở `img_031.jpg`, box `bicycle` quá rộng và lấn sang vùng người/nền. Người bán có box `person` riêng, không gộp vào nhãn xe; giỏ hàng gắn cố định được tính theo quy tắc xe đạp chở hàng. ID trên CVAT là 420; ID annotation COCO trong phiếu QC là 414.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): 62% theo Wilson; 5 ảnh không đủ để chứng minh tỷ lệ lỗi thật ≤2%.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): script kiểm cấu trúc, lớp và hình học box; không tự xác định box có ôm sát vật thật, cả người và máy cùng bỏ sót hay người review có làm đúng checklist không.

**Quyết định: Rework**, ngày 2026-11-17. Lý do (đi theo Hold → Rework → Release): 1/5 ảnh QC có lỗi = 20%, vượt ngưỡng 2%; box bicycle ở `img_031` còn quá rộng.
Nếu Rework / Hold, việc cần làm tiếp: sửa box bicycle #414 trong `img_031`, build lại release và tạo mẫu QC mới. Lần validate mới nhất của người học ghi nhận 24 ảnh/482 đối tượng, không còn cảnh báo box thấp hơn 20 px.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Tôi dùng Copilot trong VS Code để rà cấu trúc báo cáo và đối chiếu số liệu với các file kết quả. Phần review ảnh QC do tôi trực tiếp thực hiện; kết quả được ghi trong `submission/qc/qc_review.csv`.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không; tôi không xem pre-label hoặc hỏi AI trước khi hoàn tất gán gold.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows, chạy model bằng CPU, CVAT Docker v2.74.1; ban đầu có lỗi thiếu numpy khi gọi Python ngoài môi trường `.venv`, sau đó chạy bằng Python trong `.venv`.
- Dùng cache `data/cache/` thay vì tự chạy? Không; YOLO26 và YOLOE trên split `all` được chạy bằng CPU (`r05`, `r06` trong `work/runs.csv`).
- Lỗi / bug script đã gặp (nếu có): gọi sai Python environment gây `ModuleNotFoundError: numpy`; đã dùng `.venv\Scripts\python.exe`.
- Một điều bạn làm khác lần sau: quét toàn ảnh trước khi xem nhãn máy và kiểm riêng các box thấp hơn 20 px.
