# Báo cáo Day 08 — 03140 - Nguyễn Viết Tiến

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26-m | 0.10 | Đạt recall 91% (>= 90%) trên gold, precision 41% (dẫn `r01`, `r05`, `threshold_sweep.md`) |
| bicycle | YOLO26-m | 0.20 | Đạt recall 100% (>= 90%) trên gold, precision 71% (dẫn `r01`, `r05`, `threshold_sweep.md`) |
| motorbike | YOLO26-m | 0.30 (mặc định) | Không ngưỡng nào <= 0.5 đạt recall 90% (tối đa 80%, 15 vật gold); dùng mặc định 0.3 |
| car | YOLO26-m | 0.30 (mặc định) | Không ngưỡng nào <= 0.5 đạt recall 90% (tối đa 88%, 8 vật gold); dùng mặc định 0.3 |
| bus | YOLO26-m | 0.30 (mặc định) | Chỉ 0 vật trong ảnh gold (< 5): mẫu quá ít, không chọn ngưỡng riêng, dùng mặc định |
| truck | YOLO26-m | 0.30 (mặc định) | Chỉ 1 vật trong ảnh gold (< 5): mẫu quá ít, không chọn ngưỡng riêng, dùng mặc định |
| three_wheeler | YOLOE-26s | 0.15 | Đạt recall 100% (>= 90%) trên gold, precision 78% (dẫn `r03`, `r06`, `threshold_sweep.md`) |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.1,bicycle=0.2,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: Chọn **SAM 2.1 Base (`sam2.1_b.pt`)**. Lý do: Đây là phiên bản cân bằng tối ưu giữa độ chính xác của đường viền mask, tốc độ suy luận và dung lượng bộ nhớ (~161 MB). SAM 2.1 phân đoạn vượt trội so với MobileSAM (SAM 1) ở các chi tiết phức tạp, đồng thời không yêu cầu quyền truy cập gated model như SAM 3, chạy ổn định cả trên GPU và CPU.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào?
  Lỗi phân loại sai danh mục do giới hạn closed-set (YOLO26 chỉ có 80 lớp COCO nên gọi xe ba bánh tuk-tuk, xích lô, xe ba gác là `car` hoặc `truck`). Lỗi này thuộc về không gian khái niệm của mô hình nên việc tăng hay giảm ngưỡng `conf` không thể giải quyết được. Cách sửa: Phải kết hợp mô hình open-vocabulary (YOLOE-26s) với prompt văn bản tùy biến (`three_wheeler`), sau đó chạy thuật toán hợp nhất `merge_prelabels.py` ưu tiên gán đè nhãn `three_wheeler` lên box `car`/`truck` khi độ trùng khớp IoU $\ge 0.6$.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - *Điểm cao mà sai:* Ở `img_002.jpg`, YOLO26 phát hiện chiếc tuk-tuk và gán nhãn `car` với điểm tin cậy rất cao (~0.78); ở `img_025.jpg` có box ma gán thừa ở vùng bóng râm với conf > 0.6.
  - *Điểm thấp mà đúng:* Ở `img_011.jpg` và `img_019.jpg`, các xe máy ở xa chỉ đạt conf 0.15–0.22 nhưng thực tế là đối tượng hợp lệ cần gán nhãn.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Việc quét và tối ưu ngưỡng được thực hiện trên chính tập dữ liệu gold (in-sample evaluation) gồm 4 ảnh và 59 đối tượng. Do kích thước mẫu nhỏ và đánh giá trên cùng tập chuẩn, precision/recall của mô hình trên gold mang tính chất lạc quan so với thực tế triển khai trên tập dữ liệu lớn chưa nhìn thấy.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 590 | 84.5% (590/698) | 698 box máy |
| edited | 22 | 3.2% (22/698) | 698 box máy |
| deleted | 74 | 10.6% (74/698) | 698 box máy |
| added | 23 | 3.6% (23/637) | 637 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)?
  Lớp `three_wheeler` là lớp máy làm tệ nhất: có 15 box bị xóa (`deleted`) trên tổng số 40 box dự đoán (tỉ lệ xóa lên tới 37.5%) và 6 box phải chỉnh sửa (`edited`). Nguyên nhân do YOLOE có xu hướng sinh box lỏng lẻo, dễ nhầm lẫn các cụm xe máy hoặc xe ô tô ở xa thành xe ba bánh, đồng thời dính liền người lái/hàng hóa phía sau vào thân xe (như ở `img_002.jpg` và `img_032.jpg`). Ngoài ra, lớp `person` có số lượng xóa tuyệt đối lớn nhất (45 box) do ngưỡng thấp 0.10 bắt nhầm nhiều nhiễu nền/biển quảng cáo.
- So với gold: precision / recall của **máy**: Precision đạt **55.1%** (49/89, KTC 95%: 45–65%), Recall đạt **83.1%** (49/59, KTC 95%: 72–91%), với mẫu số là 59 đối tượng trong 4 ảnh gold (TP: 49, FP: 40, FN: 10).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **10/12 = 83.3%** (KTC 95%: 55–95%), vượt ngưỡng yêu cầu tối thiểu (80%).
  - Các loại bẫy: `wrong_class` bắt được 3/3; `missing` bắt được 3/3; `extra` bắt được 2/3; `loose` bắt được 2/3.
  - Bẫy bị lọt gồm 2 ca: `img_025.jpg` (loại `extra`) và `img_006.jpg` (loại `loose`).
  - Vì sao lọt: Ở `img_025.jpg`, box ma nằm ở khu vực khuất bóng râm và có hình dạng tương tự một vật thể thật, mắt bị cuốn theo gợi ý có sẵn của máy nên bỏ qua bước quét box thừa (Bước 2). Ở `img_006.jpg`, box bị lỏng nhẹ viền ngoài (~5-8px), tâm lý chấp nhận viền có sẵn khiến người duyệt không kéo chỉnh lại biên sát vật (Bước 4).
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt?
  Tỉ lệ accepted đạt 84.5% đi kèm tỉ lệ bắt bẫy 83.3% (bắt được 10/12 bẫy và sửa đổi 96 box máy) chứng minh người duyệt có thực hiện quy trình kiểm tra thực chất, không bị phụ thuộc hoàn toàn vào gợi ý máy. Tuy nhiên việc lọt 2 bẫy cho thấy thiên kiến tự động hóa (automation bias) vẫn xuất hiện ở những lỗi tinh vi khó quan sát bằng mắt thường khi duyệt số lượng ảnh lớn.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): Gán tay **5.75 phút/ảnh** (23 phút cho 4 ảnh gold) so với duyệt có pre-label **3.0 phút/ảnh** (72 phút cho 24 ảnh batch). Pre-label giúp tiết kiệm khoảng 47.8% thời gian trên mỗi ảnh. Thiên lệch: 4 ảnh gold được gán tay trước nên người thực hiện đã quen với ngữ cảnh phố xá; nếu gán tay toàn bộ 24 ảnh từ đầu thì thời gian trung bình gán tay thực tế còn cao hơn (~7-8 phút/ảnh).
- Kết luận: **CÓ ĐÁNG DÙNG**, mang lại hiệu quả vượt bậc đối với các lớp phổ biến có độ chính xác cao (`person`, `motorbike`, `car`). Đối với các lớp xe địa phương (`three_wheeler`), pre-label vẫn hữu ích trong việc gợi ý vị trí nhưng bắt buộc người duyệt phải can thiệp kéo lại viền box và kiểm tra kỹ lớp nhãn.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. *Quy tắc 1 (Bước 0 - Quét mù trước khi xem máy):* Ở 100% các ảnh, luôn ẩn toàn bộ layer nhãn máy trong 15 giây đầu để tự đếm số lượng người và phương tiện chính, sau đó mới bật nhãn máy để đối chiếu.
  2. *Quy tắc 2 (Zoom kiểm tra viền vật lớn):* Phóng to $\ge 200\%$ kiểm tra 100% các box của lớp `three_wheeler` và các box kích thước lớn để đảm bảo không bị dính người lái hoặc hàng hóa rời rạc.
  3. *Quy tắc 3 (Nghỉ ngơi theo lô ảnh):* Cứ sau mỗi 8-10 ảnh duyệt liên tục, dừng lại nghỉ 3-5 phút nhằm phục hồi độ nhạy thị giác, tránh hiện tượng mỏi mắt dẫn đến duyệt lướt bỏ sót bẫy.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 click (2 pos: 400,480; 600,110; 1 neg: 720,300) | Đã tách rời người lái (không gồm) | Nếu không có click âm, mask sẽ nuốt trọn cả người lái bên trong; có click âm tách rất chuẩn (317,255 px, 45.4% ảnh) | `masks/tuktuk_img013.jpg` |
| `img_004.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 1 box prompt + 1 click pos (654,412) | Không gồm người đứng cạnh | Mask bám sát thùng xe ba gác (93,444 px, 13.4% ảnh), tách biệt hoàn toàn với người và nền phía sau | `masks/bagac_img004.jpg` |
| `img_017.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 1 box prompt + 2 click pos + 1 click neg (500,420) | Tách riêng người lái trong cabin | Mask phủ trọn vẹn kết cấu xe tuk-tuk (86,775 px, 12.4% ảnh), đường viền sắc nét, khoét rỗng cabin người lái | `masks/tuktuk_img017.jpg` |

- Đối chiếu guideline mục 4.2:
  - *Quy tắc giúp ích:* Quy tắc "người ngồi trên xe là person riêng, xe là vật riêng" và "hai vật chạm nhau tách polygon riêng" định hướng rõ ràng việc phải đặt negative click để loại bỏ người lái khỏi mask phương tiện. Quy tắc "kính chắn gió thuộc xe" giúp SAM giữ lại toàn bộ phần cabin trước của xe tuk-tuk.
  - *Quy tắc còn thiếu:* Guideline v1.0 chưa làm rõ cách xử lý các khoảng hở / thanh giằng kim loại xuyên thấu của mui xe tuk-tuk (lỗ thủng nhỏ dưới 5% diện tích có cần khoét rỗng không), khiến mô hình SAM đôi khi bao phủ luôn cả phần nền đường nhìn xuyên qua khung xe.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: Khi sử dụng công cụ Interactor trong CVAT, giao diện tương tác trực quan cho phép thấy ngay mask biến đổi sau mỗi click chuột trái (thêm) hoặc chuột phải (bớt), giúp điều chỉnh dễ dàng hơn so với việc ước lượng toạ độ pixel trong script dòng lệnh. Tuy nhiên SAM 2.1 chạy offline với prompt kết hợp (box + point) cho chất lượng mask ổn định và bao quát trọn vẹn vật thể lớn.

## 6. Calibration (Part 6)

- Số quyết định: 4 quyết định (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-01`): Quy tắc chuẩn hóa xe ba bánh có mui (tuk-tuk/ba gác) luôn thuộc lớp `three_wheeler` và tách rời người lái thành `person` riêng biệt cả trong box lẫn mask SAM.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Chốt định nghĩa xe đạp điện (nếu có bàn đạp tính là `bicycle`, không có bàn đạp tính là `motorbike` - `CAL-02`), làm rõ tiêu chí gom cụm xe máy ở xa bằng `ignore_region` (`CAL-03`), và siết chặt dung sai ôm sát viền $\le 2$ px sau khi rút kinh nghiệm từ bẫy lọt `img_006.jpg` (`CAL-04`).
- Nguyên nhân gốc: Phân định rõ giữa lỗi do guideline mơ hồ (chưa quy định xe điện, tiêu chí ignore_region) cần sửa vào văn bản guideline v1.1, và lỗi do mô hình AI closed-set không nhận diện được xe ba bánh địa phương.

## 7. QC và quyết định (Part 6)

- Mẫu: 8 ảnh trong tập batch (gồm 3 ảnh ca khó `can_review` nhiều nhất + 5 ảnh ngẫu nhiên, seed 42); người kiểm: học viên 03140 (**hạn chế**: người duyệt cũng chính là người kiểm tra).
- Lỗi theo **đối tượng**: 0/242 = 0.0% · theo **ảnh**: 0/8 = 0.0% (đáp ứng ngưỡng Release $\le 2\%$ theo đối tượng và ảnh).
- Top lỗi: Không ghi nhận lỗi vi phạm sau khi đã hoàn thiện review và hiệu chỉnh.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Với 0 ảnh lỗi trên 8 ảnh mẫu, cận trên khoảng tin cậy Wilson 95% của tỉ lệ ảnh lỗi thực tế là **32%**. Cỡ mẫu 8 ảnh là tương đối nhỏ nên ngưỡng 2% chưa thể được chứng minh chắc chắn về mặt ý nghĩa thống kê; đây là giới hạn thực tế cần nêu rõ trong báo cáo.
- Script kiểm được / không kiểm được: Script kiểm tra tự động được định dạng file, schema nhãn, dải toạ độ hợp lệ, số lượng đối tượng và tỷ lệ accepted/edited/deleted; script không thể kiểm tra được box có thực sự ôm sát vật thể trong ảnh hay không, không phát hiện được trường hợp cả người duyệt lẫn máy cùng bỏ sót vật, và không xác minh được người duyệt có tuân thủ quy trình quét ảnh hay không.

**Quyết định: Release**, ngày 2026-11-17. Lý do: Đi theo cây quyết định Hold → Rework → Release:
1. Không có khoảng trống guideline chưa giải quyết (guideline gap = 0, toàn bộ 4 ca khó đã được chốt quy tắc trong guideline v1.1 và calibration_decisions.csv) và không có ảnh hỏng dữ liệu (data problem = 0) $\rightarrow$ Không thuộc diện Hold.
2. Tỉ lệ ảnh lỗi trong mẫu QC đạt 0.0% $\le 2\%$, tỉ lệ bắt bẫy đạt 83.3% $\ge 80\%$, không xuất hiện lỗi hệ thống lặp lại $\rightarrow$ Không thuộc diện Rework.
3. Toàn bộ các tiêu chí nghiệm thu đều đạt chuẩn $\rightarrow$ Đủ điều kiện **Release**.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Sử dụng AI Assistant để giải thích các khái niệm trong label contract, hỗ trợ sinh cú pháp lệnh shell, phân tích log thực nghiệm và định dạng báo cáo markdown; toàn bộ số liệu, quan sát lỗi và quyết định đều được kiểm tra đối chiếu trực tiếp từ file log và kết quả sinh ra của các script.
- Gold có bị nhiễm không: Không bị nhiễm; 4 ảnh gold được hoàn thành gán tay độc lập từ đầu trước khi chạy bất kỳ mô hình AI nào.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11 64-bit, CPU, Docker CVAT v2.81.0; môi trường chạy cục bộ ổn định.
- Dùng cache `data/cache/` thay vì tự chạy? Không dùng cache; toàn bộ các lần chạy prelabel từ r01 đến r06 và SAM đều được tự chạy trực tiếp trên máy.
- Lỗi / bug script đã gặp (nếu có): Đã khắc phục lỗi đường dẫn PowerShell trên Windows và điều chỉnh toạ độ điểm click cho SAM 2.1 kết hợp box prompt để đạt mask chính xác.
- Một điều bạn làm khác lần sau: Luôn phóng to kiểm tra viền box của các vật thể phức tạp ngay từ đầu để tránh bỏ lọt bẫy lỏng biên.
