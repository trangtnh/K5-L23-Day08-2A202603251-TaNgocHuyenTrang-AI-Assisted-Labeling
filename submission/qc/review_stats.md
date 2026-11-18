# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 519; đối tượng sau duyệt: 519; ngưỡng accepted: IoU >= 0.9
- Đã **loại 28 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 496 | 496/519 = 95.6% | 519 box máy |
| edited (sửa) | 10 | 10/519 = 1.9% | 519 box máy |
| deleted (xoá) | 11 | 11/519 = 2.1% | 519 box máy |
| added (vẽ thêm) | 13 | 13/519 = 2.5% | 519 đối tượng sau duyệt |

Trong số deleted + added, **2 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 11.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 264 | 5 | 1 | 5 |
| bicycle | 8 | 0 | 0 | 1 |
| motorbike | 131 | 3 | 1 | 3 |
| car | 51 | 0 | 1 | 1 |
| bus | 3 | 0 | 0 | 0 |
| truck | 6 | 0 | 0 | 2 |
| three_wheeler | 33 | 2 | 8 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 60 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 43 | 24 | 17 | 64.2% (KTC 95%: 52–75%) | 71.7% (KTC 95%: 59–81%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 1/12 = 8.3% (KTC 95%: 1–35%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 0/3
  - wrong_class: 0/3
  - missing: 1/3
  - loose: 0/3
  - lọt: img_010.jpg (extra), img_023.jpg (wrong_class), img_008.jpg (loose), img_025.jpg (loose), img_026.jpg (wrong_class), img_009.jpg (extra), img_021.jpg (extra), img_017.jpg (wrong_class), img_034.jpg (loose), img_005.jpg (missing), img_027.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
