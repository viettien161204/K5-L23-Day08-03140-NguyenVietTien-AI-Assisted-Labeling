# Thông tin học viên

- Họ và tên: Nguyễn Viết Tiến
- MSHV: 03140
- Lớp (L2 / L3 / khác): K5-L23
- Tài khoản GitHub: viettien161204
- Repo bài nộp (URL): [[https://github.com/hathienphu218/K5-L23-Day08-03140-NguyenVietTien-AI-Assisted-Labeling](https://github.com/viettien161204/K5-L23-Day08-03140-NguyenVietTien-AI-Assisted-Labeling.git)](https://github.com/viettien161204/K5-L23-Day08-03140-NguyenVietTien-AI-Assisted-Labeling.git)
- Commit chốt bài (hash): chốt hoàn thiện Part 6 : 97f4d7da0395b18f1e924de75edc1a4b4dbac52f
- Máy: Windows 11 64-bit, CPU
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 590 (84.5%) / 22 (3.2%) / 74 (10.6%) (mẫu số: 698 box máy); added: 23 (3.6%) (mẫu số: 637 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: 55.1% (49/89) / 83.1% (49/59) (mẫu số: 59 đối tượng gold)
- Bắt bẫy: 10/12 = 83.3% (lọt 2 bẫy: img_025 extra, img_006 loose)
- Thời gian: gán tay 5.75 phút/ảnh (23 phút / 4 ảnh gold); duyệt có pre-label 3.0 phút/ảnh (72 phút / 24 ảnh batch)
- Lỗi QC: 0.0% theo ảnh (0/8), 0.0% theo đối tượng (0/242)
- **Quyết định:** Release — lý do một dòng: Không có khoảng trống guideline, bắt bẫy đạt 83.3% và tỉ lệ lỗi QC đạt 0.0% <= 2%
