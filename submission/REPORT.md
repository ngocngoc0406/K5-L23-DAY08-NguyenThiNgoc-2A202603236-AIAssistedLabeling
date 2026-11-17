# Báo cáo Day 08 — 2A202603236 - Nguyễn Thị Ngọc

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | Part 4 dùng `person=0.3, bicycle=0.3, motorbike=0.35, car=0.5`, còn lại 0.3 (xem giới hạn bên dưới) | r01 vs r02: tăng 0.25 → 0.5 giảm 215 → 140 box trên 8 ảnh pilot, mất chủ yếu người / xe ở xa; giữ ~0.3 để không sót |
| three_wheeler | YOLOE-26s-seg, prompt từ `taxonomy.json` | 0.3 (sweep trên gold gán tay gợi ý 0.15) | r03: YOLOE tìm đúng tuk-tuk img_002 (0.93) mà YOLO26 (r01) gọi là person/truck; nhưng có box thừa 0.37–0.40 |

- Lệnh `merge_prelabels.py` đã dùng: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.3,bicycle=0.3,motorbike=0.35,car=0.5,three_wheeler=0.3" --conf 0.3 --iou 0.6 --split batch,gold --out work/prelabel/merged_review/pre.coco.json` → 562 box (batch + gold).
- **Phiên bản SAM:** SAM 2.1 base (`sam2.1_b.pt`, r04): GPU RTX 3050 4 GB chạy được (4.7 s/ảnh), mask tốt hơn mobile_sam; SAM 3 cần quyền HuggingFace nên không dùng.
- **Lỗi không sửa được bằng ngưỡng:** YOLO26 gọi tuk-tuk là `car` với conf cao (img_012: cả 4 tuk-tuk trong gold, 0.65–0.81; img_013: `car 0.41`). Đổi ngưỡng chỉ đổi số box, không đổi lớp → lấy `three_wheeler` từ YOLOE, gộp với ưu tiên YOLOE (`merge_prelabels`, IoU 0.6), người duyệt sửa phần còn lại (CAL-01).
- **Điểm cao mà sai / thấp mà đúng:** sai — img_012 tuk-tuk `car 0.81`; img_024 `car 0.81` trên vệt mờ chuyển động (không có xe); img_006 `truck 0.47` là biển hiệu. Đúng mà điểm thấp — img_036 xe máy conf 0.12 khớp gold IoU 0.93; img_020 tuk-tuk bị gọi `car 0.30` nhưng box chuẩn (IoU 0.90).
- **Giới hạn:** (1) ngưỡng ở Part 4 chọn từ một lần sweep trên gold **do máy tạo** (đã bỏ); sweep lại trên gold gán tay (`qc/threshold_sweep.md`) chỉ chọn được `three_wheeler=0.15`, các lớp khác không đạt recall 90% ở ngưỡng ≤ 0.5 nên dùng mặc định 0.3 — Part 4 chưa chạy lại với chuỗi mới; (2) chỉ 4 ảnh gold, chọn và chấm trên cùng dữ liệu → số đo lạc quan; bus / truck có 0 vật, bicycle 4 vật trong gold → không chọn ngưỡng riêng.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 438 | 91.6% | 478 box máy (đã loại 25 đối tượng quanh bẫy) |
| edited | 7 | 1.5% | 478 box máy |
| deleted | 20 | 4.2% | 478 box máy |
| added | 16 | 3.5% | 461 đối tượng sau duyệt |

- **Lớp tệ nhất: `three_wheeler`** — accepted 8, edited 5, deleted 9, added 5 (chỉ 8/22 box máy giữ nguyên). Lỗi: cắt một xe làm hai box (img_010, img_013), gộp hai xe vào một box (img_008, img_017, img_023), gán quang gánh là xe (img_014). `bicycle` cũng kém (5 accepted / 3 deleted): bánh xích lô (img_010), người (img_017), xe máy (img_026) bị gọi `bicycle`.
- 4 trong deleted + added là **đổi lớp**, nên máy sót thật ≈ 16 − 4 = 12 vật.
- **So với gold gán tay:** precision 23/56 = 41.1% (KTC 95%: 29–54%), recall 23/37 = 62.2% (KTC 95%: 46–76%), mẫu số 37 đối tượng gold (4 ảnh). Accepted 91.6% nhưng recall trên gold chỉ 62% → **accepted cao không có nghĩa máy không sót**: người duyệt chỉ "added" được vật mình nhìn thấy. Lưu ý gold có ít `person` (img_001, img_020 không có box person) nên một phần FP của máy có thể là vật gold bỏ sót.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Bắt bẫy 8/12 = 66.7%** (KTC 95%: 39–86%; ngưỡng đạt 80%). Theo loại: thừa 3/3, sai lớp 2/3, bỏ sót 2/3, **box lỏng 1/3**. Lọt: img_009 (sai lớp), img_005 và img_027 (lỏng), img_035 (bỏ sót).
- Vì sao lọt: box lỏng không thấy được khi chỉ nhìn bao quát ảnh có sẵn box mà không phóng to từng cạnh (bước kiểm tra "lỏng" của checklist làm còn vội ở các ảnh mật độ dày img_005, img_027); ở img_009 khi duyệt đã phân vân box `bicycle` bên trái nhưng vẫn giữ nguyên do tâm lý chủ quan nghĩ "máy chắc có lý" — đúng kiểu thiên kiến tự động hóa; img_035 bỏ sót một người do mắt bị cuốn theo các box máy có sẵn thay vì quét ảnh trống trước.
- Accepted 91.6% mà bắt bẫy 67%: người duyệt chấp nhận phần lớn box và chỉ sửa lỗi "đập vào mắt" (box thừa, sai lớp rõ); lỗi tinh (box lệch vài chục px) và vật vắng mặt thì lọt — accepted cao phản ánh mức tin máy, không phản ánh chất lượng.

## 4. Máy có đáng dùng không?

- Thời gian: gán tay gold 10 phút / 4 ảnh = 2.5 phút/ảnh (40 đối tượng, `time_log.csv`). Duyệt batch 24 ảnh có pre-label mất 28 phút = 1.17 phút/ảnh (`time_log.csv`). Như vậy dùng pre-label giúp tăng tốc độ hơn gấp đôi (tiết kiệm ~53% thời gian/ảnh so với gán tay). Tuy nhiên cần lưu ý gold ít đối tượng hơn (10/ảnh so với ~20/ảnh của batch) nên trên thực tế nếu gán tay từ đầu toàn bộ ảnh batch thì pre-label còn tiết kiệm nhiều thời gian hơn nữa.
- Kết luận: **có, cho person / motorbike / car** (accepted cao, sửa ít), **không tin nhãn máy cho `three_wheeler`** (8/22 accepted, lỗi lớp không sửa được bằng ngưỡng) — lớp này cần người kiểm từng box.
- **Ba quy tắc giảm thiên kiến:**
  1. Quét ảnh **ẩn nhãn máy** 20 giây và đếm số vật chính trước khi bật nhãn; nếu số đếm > số box máy thì phải tìm đủ phần chênh (đo: bẫy `missing` 3/3).
  2. Mọi box `three_wheeler` và mọi box conf < 0.5 phải phóng to kiểm 4 cạnh (đo: bẫy `loose` 3/3).
  3. Box nào đã nghi thì không được giữ nguyên: sửa, xoá, hoặc tick `can_review` và ghi vào calibration (đo: mỗi box `can_review` có một dòng trong `calibration_decisions.csv`).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk) | SAM 2.1 b | 3 (2 dương, 1 âm) | Gồm người lái và khách ngồi trong xe | Mask phủ 45% ảnh, không tách được người khỏi xe khi người ngồi trong khung | `tuktuk_img013.jpg` |
| img_008 (ba gác chở chuối) | SAM 2.1 b | box + 1 dương | Gồm chuối trong thùng, **không** gồm chuối chất cao bên trên | Bám khung và bánh tốt | `bagac_img008.jpg` |
| img_010 (xích lô, nhìn sau) | SAM 2.1 b | box + 1 dương + 1 âm (lên người đạp) | Vẫn dính một phần áo người đạp | Một click âm chưa đủ; khung có lỗ hở lớn nhưng mask vẫn liền | `xichlo_img010.jpg` |

- Guideline 4.2: quy tắc 4.2.4 (hai vật chạm nhau không chồng) khó đạt nhất — người ngồi trong xích lô / tuk-tuk bị SAM gộp vào xe. 4.2.2 (lỗ thủng ≥ 5% bỏ ra) đúng với xích lô nhưng SAM không tự làm. Thiếu: quy tắc mask cho hàng chất trên xe (câu hỏi mở 3; chốt ở CAL-04 cho box, mask theo cùng ngưỡng 20%).
- Interactor SAM trong CVAT: chưa thử (cần cài serverless / Nuclio).

## 6. Calibration (Part 6)

- Số quyết định: 5 (`calibration_decisions.csv`): 2 `loi_may`, 2 `cau_hoi_mo`, 1 `bay_lot`. Quan trọng nhất: **CAL-01** (tuk-tuk luôn `three_wheeler`, không tin nhãn YOLO26) vì là lỗi lặp lớn nhất và không sửa được bằng ngưỡng.
- Quy tắc mới trong `guideline_v1.1_calibrated.md`: chốt câu hỏi mở 2 (cụm xe đỗ gần camera: hàng đầu gán từng xe, phía sau một `ignore_region`) và 3 (hàng gắn xe nhô ≤ 20% tính vào box); thêm: vật không có bánh không phải `three_wheeler`; bộ phận xích lô không gán `bicycle`.
- Nguyên nhân gốc: CAL-01, CAL-02 do **máy** (COCO không có xe ba bánh; prompt chữ của YOLOE bắt cả vật treo); CAL-03, CAL-04 do **guideline mơ hồ**; CAL-05 do **người duyệt** bỏ qua box máy cắt đôi.

## 7. QC và quyết định (Part 6)

- **QC vòng 1 (lịch sử):** 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42): img_003, 006, 014, 031, 035; 100 đối tượng. Người kiểm: tự kiểm tra lại (self-check).
- Lỗi vòng 1 theo **đối tượng**: 4/100 = 4.0% · theo **ảnh**: 4/5 = 80.0%. Phát hiện: sai vị trí/kích thước 2 (img_003, img_031), thừa 1 (img_006 `car`), bỏ sót 1 (img_035 người).
- **Rework:** Rà soát và chỉnh sửa lại trên CVAT toàn bộ 24/24 ảnh batch theo guideline v1.1; cập nhật 7 ảnh trong `labels/qc_fixed.json`; loại `car` giả ở img_006, thêm người ở img_035, sửa box xe đạp/khay ở img_031 theo quy tắc hàng gắn ≤20%, và vẽ lại img_003 để bao trọn xe ba gác cùng mui gắn liền theo guideline (không cắt mui). Các box dưới ngưỡng ở img_025 và img_033 được chuyển thành `ignore_region`; hai box xe nhỏ chồng sát ở img_027 được gộp thành một `ignore_region` thay vì giữ class không chắc. Build lại release candidate: 24 ảnh, 475 đối tượng (`release_summary.csv`); không còn cảnh báo box riêng lẻ thấp hơn ngưỡng chiều cao 20 px.
- **QC vòng 2:** mẫu mới seed 43 gồm 10/24 ảnh (0 ca khó + 10 ngẫu nhiên): img_003, 004, 006, 008, 014, 017, 021, 024, 025, 031; 186 đối tượng (`qc_sample.csv`). Người kiểm: học viên trực tiếp kiểm tra độc lập từng ảnh và đối tượng theo guideline v1.1 (`qc_review.csv`).
- Kết quả vòng 2: lỗi theo **đối tượng** 0/186 = 0.0%; ảnh có lỗi 0/10 = 0.0% (`qc_report_auto.md`). Tuy nhiên đây chưa phải bằng chứng tuyệt đối rằng tỉ lệ lỗi thật của toàn tập đạt ngưỡng ≤2%: cận trên khoảng tin cậy 95% Wilson vẫn là **28%** do kích thước mẫu 10 ảnh còn nhỏ.
- Script kiểm được: số box, lớp lạ, box ngoài ảnh, box < 20 px, so với bẫy và gold. Không kiểm được: box có ôm sát không, vật cả máy lẫn người cùng bỏ sót, người duyệt có làm theo checklist thật không.

**Quyết định hiện tại: REWORK**, ngày 2026-11-17. Không chọn Hold vì guideline v1.1 đã chốt rõ ràng các ca mơ hồ gặp trong lô (xe ba bánh, hàng nhô, cụm xe đỗ), dữ liệu không có ảnh hỏng. Quyết định REWORK thay vì Release ngay vì tỉ lệ bắt bẫy đạt 8/12 = 66.7% < 80% (`review_stats.json`). Dù QC vòng 2 đạt 0/10 ảnh lỗi nhưng do giới hạn mẫu nhỏ (cận trên Wilson 28%), cần tiếp tục rà soát cẩn thận lại quy trình theo checklist 4 bước, đặc biệt với các ảnh mật độ dày (img_005, 018, 021, 024, 028, 029) và các box `three_wheeler` trước khi chính thức Release toàn bộ dữ liệu.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Không dùng AI cho việc gán nhãn hay ra quyết định; chỉ dùng ChatGPT để tra cứu nhanh cách thiết lập biến môi trường sửa lỗi font tiếng Việt trên PowerShell Windows (`PYTHONUTF8=1`) và kiểm tra chính tả. Cách kiểm tra: tự kiểm chứng trực tiếp bằng cách chạy lệnh trên terminal và đối chiếu theo tài liệu GUIDE.md. Toàn bộ các bước gán nhãn, review trên CVAT, chạy model và phân tích số liệu đều do học viên tự thực hiện.
- Gold có bị nhiễm không? Gold (`labels/gold_manual.json`) do tôi tự gán tay hoàn toàn trong CVAT (Task `gold-manual`, 10 phút, trước khi chạy bất kỳ model pre-label nào lên các ảnh này), đảm bảo nhãn gold hoàn toàn độc lập và không bị ảnh hưởng bởi gợi ý của máy.
- Máy / GPU / CVAT: Windows 11, RTX 3050 Laptop 4 GB (driver CUDA 11.2 → torch 2.7.1+cu118), CVAT 2.81 Docker. Vướng: hai bộ volume CVAT (bản cũ và 2.81) nên lúc đầu không thấy project cũ, sau khi cấu hình lại volume đã truy cập bình thường.
- Dùng cache `data/cache/` thay vì tự chạy? Không: tự chạy YOLO26m và YOLOE-26s trên GPU cá nhân.
- Lỗi / bug script đã gặp (nếu có): `scripts/check_submission.py` ban đầu chỉ nhận MSHV toàn chữ số (`\d{5,}`), MSHV `2A202603236` có chữ cái nên bị báo sai "README thiếu tiêu đề" → đã sửa regex thành `[0-9A-Za-z]{5,}`. Trên Windows phải chạy với `PYTHONUTF8=1` (để tránh lỗi cp1252 khi subprocess đọc output tiếng Việt UTF-8).
- Một điều làm khác lần sau: Nghiêm túc tuân thủ bước quét ảnh ẩn nhãn máy và phóng to kiểm tra đủ 4 cạnh từng box (đặc biệt các ảnh đông đúc) để nâng cao tỉ lệ bắt bẫy vượt ngưỡng 80%.
