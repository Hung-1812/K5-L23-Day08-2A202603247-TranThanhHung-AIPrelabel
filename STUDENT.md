# Thông tin học viên

- Họ và tên: <<Trần Thành Hùng>>
- MSHV: <<2A202603247>>
- Lớp (L2 / L3 / khác): <<L2>>
- Tài khoản GitHub: <<>>
- Repo bài nộp (URL): <<>>
- Commit chốt bài (hash): <<>>
- Máy: <<hệ điều hành, GPU (CUDA / MPS / không có)>>
- CVAT (Docker): <<phiên bản, ví dụ v2.81.0>>

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted 343/433 = 79.2%; edited 38/433 = 8.8%; deleted 52/433 = 12.0% (mẫu số: 433 box máy); added 35/416 = 8.4% (mẫu số: 416 đối tượng sau duyệt).
- So với gold: precision 30/66 = 45.5% (KTC 95%: 34–57%); recall 30/38 = 78.9% (KTC 95%: 64–89%).
- Bắt bẫy: 12/12 = 100% (KTC 95%: 76–100%).
- Thời gian: gán tay 30/4 = 7.5 phút/ảnh; duyệt có pre-label: chưa ghi trong `time_log.csv`.
- Lỗi QC: 20.0% theo ảnh (1/5); 1.1% theo đối tượng (1/89).
- **Quyết định: Hold** — còn 5 câu guideline chưa chốt; QC cũng ghi nhận 1/5 ảnh lỗi, vượt ngưỡng 2%.
