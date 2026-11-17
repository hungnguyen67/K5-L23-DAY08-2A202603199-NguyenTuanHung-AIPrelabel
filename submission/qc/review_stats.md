# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 518; đối tượng sau duyệt: 467; ngưỡng accepted: IoU >= 0.9
- Đã **loại 26 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 364 | 364/518 = 70.3% | 518 box máy |
| edited (sửa) | 71 | 71/518 = 13.7% | 518 box máy |
| deleted (xoá) | 80 | 80/518 = 15.4% | 518 box máy |
| added (vẽ thêm) | 32 | 32/467 = 6.9% | 467 đối tượng sau duyệt |

Trong số deleted + added, **11 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 21.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 232 | 19 | 17 | 4 |
| bicycle | 3 | 2 | 3 | 2 |
| motorbike | 82 | 35 | 19 | 12 |
| car | 36 | 6 | 10 | 0 |
| bus | 2 | 0 | 1 | 1 |
| truck | 3 | 0 | 2 | 7 |
| three_wheeler | 6 | 9 | 28 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 42 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 37 | 29 | 5 | 56.1% (KTC 95%: 44–67%) | 88.1% (KTC 95%: 75–95%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - loose: 3/3
  - wrong_class: 3/3
  - missing: 1/3
  - lọt: img_013.jpg (missing), img_028.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
