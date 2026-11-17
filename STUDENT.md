# Thông tin học viên

- Họ và tên: Võ Duy Thành
- MSHV: 2A202603211
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: 11thanh2233
- Repo bài nộp (URL): https://github.com/11thanh2233/K5-L23-DAY08-2A202603211-VoDuyThanh-AIPrelabel.git
- Commit chốt bài (hash): 4a9e4ad
- Máy: Windows 10 CPU không GPU
- CVAT (Docker): 2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 403 / 36 / 149 = 64.8% / 5.8% / 24.0% (mẫu số: 622 box máy); added: 56 = 11.3% (mẫu số: 495 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 41/66 = 62.1% / 41/48 = 85.4% (4 ảnh gold, 48 đối tượng)
- Bắt bẫy: 8/12 = 66.7%
- Thời gian: gán tay 2.0 phút/ảnh (8 phút / 4 ảnh); duyệt có pre-label 1.67 phút/ảnh (40 phút / 24 ảnh)
- Lỗi QC: 40% theo ảnh (2/5), 5.6% theo đối tượng (5/89)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 2/5 = 40% > 2%, lỗi "thừa" lặp lại 3/5 lỗi, bắt bẫy 8/12 = 67% < 80%
