# Label contract — Phố châu Á — BỔ SUNG v1.1 — <<2A202603247 - Trần Thành Hùng>>

## 1. Câu hỏi mở đã chốt

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 3 | Hàng nhô ra: box xe có gồm hàng không? | Giỏ/khay gắn cố định thuộc xe; hàng chở rời nhô quá khoảng 20% chiều dài không tính vào box xe. | `img_031` | `CAL-03` |
| 5 | Kính chắn gió/cửa sổ có thuộc mask xe không? | Kính nhìn thấy thuộc mask xe; người nhìn thấy qua kính vẫn là `person` riêng. | `img_013` | `CAL-02` |

## 2. Ca khó mới

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định | Quy tắc mới |
|---|---|---|---|---|
| `img_013` | Tuk-tuk thuộc lớp `three_wheeler` | YOLO26 không có lớp này; đổi confidence không giải quyết được | Dùng YOLOE để đề xuất lớp phù hợp, sau đó kiểm tra theo taxonomy | Model chỉ là gợi ý; gán lớp theo vật thể |
| `img_031` | Box bicycle thiếu phần khay hàng nhìn thấy | QC phát hiện box #368 chưa bao hết khay gắn cố định | Sửa box bao phần xe và khay gắn cố định; không đưa hàng rời nhô quá khoảng 20% vào box xe | Phân biệt bộ phận gắn cố định với hàng chở rời |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở 3 và 5; thêm quy tắc kiểm tra lớp `three_wheeler` và khay hàng gắn cố định | `CAL-01`–`CAL-03` |
