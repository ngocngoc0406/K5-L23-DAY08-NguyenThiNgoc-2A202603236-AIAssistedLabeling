# Thông tin học viên

- Họ và tên: Nguyễn Thị Ngọc
- MSHV: 2A202603236
- Lớp (L2 / L3 / khác): K5-L23
- Tài khoản GitHub: ngocngoc0406
- Repo bài nộp (URL): https://github.com/ngocngoc0406/K5-L23-DAY08-NguyenThiNgoc-2A202603236-AIAssistedLabeling
- Commit chốt bài (hash): f1d72e65eb736c256ea37d84a0a08779459a4787
- Máy: Windows 11, GPU NVIDIA RTX 3050 Laptop 4 GB (CUDA)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 438 / 7 / 20 (mẫu số: 478 box máy); added: 16 (mẫu số: 461 đối tượng)
- So với gold: precision / recall của **máy** 23/56 = 41.1% / 23/37 = 62.2%
- Bắt bẫy: 8/12 = 66.7% (học viên tự duyệt theo checklist 4 bước, xem REPORT mục 3)
- Thời gian: gán tay 2.5 phút/ảnh; duyệt có pre-label: 1.17 phút/ảnh (28 phút / 24 ảnh, tiết kiệm ~53% thời gian)
- QC vòng 2 (học viên kiểm tra độc lập 10 ảnh sau rework): 0/10 ảnh lỗi, 0/186 đối tượng lỗi; cận trên Wilson 95% là 28%, chưa chứng minh được ngưỡng 2%
- **Quyết định:** REWORK — bắt bẫy 8/12 = 66.7% < 80%; cần tiếp tục rà soát kỹ các ca khó theo checklist trước khi Release
