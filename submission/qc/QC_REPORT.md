# QC_REPORT — Day 07 (HOLD, bản tích hợp máy, CHƯA được người kiểm xác nhận)

## 1. Mục đích, dữ liệu và phiên bản
- Mục đích và giới hạn sử dụng: Thực hành gán nhãn 2D trên 40 ảnh; bản v3 là nhãn MÁY ĐỀ XUẤT, không được dùng làm ground truth đã QC.
- Nguồn và phép sử dụng dữ liệu: Xem data/ATTRIBUTION.csv; không suy luận quyền ngoài thông tin đã ghi.
- Manifest và checksum nhãn cuối: Có labels_calibrated.json (bản MÁY CHƯA XÁC NHẬN), labels_calibrated.csv và batch_manifest.json; SHA256 COCO=b1a9fd73e7bc3a84b9b91aba5448f943837415a6fcb91fe45d15df2f2c5312b1. Manifest có status=HOLD_MACHINE_ONLY.
- Guideline và taxonomy khi gán / khi bàn giao: V1.0 cho gán độc lập, guideline v1.1 cho bản đề xuất; taxonomy v1.0, thay đổi semantic cần người kiểm cả lô.

## 2. Phân công và IAA trước calibration
- Lượt hoàn thành / được giao: 176/176 lượt trên 40 ảnh, theo assignment.csv và iaa_summary.md.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 1.137 cặp ghép; IoU trung bình 0,8383 (ngưỡng 0,85), cùng lớp 96,48% (ngưỡng 95%); các cặp chưa ghép không được tính vào mẫu số.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: Có cảnh báo ca bỏ sót/thừa và 412 ứng viên bất đồng ignore, xem iaa_summary.md và calibration_candidates.csv.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Chưa có người xác minh ảnh tham chiếu; ưu tiên 5 ảnh đối chứng giả, tượng, đồ chơi theo R-01.

## 3. Kết quả tích hợp máy
- Số ca máy đề xuất: 3678/3678; người xác nhận: 0/3678.
- Mức tin cậy của máy: {'inferred': 3073, 'low': 406, 'rule_specific': 199}. Đây KHÔNG phải tỉ lệ chính xác dự đoán.
- Bộ COCO máy đề xuất: 678 -> 508 box; số thao tác máy: 532.
- Loại thay đổi tự động: MEDIAN_BBOX=298, TRUNCATED_EDGE_DEFAULT=48, OCCLUDED_MAJORITY=7, TRUNCATED_INTERIOR=7, DROP_BOX_UNDER_MIN_HEIGHT=5, DROP_REFERENCE_IMAGE=44, CLASS_MAJORITY=2, NEAR_IDENTICAL_DEDUP=121.
- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Các quyết định CHƯA ĐƯỢC NGƯỜI CHỐT nằm ở auto_machine_decisions.csv; calibration_decisions.csv gốc vẫn chưa được điền đủ. Không mạo nhận số ca đã hiệu chuẩn.
- Tác động v1.1 lên nhãn và ảnh: 40/40 ảnh thuộc phạm vi semantic; phải kiểm lại việc gộp box gần nhau, người thật bỏ sót, ignore, và occlusion.
- Tham chiếu bằng chứng khi Rework: auto_machine_changes.csv, auto_machine_report.json và manifest SHA256; vòng QC sau cần dựa trên nhãn được người xác nhận.

## 4. Duyệt QC
- Vòng QC: 2
- Người review và ngày: Chưa có reviewer thực tế; việc tạo phiếu trống không tính là kiểm ảnh.
- Đã duyệt / cả lô: 0/40 ảnh đã được người kiểm thực tế; chưa tạo được phiếu QC.
- Ảnh lỗi / ảnh đã duyệt: N/A (0 ảnh được người kiểm thực tế).
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: N/A (0 annotation được người kiểm thực tế).
- Vật bỏ sót, ảnh và vùng: Chưa được kiểm trực quan; không coi bo_sot_thua trong IAA là số lỗi thực tế.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Ưu tiên low-confidence và box bị gộp; xem auto_machine_decisions.csv + auto_machine_changes.csv.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Có thể kiểm kích thước/biên box; chỉ nhìn ảnh thật mới xác định đúng đối tượng và che khuất.

## 5. Quyết định bàn giao
- Quyết định: Hold
- Lý do, người nhận và phạm vi sử dụng: HOLD vì 3678/3678 quyết định chỉ do máy đề xuất, chưa có xác minh người thật và phiếu QC; mã 291 bàn giao bản NHÁP để nhóm kiểm lại, không công bố Release.
- Nếu còn việc: Nhóm duyệt quyết định calibration, QC 40 ảnh, bổ sung individual_notes từng thành viên; nộp theo SUBMISSION.md kênh riêng tư.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Mở submission/qc/auto_machine_decisions.csv (case_id, rule_id, confidence), đối chiếu submission/qc/auto_machine_changes.csv và ảnh data/images; ví dụ 5 ảnh img_017/020/027/033/038 phải có 0 nhãn theo R-01/R-13.

> LƯU Ý: `qc_report.py` có thể xuất HOLD theo bằng chứng thiếu; việc có manifest/hash hợp lệ KHÔNG chứng minh box đúng thực tế. Không dùng report này để tuyên bố Release.
