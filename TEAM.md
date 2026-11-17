# Thông tin nhóm — bàn giao HOLD

- Tên nhóm: N (ký hiệu theo repository; tên chính thức chưa xác nhận).
- Mã thành viên (ba chữ số): 066, 263, 273, 277, 291.
- Người điều phối / đầu mối tổng hợp hồ sơ hiện tại: 291. Không suy diễn vai trò điều phối ở các pha trước.
- Người review bản cuối: Chưa có reviewer được xác nhận.
- Commit chốt bài: Điền sau khi hoàn tất nộp qua kênh lớp.

## Kết quả tóm tắt

- Lượt gán hoàn tất / được giao: **176/176**, 40 ảnh; theo `submission/qc/assignment.csv`.
- IAA trước calibration (kèm mẫu số): **1.137 cặp box ghép**, mean IoU **0,8383**, cùng lớp **96,48%**; IoU chưa đạt ngưỡng 0,85, đồng thuận lớp đạt ngưỡng 95%.
- Số ảnh phải calibration: **40/40**; **3.678** ứng viên, **0/3.678** quyết định có xác nhận.
- Vòng QC và số ảnh đã duyệt / cả lô: Chưa tạo được phiếu QC; **0/40** ảnh được người kiểm xác nhận.
- Tỉ lệ lỗi: N/A theo ảnh và đối tượng vì chưa duyệt; không dùng 0%.
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: **Hold** — đã có nhãn máy đề xuất 508 box cho 3678 ca, CHƯA xác nhận người thật hay hoàn tất vòng QC.

## Bằng chứng đóng góp theo mã

| Mã | Guideline cá nhân | COCO | Ảnh theo phân công | Số annotation |
|---|---|---|---:|---:|
| 066 | `submission/guidelines/individual/guideline_066.md` | `submission/labels/066.json` | 16 | 75 |
| 263 | `submission/guidelines/individual/guideline_263.md` | `submission/labels/263.json` | 40 | 326 |
| 273 | `submission/guidelines/individual/guideline_273.md` | `submission/labels/273.json` | 40 | 343 |
| 277 | `submission/guidelines/individual/guideline_277.md` | `submission/labels/277.json` | 40 | 318 |
| 291 | `submission/guidelines/individual/guideline_291.md` | `submission/labels/291.json` | 40 | 311 |

Các số liệu trên chứng minh sự tồn tại của file nhãn/guideline, không xác nhận từng người đã làm calibration hay review QC.

## Bàn giao

- Đã có: ảnh và ghi công nguồn, guideline các phiên bản, taxonomy, nhãn độc lập, assignment, IAA, ứng viên calibration, consensus nháp.
- Có bản labels_calibrated.json + manifest từ máy với trạng thái HOLD_MACHINE_ONLY; còn thiếu quyết định người chốt, QC thực tế, và ghi chú cá nhân tự xác nhận.
- Đầu mối hiện tại: 291; chưa phân công người review tiếp.
- Hồ sơ **HOLD** được nộp như bằng chứng đã làm theo `SUBMISSION.md`, không phải dữ liệu Release; `check_submission.py` chưa thể PASS.
