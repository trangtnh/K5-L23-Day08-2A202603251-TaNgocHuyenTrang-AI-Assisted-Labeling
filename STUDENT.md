# Thông tin học viên

- Họ và tên: Tạ Ngọc Huyền Trang
- MSHV: 2A202603251
- Lớp (L2 / L3 / khác): K5-L23
- Tài khoản GitHub: trangtnh
- Repo bài nộp (URL): https://github.com/trangtnh/K5-L23-Day08-2A202603251-TaNgocHuyenTrang-AI-Assisted-Labeling
- Commit chốt bài (hash): main
- Máy: macOS, kiến trúc arm64 (Apple Silicon); hỗ trợ tăng tốc MPS
- CVAT (Docker): CVAT v2.74.1 chạy local qua Docker

> Tiến độ: Đã hoàn thành toàn bộ Part 0 đến Part 6. File gold manual, batch review, mask SAM, calibration guideline và QC report đã được sinh và kiểm định đầy đủ.

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 496/519 = 95.6% / 10/519 = 1.9% / 11/519 = 2.1% (mẫu số: 519 box máy); added: 13/519 = 2.5% (mẫu số: 519 đối tượng sau duyệt)
- So với gold: precision 43/67 = 64.2% / recall 43/60 = 71.7% (mẫu số: 60 đối tượng gold)
- Bắt bẫy: 1/12 = 8.3% (bẫy lọt: 11 bẫy, phản ánh thiên kiến tự động hoá)
- Thời gian: gán tay 3.75 phút/ảnh (15 phút / 4 ảnh); duyệt có pre-label 2.58 phút/ảnh (62 phút / 24 ảnh)
- Lỗi QC: 0.0% theo ảnh (0/5), 0.0% theo đối tượng (0/110)
- **Quyết định:** REWORK — lý do một dòng: Tỉ lệ bắt bẫy đạt 8.3% < 80% do ảnh hưởng của Automation Bias khi duyệt máy.
