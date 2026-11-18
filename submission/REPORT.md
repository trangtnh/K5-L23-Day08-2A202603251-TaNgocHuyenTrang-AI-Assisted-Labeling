# Báo cáo Day 08 — 2A202603251 - Tạ Ngọc Huyền Trang

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

> Tiến độ: Đã hoàn thành toàn bộ Part 0 đến Part 6 (gán gold, tour model, sweep threshold, merge & review 24 ảnh batch, mask SAM 2.1, calibration, build release, QC sample và báo cáo).

## 1. Model và recipe (Part 2–3)

Thông số và số lượng đầu ra lấy từ `submission/model_log.csv` (r01–r06); ngưỡng lấy từ `submission/qc/threshold_sweep.md` và `.csv`. Các lượt model chạy trên MPS, kích thước ảnh đầu vào `imgsz=1024`.

| Lớp | Model + cỡ | Ngưỡng dùng cho bước gộp tiếp theo | Căn cứ |
|---|---|---|---|
| person / motorbike / car | YOLO26m | 0.30 (mặc định) | r01, r02 thử trên pilot; r05 tạo dự đoán toàn bộ ảnh. Quét ngưỡng không đạt recall mục tiêu 90%: cao nhất lần lượt 23/27, 11/13, 7/8 vật gold. Giữ mặc định theo script; không coi đây là ngưỡng đã đạt mục tiêu. |
| bicycle / bus / truck | YOLO26m | 0.30 (mặc định) | Gold chỉ có lần lượt 4, 1, 0 vật, dưới 5 mẫu/lớp; chưa đủ cơ sở chọn ngưỡng riêng. Nguồn dự đoán: r05. |
| three_wheeler | YOLOE-26s, trọng số `yoloe-26s-seg.pt`; prompt `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15 | r03 thử trên pilot, r06 tạo dự đoán toàn bộ ảnh. Ở ngưỡng 0.15 trên gold: TP=7, FP=2, FN=0; recall 7/7 = 100%, precision 7/9 = 77.8%. |

### Kết quả chạy Part 2

| Run | Thử nghiệm | Số ảnh | Box | Mask | Thời gian suy luận/ảnh |
|---|---|---:|---:|---:|---:|
| r01 | YOLO26m, conf 0.25 | 8 pilot | 215 | 0 | 0.51 s |
| r02 | YOLO26m, conf 0.50 | 8 pilot | 139 | 0 | 0.25 s |
| r03 | YOLOE-26s, three_wheeler, conf 0.25 | 8 pilot | 8 | 0 | 0.30 s |
| r04 | YOLO26m → SAM 2.1 base, conf 0.30 | 2 ảnh: img_004, img_013 | 16 | 16 | 3.17 s |
| r05 | YOLO26m, conf 0.05, không xuất viz | 36 | 1669 | 0 | 0.14 s |
| r06 | YOLOE-26s, three_wheeler, conf 0.05, không xuất viz | 36 | 125 | 0 | 0.08 s |

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 496 | 496/519 = 95.6% | 519 box máy |
| edited | 10 | 10/519 = 1.9% | 519 box máy |
| deleted | 11 | 11/519 = 2.1% | 519 box máy |
| added | 13 | 13/519 = 2.5% | 519 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)?
  Lớp `three_wheeler` là lớp máy làm tệ nhất: có 8 box bị xoá (deleted) do dự đoán sai vị trí hoặc nhầm lẫn với các vật thể ven đường, đồng thời YOLO26 mặc định không nhận diện được xe ba bánh mà thường đoán nhầm thành car hoặc truck (điển hình trên `img_013.jpg`, `img_017.jpg`). Ngoài ra, lớp `person` và `motorbike` ở hậu cảnh xa thường bị máy bỏ sót (sót 5 person và 3 motorbike trên các ảnh đông đúc như `img_005.jpg`, `img_018.jpg`).
- So với gold: precision / recall của **máy**:
  Precision = 43/67 = 64.2% (khoảng tin cậy 95%: 52–75%), Recall = 43/60 = 71.7% (khoảng tin cậy 95%: 59–81%), trên mẫu số 60 đối tượng gán chuẩn trong 4 ảnh gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: Bắt được 1/12 bẫy = 8.3% (khoảng tin cậy 95%: 1–35%).
- Loại nào lọt: 11 bẫy bị lọt gồm:
  - 3/3 bẫy thừa (extra): `img_010.jpg`, `img_009.jpg`, `img_021.jpg`
  - 3/3 bẫy sai nhãn (wrong_class): `img_023.jpg`, `img_026.jpg`, `img_017.jpg`
  - 3/3 bẫy lỏng (loose): `img_008.jpg`, `img_025.jpg`, `img_034.jpg`
  - 2/3 bẫy sót (missing): `img_005.jpg`, `img_027.jpg`
  (Chỉ bắt được 1 bẫy missing).
- Vì sao (bạn đã bỏ bước nào của checklist)?
  Nguyên nhân lọt bẫy là do đã bỏ qua Bước 0 (không ẩn box máy để quét mắt độc lập 15s trước khi bật gợi ý) và Bước 4 (không zoom sát để kiểm tra 4 mép bounding box). Do ảnh hưởng tâm lý tin tưởng máy và mong muốn đẩy nhanh tiến độ, người duyệt đã chấp nhận lướt nhanh các đề xuất sẵn có.
- Accepted cao mà bắt bẫy thấp nói gì về cách bạn duyệt?
  Tỉ lệ accepted lên tới 95.6% đi kèm tỉ lệ bắt bẫy chỉ 8.3% là minh chứng rõ nét cho hiện tượng **Automation Bias** (thiên kiến tự động hoá): người duyệt đóng vai trò xác nhận thụ động thay vì chốt chặn kiểm soát chất lượng chủ động.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 15 phút / 4 ảnh = 3.75 phút/ảnh vs duyệt có pre-label 62 phút / 24 ảnh = 2.58 phút/ảnh.
  Thiên lệch: Tốc độ duyệt nhanh hơn gán tay 1.17 phút/ảnh (tiết kiệm ~31% thời gian), tuy nhiên sự chênh lệch này một phần do người duyệt đã lướt nhanh và bỏ sót bẫy (đánh đổi chất lượng lấy tốc độ).
- Kết luận: **Chỉ đáng dùng có điều kiện**. Pre-label rất hữu ích với các lớp phổ biến rõ ràng như person, motorbike, car ở cự ly gần đến trung bình. Nhưng đối với lớp ba bánh (`three_wheeler`) hoặc các vùng dày đặc, máy không đáng tin cậy nếu không có quy trình kiểm tra chéo nghiêm ngặt.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. *Quy tắc 15 giây mù*: Bắt buộc tắt hiển thị box máy trong 15 giây đầu tiên của mỗi ảnh để tự đếm số lượng người và phương tiện trước khi bật nhãn AI.
  2. *Quy tắc Zoom 200% kiểm tra mép*: Với mỗi ảnh, bắt buộc phóng to ít nhất 3 box ngẫu nhiên lên mức 200% để kiểm tra độ ôm khít của 4 cạnh hộp bẫy lỏng.
  3. *Quy tắc kiểm định bẫy ngẫu nhiên (Quality Gate)*: Cài ngẫu nhiên 10-15% bẫy vào mỗi lô gán nhãn; nếu người duyệt không đạt tỉ lệ bắt bẫy tối thiểu 80% thì từ chối nghiệm thu toàn bộ lô và yêu cầu duyệt lại.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013.jpg | SAM 2.1 base (`sam2.1_b.pt`) | 2 dương, 1 âm | Không gồm người lái | Mask ôm sát thân tuk-tuk, tách người lái thành công nhờ click âm | `tuktuk_img013_sam2.jpg` |
| img_004.jpg | SAM 2.1 base (`sam2.1_b.pt`) | 2 dương, 1 âm | Không gồm người lái | Bám sát thân xe và bánh trước, không lem sang nền đường | `tuktuk_img004_sam2.jpg` |
| img_017.jpg | SAM 2.1 base (`sam2.1_b.pt`) | 2 dương, 1 âm | Không gồm người lái | Tách tốt mui xe và khung kim loại, loại bỏ được cabin lái | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: Quy tắc tách người lái thành đối tượng riêng (4.1.1) và bỏ bóng đổ (4.2.3) áp dụng rất tốt khi dùng click âm (`--neg`). Guideline cần bổ sung rõ quy định về phần kính chắn gió/cửa hở nhìn xuyên thấu khi phân đoạn đa giác.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: CVAT local Docker yêu cầu cấu hình Nuclio server riêng nên việc chạy offline qua script `sam2_points.py` giúp kiểm soát chính xác toạ độ click và xem kết quả nhanh chóng.

## 6. Calibration (Part 6)

- Số quyết định: 3 quyết định được ghi nhận trong `calibration_decisions.csv`. Quan trọng nhất là `CAL-01` (xử lý lỗi máy nhầm xe ba bánh thành car/truck) và `CAL-02` (tiêu chí phân định xe điện có/không bàn đạp).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Quy định rõ mọi xe ba bánh gán `three_wheeler`; xe đạp điện có bàn đạp tính là `bicycle`, không bàn đạp tính là `motorbike`; cụm xe máy ở xa < 20px gộp chung thành một `ignore_region`.
- Nguyên nhân gốc: Guideline v1.0 chưa bao quát hết các phương tiện giao thông đặc thù đô thị châu Á (xe đạp điện, tuk-tuk) kết hợp hạn chế nhãn chuẩn COCO của YOLO.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên với seed 42: `img_003.jpg`, `img_006.jpg`, `img_014.jpg`, `img_031.jpg`, `img_035.jpg`); người kiểm: học viên thực hiện.
- Lỗi theo **đối tượng**: 0/110 = 0.0% · theo **ảnh**: 0/5 = 0.0% (ngưỡng Release ≤ 2%).
- Top lỗi: Không phát hiện lỗi sai nhãn hay sai vị trí lớn trong 5 ảnh mẫu được rút ngẫu nhiên.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Với 0 lỗi trên 5 ảnh mẫu, cận trên khoảng tin cậy 95% Wilson của tỉ lệ ảnh lỗi thật lên tới **43%**. Cỡ mẫu 5 ảnh quá nhỏ để chứng minh mặt thống kê rằng toàn bộ tập dữ liệu đạt ngưỡng sai số dưới 2%.
- Script kiểm được / không kiểm được: Script kiểm tra được tính hợp lệ của định dạng JSON và bounding box hợp lệ; nhưng không tự kiểm tra được ngữ cảnh thực tế (vật bị che khuất hay độ khít vi mô).

**Quyết định: REWORK**, ngày 2026-11-18.
- Lý do: Theo cây quyết định Hold → Rework → Release: Mặc dù mẫu kiểm định QC 5 ảnh không có lỗi bề mặt (0%), nhưng chỉ số đo lường thiên kiến bắt bẫy thực tế chỉ đạt 1/12 = 8.3% (thấp hơn nhiều so với ngưỡng yêu cầu ≥ 80%), đồng thời tỉ lệ chấp nhận nhãn máy (accepted) đạt 95.6%. Điều này chứng minh lô duyệt 24 ảnh chưa đủ tin cậy và đã bị chi phối nặng nề bởi Automation Bias.
- Việc cần làm tiếp khi Rework:
  1. Duyệt lại toàn bộ lô 24 ảnh batch trên CVAT theo đúng quy trình 4 bước.
  2. Bắt buộc thực hiện Bước 0 (tắt nhãn máy 15s) và phóng to kiểm tra kỹ các đối tượng ba bánh và góc khuất.
  3. Kiểm tra lại 11 ảnh đã bị lọt bẫy để sửa dứt điểm các lỗi thừa, thiếu và sai nhãn.

## 8. Ghi chú cá nhân

### Theo dõi Part 1
- File: `submission/labels/gold_manual.json`.
- Lệnh kiểm tra: `python3 scripts/validate_labels.py submission/labels/gold_manual.json --expect gold`.
- Kết quả kiểm tra cấu trúc: **4 ảnh, 74 đối tượng, OK**.
- Cảnh báo về box bicycle dưới 20px trên `img_001.jpg`: Giữ nguyên để phản ánh trung thực kết quả gán tay độc lập ban đầu của học viên làm đối chứng.
- Thời gian gán tay thật: 15 phút cho 4 ảnh (ghi nhận trong `time_log.csv`).
- Toàn bộ quá trình gán gold hoàn toàn không sử dụng AI hỗ trợ.
