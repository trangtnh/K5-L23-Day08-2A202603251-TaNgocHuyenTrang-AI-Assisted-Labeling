# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603251 - Tạ Ngọc Huyền Trang

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 và thêm quy tắc mới rút ra từ lỗi của máy. Mỗi ca khó có ảnh ví dụ (`img_0xx`) và khớp dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: bicycle hay motorbike? | Xe có bàn đạp trợ lực tính là bicycle; xe không bàn đạp, sàn để chân phẳng hoặc dáng scooter điện tính là motorbike | img_002.jpg | CAL-02 |
| 2 | Khối xe máy đỗ dày đặc ở xa: từng xe hay ignore_region? | Cụm ≥ 5 xe nhỏ ở xa có chiều cao < 20 px gom chung một box ignore_region để tránh làm nhiễu mô hình | img_005.jpg | CAL-03 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013.jpg | Xe tuk-tuk và xe ba gác chở đồ | YOLO26 nhận nhầm thành car hoặc truck | Đổi lại nhãn three_wheeler, dùng YOLOE nhận diện lớp này | Mục 4.3: Mọi phương tiện 3 bánh gán three_wheeler, không gán car hay truck |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | — |
| v1.1 | Chốt phân loại xe điện có/không bàn đạp (CAL-02), quy tắc gom cụm xe nhỏ ignore_region < 20px (CAL-03), chuẩn hoá nhãn three_wheeler tách biệt với car/truck (CAL-01) | Hiệu chỉnh sau khi chạy model, review 24 ảnh và bắt lỗi bẫy máy (CAL-01, CAL-02, CAL-03) |
