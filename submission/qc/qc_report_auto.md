# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 10 ảnh (0 ca khó + 10 ngẫu nhiên), 186 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 0/186 = 0.0% | 186 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 0/10 = 0.0% | 10 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 438/478 = 91.6% / 7/478 = 1.5% / 20/478 = 4.2% | 478 box máy |
| Máy: added | 16/461 = 3.5% | 461 đối tượng sau duyệt |
| Bắt lỗi bẫy | 8/12 = 66.7% | 12 bẫy |

## Top 3 loại lỗi
(không ghi nhận lỗi nào)

## Giới hạn thống kê
- 0 ảnh lỗi trong 10 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **28%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- bắt bẫy 67% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
