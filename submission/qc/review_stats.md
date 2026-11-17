# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 698; đối tượng sau duyệt: 637; ngưỡng accepted: IoU >= 0.9
- Đã **loại 31 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 590 | 590/698 = 84.5% | 698 box máy |
| edited (sửa) | 22 | 22/698 = 3.2% | 698 box máy |
| deleted (xoá) | 74 | 74/698 = 10.6% | 698 box máy |
| added (vẽ thêm) | 23 | 23/637 = 3.6% | 637 đối tượng sau duyệt |

Trong số deleted + added, **5 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 18.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 390 | 5 | 45 | 7 |
| bicycle | 5 | 1 | 3 | 3 |
| motorbike | 120 | 8 | 6 | 9 |
| car | 49 | 2 | 4 | 0 |
| bus | 2 | 0 | 0 | 2 |
| truck | 5 | 0 | 1 | 0 |
| three_wheeler | 19 | 6 | 15 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 59 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 49 | 40 | 10 | 55.1% (KTC 95%: 45–65%) | 83.1% (KTC 95%: 72–91%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 3/3
  - extra: 2/3
  - missing: 3/3
  - loose: 2/3
  - lọt: img_025.jpg (extra), img_006.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
