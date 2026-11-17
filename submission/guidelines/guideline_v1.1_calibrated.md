# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603199

> Bổ sung cho guideline v1.0; các quyết định dưới đây được ghi trong `submission/qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 5 | Kính/cửa sổ và người nhìn thấy qua cửa có thuộc mask xe không? | Kính và khung cửa thuộc mask phương tiện. Người nhìn thấy qua kính/cửa là instance `person` riêng, không gộp vào mask xe. | `img_013`, `img_004`, `img_017` | CAL-03 |

## 2. Ca khó mới

| Ảnh | Tình huống | Máy làm gì | Quyết định | Bổ sung vào guideline |
|---|---|---|---|---|
| `img_013` | YOLO26 nhận tuk-tuk là `car` | Box car bao xe ba bánh; SAM không sửa được nhãn lớp | Gán `three_wheeler` theo định nghĩa lớp, không dựa riêng vào tên lớp hoặc confidence của COCO | Mục 4.3: nếu không xác định được số bánh, tick `can_review` |
| `img_013`, `img_028` | Bẫy `missing` bị lọt trong lượt review đầu | Pre-label không có box cho vật cần gán | Ghi nhận kết quả lượt đầu là 10/12; thêm bước quét toàn khung hình để tìm vật không có box | Mục 4.1: quét ảnh trước khi chấp nhận pre-label |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt cách xử lý kính/người trong mask; thêm bước quét ảnh và nhắc kiểm tra lớp tuk-tuk bằng định nghĩa taxonomy | CAL-01, CAL-02, CAL-03 |
