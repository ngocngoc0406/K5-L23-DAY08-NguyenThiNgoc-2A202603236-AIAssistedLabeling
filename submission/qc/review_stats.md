# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 478; đối tượng sau duyệt: 461; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 438 | 438/478 = 91.6% | 478 box máy |
| edited (sửa) | 7 | 7/478 = 1.5% | 478 box máy |
| deleted (xoá) | 20 | 20/478 = 4.2% | 478 box máy |
| added (vẽ thêm) | 16 | 16/461 = 3.5% | 461 đối tượng sau duyệt |

Trong số deleted + added, **4 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 12.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 257 | 1 | 5 | 1 |
| bicycle | 5 | 0 | 3 | 2 |
| motorbike | 125 | 1 | 0 | 4 |
| car | 37 | 0 | 1 | 1 |
| bus | 2 | 0 | 1 | 2 |
| truck | 4 | 0 | 1 | 1 |
| three_wheeler | 8 | 5 | 9 | 5 |

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 2/3
  - extra: 3/3
  - loose: 1/3
  - missing: 2/3
  - lọt: img_009.jpg (wrong_class), img_005.jpg (loose), img_035.jpg (missing), img_027.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
