# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603236 - Nguyễn Thị Ngọc

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Mỗi ca khó có ảnh ví dụ và khớp một dòng trong `qc/calibration_decisions.csv`.
> *Bản nháp do Claude (AI) soạn từ lỗi máy và kết quả review — xem `REPORT.md` mục 8.*

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày gần camera: từng xe hay `ignore_region`? | Xe ở hàng đầu (box cao ≥ 60 px và thấy ≥ 50% xe) gán từng chiếc; các hàng phía sau chồng lấn không tách được → **một** `ignore_region` bao cả khối | img_026 | CAL-03 |
| 3 | Hàng nhô ra: box xe có gồm hàng không? | Hàng / khay **gắn cố định** nhô ≤ 20% chiều dài xe → **tính vào box xe**; nhô > 20% hoặc chất cao trên thùng → **không** tính | img_031, img_008 | CAL-04 |

Chưa chốt: câu 1 (xe đạp điện — lô này không có ví dụ rõ), câu 4, 5, 6, 7 (không gặp ca đủ rõ để quyết định; giữ nguyên v1.0).

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_012, img_013 | Tuk-tuk có mui, nhìn ngang / sau | YOLO26m gán `car` (img_012: 4/4 tuk-tuk, conf 0.65–0.81) | Luôn `three_wheeler`; không tin nhãn YOLO26 cho xe có mui ba bánh | Mục 2: thêm "máy COCO gọi car/truck → vẫn `three_wheeler`" (CAL-01) |
| img_014 | Quang gánh, thúng hàng treo | YOLOE-26s (prompt three_wheeler) gán `three_wheeler` conf 0.33–0.63 | Không gán lớp nào, xoá box | Mục 2: `three_wheeler` phải có **bánh xe nhìn thấy** (CAL-02) |
| img_010 | Xích lô nhìn từ sau, người đạp che giữa xe | YOLOE cắt xe làm 2 box (trên / dưới); YOLO26 gán bánh sau là `bicycle` | Một box `three_wheeler` ôm mui → bánh; không gán `bicycle` cho bộ phận xích lô | Mục 5: thêm dòng ca khó (CAL-05) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở 2 (cụm xe gần camera) và 3 (hàng nhô); thêm 3 quy tắc: tuk-tuk luôn `three_wheeler`, vật không bánh không phải `three_wheeler`, bộ phận xích lô không gán `bicycle` | CAL-01 … CAL-05 |
