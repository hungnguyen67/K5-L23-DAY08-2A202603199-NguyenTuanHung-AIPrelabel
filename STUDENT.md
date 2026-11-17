# Thông tin học viên

- Họ và tên: Nguyen Tuan Hung
- MSHV: 2A202603199
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: hungnguyen67
- Repo bài nộp (URL): https://github.com/hungnguyen67/K5-L23-DAY08-2A202603199-NguyenTuanHung-AIPrelabel.git
- Commit chốt bài (hash): 8ae8dbca2fc6fc6921b1ce45ed3f93147ebdcecf
- Máy: Windows; chạy model bằng CPU
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (từ `submission/qc/*`)

- Máy: accepted 364/518 = 70.3%; edited 71/518 = 13.7%; deleted 80/518 = 15.4%; added 32/467 = 6.9%.
- So với gold: precision 37/66 = 56.1%; recall 37/42 = 88.1%.
- Bắt bẫy: 10/12 = 83.3%.
- Thời gian (từ `submission/time_log.csv`): gán tay 25/4 = 6.25 phút/ảnh; duyệt có pre-label 87/24 = 3.63 phút/ảnh.
- Lỗi QC: 20.0% theo ảnh (1/5), 1.0% theo đối tượng (1/96); tôi trực tiếp review 5 ảnh và ghi kết quả trong `qc_review.csv`.
- **Quyết định: Rework** — ảnh có lỗi 1/5 = 20.0%, cao hơn ngưỡng 2%; cần sửa lỗi box ở mẫu QC rồi lấy mẫu kiểm lại.
