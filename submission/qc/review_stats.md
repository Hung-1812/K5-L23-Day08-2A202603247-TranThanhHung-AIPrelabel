# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 433; đối tượng sau duyệt: 416; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 343 | 343/433 = 79.2% | 433 box máy |
| edited (sửa) | 38 | 38/433 = 8.8% | 433 box máy |
| deleted (xoá) | 52 | 52/433 = 12.0% | 433 box máy |
| added (vẽ thêm) | 35 | 35/416 = 8.4% | 416 đối tượng sau duyệt |

Trong số deleted + added, **7 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 28.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 216 | 17 | 18 | 8 |
| bicycle | 5 | 1 | 3 | 3 |
| motorbike | 79 | 8 | 2 | 11 |
| car | 32 | 6 | 12 | 5 |
| bus | 2 | 0 | 1 | 2 |
| truck | 1 | 0 | 6 | 2 |
| three_wheeler | 8 | 6 | 10 | 4 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 38 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 30 | 36 | 8 | 45.5% (KTC 95%: 34–57%) | 78.9% (KTC 95%: 64–89%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 12/12 = 100.0% (KTC 95%: 76–100%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 3/3
  - extra: 3/3
  - missing: 3/3
  - wrong_class: 3/3

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
