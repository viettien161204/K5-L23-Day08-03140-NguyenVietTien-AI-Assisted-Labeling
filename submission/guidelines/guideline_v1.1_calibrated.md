# Label contract — Phố châu Á — BỔ SUNG v1.1 — 03140 - Nguyễn Viết Tiến

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Phân biệt dựa trên cơ cấu truyền động: Xe đạp điện có bàn đạp trợ lực (pedals) nhìn thấy được gán `bicycle`; xe máy điện không có bàn đạp, vận hành hoàn toàn bằng tay ga cơ giới gán `motorbike`. | `img_004.jpg` | CAL-02 |
| 2 | Khối xe máy đỗ dày đặc ở cự ly gần và trung bình: từng xe hay `ignore_region`? | Xe máy ở cự ly gần nhìn rõ khung viền cá thể vẫn gán từng box riêng; nếu cụm xe đỗ quá sát nhau phía xa, bị che khuất > 80% hoặc chiều cao vật thể < 20 px thì gộp toàn bộ cụm thành một `ignore_region` duy nhất. | `img_018.jpg` | CAL-03 |
| 3 | Hàng nhô ra: xe chở tre, rổ nhô xa: box xe có gồm hàng không? | Hàng hoá gắn cố định trên giỏ/giá xe tính vào box xe. Hàng nhô ra khỏi xe quá 20% chiều dài xe thì box xe chỉ ôm phần thân xe, không mở rộng ôm toàn bộ phần nhô ra. | `img_004.jpg` | CAL-02 |
| 4 | Người ngồi trong xe ba bánh có mui che: ngưỡng còn thấy ≥ 20%? | Giữ nguyên quy tắc diện tích nhìn thấy ≥ 20%, nhưng làm rõ: nếu chỉ thấy một phần cánh tay hoặc đỉnh đầu (< 20%) thì không tạo box `person` riêng để tránh gây nhiễu cho detector. | `img_013.jpg` | CAL-01 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg` | Xe tuk-tuk có mui che chở người lái và khách | YOLO26 gọi là `car`; SAM tạo mask gộp cả người lái vào thân xe | Gán lớp `three_wheeler` cho xe; tách riêng người lái thành box `person`. Khi chạy SAM bắt buộc thêm negative click lên người lái để tách mask xe | Bổ sung mục 2 & 4.2: xe ba bánh địa phương luôn là `three_wheeler`, mask xe không được gồm người lái (CAL-01) |
| `img_006.jpg` | Bounding box bao quanh xe máy bị lỏng viền ngoài (~8 px) | Máy tạo box lỏng; người duyệt bỏ sót do thiên kiến tự động hóa | Kéo lại viền box ôm sát vật thể thật; yêu cầu zoom tối thiểu 200% để kiểm tra 4 cạnh ngoài cùng | Bổ sung mục 4.1.1: siết dung sai cạnh box $\le 2$ px, kiểm tra độ ôm sát viền (CAL-04) |
| `img_014.jpg` | Xe ba gác chở hàng cồng kềnh phía sau | YOLOE tạo box bao trọn cả người kéo lẫn hàng hóa nhô cao | Tách người kéo thành `person`, thân xe gán `three_wheeler`, hàng nhô quá 20% không tính vào xe | Bổ sung mục 2 & 4.1.5: quy tắc phân tách người kéo và hàng chở cồng kềnh (CAL-01) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | Khung tiêu chuẩn ban đầu |
| v1.1 | 1. Phân định rõ xe đạp điện và xe máy điện dựa vào bàn đạp trợ lực.<br/>2. Chuẩn hóa quy tắc xe ba bánh có mui luôn là `three_wheeler` và tách người lái cả ở box lẫn mask SAM.<br/>3. Làm rõ tiêu chí vẽ `ignore_region` cho cụm xe máy đông đúc ở xa.<br/>4. Siết dung sai ôm sát viền $\le 2$ px sau khi phát hiện bẫy loose bị lọt. | Rút ra từ câu hỏi mở v1.0, lỗi máy YOLO26/SAM ở `img_013.jpg`, bẫy lọt ở `img_006.jpg`, và ca khó ở `img_004.jpg`, `img_018.jpg` (CAL-01, CAL-02, CAL-03, CAL-04) |
