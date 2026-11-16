# Báo cáo QC — Day 07 (HOLD, chưa hoàn tất QC)

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Thực hành xây dựng guideline, gán và so sánh nhãn bounding box trên 40 ảnh đường phố. Chưa đủ bằng chứng phát hành nhãn cuối.
- Nguồn và phép sử dụng dữ liệu: `data/ATTRIBUTION.csv` lưu tên ảnh, tác giả, giấy phép và nguồn.
- Manifest và checksum nhãn cuối: **Chưa có** `labels_calibrated.json`, `labels_calibrated.csv`, `batch_manifest.json` hợp lệ. `consensus_draft.json` là bản nháp chưa được kiểm lại; không dùng checksum suy đoán.
- Guideline và taxonomy khi gán / khi bàn giao: Gán độc lập và IAA theo v1.0. Có tài liệu guideline v1.1 với thay đổi `semantic`, chưa có quyết định và kiểm lại cả lô đầy đủ.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: **176/176**, 40 ảnh; mã 066 được giao 16 ảnh, mã 263, 273, 277 và 291 mỗi mã 40 ảnh.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: **1.137** cặp; IoU TB **0,8383** (< 0,85); đồng thuận lớp **96,48%** (>= 95%). Mẫu số chỉ gồm các cặp box ghép được.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: **999/1.260** box chưa ghép ở hai phía; **5** box bỏ qua do ignore; **412** ứng viên bất đồng ignore.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Chưa có biên bản đối chiếu tham chiếu coach. Ví dụ cần xem `img_002.jpg` về xe chở xe (R-06) và các ảnh vật mô phỏng (R-01). Người gán giống nhau chưa bảo đảm nhãn đúng.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: **0/3.678** ca có quyết định được điền tại `calibration_decisions.csv`; số 3.678 là số ứng viên do script xuất, không phải số quyết định hoàn tất.
- Tác động v1.1 lên nhãn và ảnh: **40/40** ảnh xuất hiện bất đồng, guideline thay đổi `semantic` nên cần rà lại cả lô. Đề xuất rà thuộc tính occluded/truncated, phân lớp xe, ignore và vật mô phỏng; chưa xác nhận số box đã sửa.
- Tham chiếu bằng chứng khi Rework: Chưa có vòng QC hợp lệ để kết luận Rework hay Release; hiện HOLD do thiếu bằng chứng.

## 4. Kết quả duyệt

- Vòng QC: Chưa có vòng duyệt hoàn tất (dự kiến vòng 1).
- Người review và ngày: Chưa có reviewer và ngày duyệt được xác nhận.
- Đã duyệt / cả lô: **0/40** ảnh có phiếu QC hợp lệ.
- Ảnh lỗi / ảnh đã duyệt: **N/A (0 ảnh đã duyệt)**, không phải 0%.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: **N/A (0 annotation được QC xác nhận)**, không phải 0%.
- Vật bỏ sót, ảnh và vùng: Chưa có phiếu QC xác nhận; `bo_sot_thua` là cảnh báo so sánh giữa người gán.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: **3.678** ứng viên gồm 2.022 bỏ sót/thừa, 663 sai thuộc tính, 442 IoU thấp, 412 bất đồng ignore, 99 cần review, 40 sai lớp. Ví dụ **C296** (`img_004.jpg`) là khác lớp `other_vehicle`/`car` giữa 273|291 dù IoU **0,96**; chưa có quyết định cuối. Xem `IAA_DIAGNOSTICS.md`.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Script có thể xác thực hình học box, class và thuộc tính theo taxonomy; người kiểm bằng mắt mới xác định được một vật thực sự có bị bỏ sót hay gán thừa. Chưa khẳng định đã kiểm bằng mắt.

## 5. Quyết định bàn giao

- Quyết định: **Hold**
- Lý do, người nhận và phạm vi sử dụng: Chưa hoàn tất calibration, nhãn cuối và QC nên không đủ căn cứ Release. Mã **291** đang tổng hợp hồ sơ; chưa phân công reviewer độc lập. Dùng hồ sơ để đánh giá tiến độ lab, không sử dụng như dữ liệu chuẩn đã QC.
- Nếu còn việc: Chốt người review, xác nhận quyết định theo ảnh và rule, điền calibration, tạo nhãn cuối/manifest, duyệt 40 ảnh nếu phát hành, chạy `qc_report.py` và `check_submission.py`, bổ sung ghi chú cá nhân được người liên quan xác nhận.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: **C001** trong `calibration_candidates.csv` thuộc `img_001.jpg`, cặp 263|273, lớp person có box `[109,113,89,130]` ở phía 263 nhưng phía 273 không có box tương ứng. C001 chưa được chốt. **C296** là bất đồng lớp `other_vehicle`/`car` với IoU 0,96, đối chiếu R-03 trước khi quyết định.

> Báo cáo này là hồ sơ HOLD do thiếu đầu vào theo `SUBMISSION.md`. Không tự nhận `check_submission.py` đã PASS hoặc đã hoàn tất review QC.
