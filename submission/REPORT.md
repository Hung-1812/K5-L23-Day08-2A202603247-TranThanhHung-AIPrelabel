# Báo cáo Day 08 — <<2A202603247-Trần Thành Hùng>>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | mặc định `conf=0.30` | Quét gold có 16 người; recall tối đa 88%, chưa đạt mục tiêu 90%, nên giữ ngưỡng mặc định (`threshold_sweep.md`). |
| bicycle | YOLO26m | mặc định `conf=0.30` | Chỉ có 3 xe đạp trong gold, mẫu quá ít để chọn ngưỡng riêng (`threshold_sweep.md`). |
| motorbike | YOLO26m | mặc định `conf=0.30` | Có 8 xe máy trong gold; recall tối đa 88%, chưa đạt mục tiêu 90% (`threshold_sweep.md`). |
| car | YOLO26m | mặc định `conf=0.30` | Có 6 ô tô trong gold; recall tối đa 83%, chưa đạt mục tiêu 90% (`threshold_sweep.md`). |
| bus / truck | YOLO26m | mặc định `conf=0.30` | Không có đối tượng bus hoặc truck trong gold nên không chọn được ngưỡng riêng (`threshold_sweep.md`). |
| three_wheeler | YOLOE-26s; prompt theo taxonomy: `tuk-tuk`, `cyclo`, `cargo tricycle` | `0.15` | Quét trên 5 đối tượng gold cho recall 5/5 = 100%; precision báo cáo 56% (`threshold_sweep.md`; bảng không ghi TP/FP để xác định mẫu số precision). |



- Lệnh `merge_prelabels.py` đã dùng: python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.35,motorbike=0.5,car=0.35" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json

- SAM dùng: SAM 2.1 (`sam2.1_b.pt`) cho ba ảnh mục tiêu; có so sánh SAM 1 (MobileSAM) trên `img_013`. Log `r04` ghi 16 mask/2 ảnh và 7.23 giây/ảnh trên CPU. Trong ảnh so sánh `img_013`, mask SAM 1 quan sát được liền mạch hơn quanh người lái; RAM không được ghi lại.
- Lỗi không sửa được bằng đổi ngưỡng: YOLO26 không có lớp `three_wheeler` trong cấu hình đã chạy (`model_log.csv`, `r01`–`r02`); hạ/tăng confidence không thể thêm lớp này. Dùng YOLOE để đề xuất lớp `three_wheeler`, rồi kiểm tra theo taxonomy.
- Ví dụ box điểm cao mà sai / điểm thấp mà đúng: **không có**.
- Ngưỡng được chọn trên chính 4 ảnh gold có 38 đối tượng rồi cũng được đánh giá trên gold đó; vì vậy precision/recall in-sample có thể lạc quan và mẫu nhỏ, không đại diện chắc chắn cho ảnh mới (`review_stats.md`).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---:|
| accepted | 343 | 343/433 = 79.2% | 433 box máy |
| edited | 38 | 38/433 = 8.8% | 433 box máy |
| deleted | 52 | 52/433 = 12.0% | 433 box máy |
| added | 35 | 35/416 = 8.4% | 416 đối tượng sau duyệt |


- Theo tỉ lệ box máy bị xoá, truck có tỉ lệ cao nhất: 6/7 = 85.7%; mẫu truck nhỏ nên kết luận này yếu. Với `three_wheeler`, có 10/24 box bị xoá và 6/24 bị sửa. `review_stats.md` chỉ có thống kê theo lớp, không cho biết ảnh cụ thể của từng box bị xoá.
- So với gold: precision của máy 30/66 = 45.5% (KTC 95%: 34–57%); recall 30/38 = 78.9% (KTC 95%: 64–89%). Mẫu số recall là 38 đối tượng gold (`review_stats.md`).


## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt được 12/12 bẫy = 100%; không có loại bẫy nào bị lọt theo `review_stats.md`: loose 3/3, extra 3/3, missing 3/3, wrong_class 3/3.
- Số liệu bẫy cho thấy đã sửa các bẫy được cài, nhưng không chứng minh rằng checklist đã được theo đúng từng bước trên mọi ảnh. Accepted 343/433 = 79.2% và bắt bẫy 12/12 là hai phép đo khác nhau; QC vẫn phát hiện lỗi kích thước/vị trí ở 1/5 ảnh mẫu (`img_031`).

## 4. Máy có đáng dùng không?

- Thời gian gán tay là 30/4 = 7.5 phút/ảnh (`time_log.csv`).
- Kết luận: dùng máy như công cụ tạo pre-label để tiết kiệm bước vẽ ban đầu, nhưng không dùng để tự động phát hành. Cơ sở: accepted 343/433 = 79.2%, precision 30/66 = 45.5%, recall 30/38 = 78.9%, và QC có lỗi ở 1/5 ảnh. Riêng lớp `three_wheeler` cần kiểm tra kỹ vì YOLO26 không có lớp này; YOLOE đạt recall 5/5 nhưng precision báo cáo 56% trên gold.
- Ba quy tắc giảm thiên kiến cho lượt sau:
  1. Trước khi hiện pre-label, quét ảnh và tự đếm đối tượng theo lớp ở 100% ảnh review; ghi số đếm trước khi so với máy.
  2. Kiểm tra mọi box, kể cả box confidence cao; dùng đủ bốn bước checklist trên 100% ảnh.
  3. Sau review, làm QC ngẫu nhiên và ghi lỗi theo cả ảnh lẫn đối tượng; nếu có lỗi thì sửa, chạy lại QC trước khi quyết định phát hành.


## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---:|---|---|---|
| `img_004` | SAM 2.1 (`sam2.1_b.pt`) | 3 | Không thấy rõ người lái hoặc hàng hoá trong vùng mask | Mask bao được thân và mái xe; cần rà lại mép bánh, gầm xe và vùng bóng đổ để tránh lấy thừa | `tuktuk_img004_sam2.jpg` |
| `img_013` | SAM 2.1 (`sam2.1_b.pt`) | 3 | Có lẫn một phần vùng người lái/nội thất; không thấy hàng hoá rõ | Mask xe khá đầy đủ nhưng còn vùng nhiễu, phân mảnh quanh người lái; một click âm chưa tách sạch người và xe | `tuktuk_img013_sam2.jpg` |
| `img_013` | SAM 1 (MobileSAM) | 3 | Người lái phần lớn nằm ngoài mask; không thấy hàng hoá rõ | Mask xe liền mạch hơn và ít lẫn người lái hơn SAM 2.1 trong ảnh này; vẫn cần kiểm tra kỹ vùng tiếp giáp | `tuktuk_img013_sam1.jpg` |
| `img_017` | SAM 2.1 (`sam2.1_b.pt`) | 3 | Không thấy rõ người lái hoặc hàng hoá trong vùng mask | Tách được xe mục tiêu; cần rà mép bánh/gầm và vùng sát xe phía sau để mask không ăn sang nền hay vật khác | `tuktuk_img017_sam2.jpg` |


- **So sánh SAM trên cùng ảnh `img_013`:** với cùng 3 click, SAM 1 (MobileSAM) cho mask tuk-tuk liền mạch hơn và ít lẫn vùng người lái hơn SAM 2.1 trong kết quả quan sát được. SAM 2.1 vẫn lấy được phần lớn xe nhưng còn nhiễu/phân mảnh quanh người lái và nội thất. Đây là nhận xét trên ảnh thử này, không khẳng định SAM 1 tốt hơn trong mọi trường hợp.
- **Đối chiếu guideline mục 4.2:** quy tắc bám biên phần nhìn thấy giúp xác định phạm vi xe; kính xe thuộc xe, còn bóng đổ không thuộc mask. Quy tắc xử lý lỗ giúp tránh chạy theo các chi tiết nhỏ của khung xe. Cần rà thủ công các vùng hở, bánh/gầm và chỗ xe tiếp xúc với người hoặc nền; quy tắc người ngồi trên xe phải là đối tượng riêng cũng cho thấy một click âm chưa đủ để tách sạch người lái khỏi xe.
- **Interactor SAM trong CVAT:** 

## 6. Calibration (Part 6)

- Có 3 quyết định (`CAL-01` đến `CAL-03`) trong `calibration_decisions.csv`.
- Quan trọng nhất: `CAL-01` — YOLO26 không có lớp `three_wheeler`, nên không thể sửa bằng đổi confidence; cần model có lớp phù hợp và phải kiểm tra theo taxonomy.
- `guideline_v1.1_calibrated.md` đã chốt câu hỏi mở 3 (hàng gắn cố định/hàng chở rời) và 5 (kính xe thuộc mask). Năm câu hỏi mở còn lại là câu 1, 2, 4, 6 và 7 trong guideline v1.0.
- Nguyên nhân gốc gồm giới hạn lớp của model và quy tắc guideline chưa chốt đủ rõ.

## 7. QC và quyết định (Part 6)


- Mẫu QC gồm 5 ảnh: 0 ca `can_review` + 5 ảnh ngẫu nhiên, seed 42; tổng 89 đối tượng (`qc_sample.csv`). Người kiểm là người duyệt nhãn, đây là một hạn chế.
- Lỗi theo đối tượng: 1/89 = 1.1%; lỗi theo ảnh: 1/5 = 20.0% (ngưỡng Release ≤ 2%).
- Lỗi ghi nhận: `sai_vi_tri_kich_thuoc`, 1 ảnh là `img_031` (box bicycle #368).
- Cận trên Wilson 95% theo báo cáo QC hiện tại là 62% với 1/5 ảnh lỗi. Mẫu nhỏ không chứng minh được tỉ lệ lỗi thật ≤ 2%. Ngay cả 0/7 ảnh lỗi, cận trên Wilson vẫn khoảng 35%; với 4/7 ảnh lỗi, cận trên khoảng 84%.
- Script kiểm được số lượng, lớp được ghi, hình học nằm trong ảnh và thống kê; không tự xác định được box có ôm sát vật thật, vật bị cả máy và người cùng bỏ sót, hay người review có làm đúng checklist không.
- **Quyết định: HOLD**, ngày 2026-11-19. Guideline còn 5 câu chưa chốt nên Hold được ưu tiên theo Hold → Rework → Release. Báo cáo QC tự động hiện gợi ý Rework vì 1/5 ảnh lỗi = 20% > 2%; sau khi chốt các câu hỏi guideline, cần sửa box #368 ở `img_031`, lấy mẫu và chạy QC lại trước khi cân nhắc Release.
- Chưa có ảnh hỏng nào được ghi nhận trong mẫu QC.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** có dùng trợ lý AI để đọc log, xem ảnh QC/mask và soạn nháp báo cáo; các số liệu trong báo cáo được đối chiếu với các file QC/model log. 
- Gold có bị nhiễm hay không: **Không**
- Máy: Windows; các lượt inference trong log chạy CPU. GPU cụ thể không được ghi. Phiên bản CVAT: ****.
- Dùng cache `data/cache/`: **có**.
- Lỗi đã gặp: YOLO26 không có lớp `three_wheeler`; xử lý bằng lượt YOLOE riêng, không đổi confidence.
- Điều làm khác lần sau: chốt các câu hỏi guideline còn mở trước khi phát hành.
