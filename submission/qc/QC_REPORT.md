# Báo cáo QC — Day 07

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Thực hành đặc tả và QC bounding box 2D trên 40 ảnh đường phố tuyển chọn. Chưa đánh giá độ đại diện cho dữ liệu vận hành.
- Nguồn và phép sử dụng dữ liệu: Xem `data/ATTRIBUTION.csv` để tra tác giả, nguồn và giấy phép từng ảnh.
- Manifest và checksum nhãn cuối: Bản đưa duyệt gồm 40 ảnh và 629 annotation (628 đối tượng, 1 vùng ignore). Manifest: batch_manifest.json. SHA-256: 8de5258bf18839a3f1f43b98c062d92bfdc891bfda58bc42a9c4e898145bef61.
- Guideline và taxonomy khi gán / khi bàn giao: IAA dùng guideline và taxonomy v1.0. Repo có guideline v1.1, còn chờ hoàn tất calibration và xác minh bản bàn giao.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 176/176 lượt gán độc lập, 40 ảnh, 5 người gán.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 1.137 cặp box ghép, IoU trung bình 0,838, đồng thuận lớp 96,5% trên 1.137 cặp. IoU thấp hơn ngưỡng 0,85.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 999/1.260 box chưa ghép theo hai phía, 5 box được bỏ qua do ignore, 412 ca bất đồng vùng ignore.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Cần kiểm đủ xe con trên hai tầng của xe chở ô tô ở `img_002.jpg` bằng ảnh gốc và quy tắc xe chở xe.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Đã điền 3.678/3.678 đề xuất trong calibration_decisions.csv theo guideline v1.1. Các ID khung trong bảng là ID của consensus_draft.json. Phiếu QC dùng ID trên ảnh overlay. Người duyệt xác nhận từng ca cần đối chiếu.
- Tác động v1.1 lên nhãn và ảnh: 40/40 ảnh có ca cần calibration. Dùng bản đồng thuận làm cơ sở. Bỏ 44 khung trên 5 ảnh đối chứng và 5 khung dưới 20 px. Chuẩn hóa truncated ở mép ảnh theo v1.1. Các lựa chọn theo IoU và thuộc tính còn cần đối chiếu trực tiếp trên ảnh.
- Tham chiếu bằng chứng khi Rework: Chưa có vòng duyệt đủ bằng chứng để kết luận Rework.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: Chưa có mã reviewer độc lập hoặc ngày duyệt.
- Đã duyệt / cả lô: 0/40 ảnh.
- Ảnh lỗi / ảnh đã duyệt: Chưa đo (0 ảnh đã duyệt).
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: Chưa đo (0 annotation đã duyệt).
- Vật bỏ sót, ảnh và vùng: Chờ phiếu duyệt. Các bất đồng IAA chỉ là ca cần đối chiếu.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: 3.678 ca IAA, gồm 2.022 ca bỏ sót/thừa, 442 IoU thấp, 663 sai thuộc tính, 99 cần review, 412 bất đồng ignore và 40 sai lớp.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy kiểm được box vượt biên ảnh. Người duyệt xác định có người hoặc xe bị bỏ sót trong vùng chồng lấn.

## 5. Quyết định bàn giao

- Quyết định: Hold
- Lý do, người nhận và phạm vi sử dụng: Vòng 1 đã mở trên cả lô, 0/40 ảnh có kết quả duyệt. Bản đưa duyệt còn khung can_review.
- Nếu còn việc: Reviewer khác người sửa duyệt đủ 40 ảnh theo ảnh overlay ở work/review/round-01 và điền reviews/round-01/qc_review.csv. Chạy lại qc_report.py --round 1 và check_submission.py --round 1 sau khi duyệt.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca C001 dẫn tới img_001.jpg trong calibration_candidates.csv và box person của 263 tại [109,113,89,130] theo COCO xywh. Reviewer đối chiếu phần nhìn thấy với guideline v1.1 và ghi quyết định. Việc kiểm độc lập còn chờ thực hiện.
