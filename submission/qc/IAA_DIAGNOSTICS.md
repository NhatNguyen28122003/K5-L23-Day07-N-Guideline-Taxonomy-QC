# Phân tích IAA và các nguyên nhân bất đồng — dữ liệu tự động

> Tài liệu bổ sung cho phần lập luận của rubric. Các nhận xét dưới đây là **chẩn đoán từ CSV và guideline**; chưa phải quyết định calibration hay bằng chứng đã xem tất cả ảnh bằng mắt.

## Phạm vi và mẫu số

- 40 ảnh, 5 mã người gán: 066, 263, 273, 277, 291.
- 176/176 lượt gán theo assignment. Mã 066 có 16 ảnh, bốn mã khác có 40 ảnh/mã.
- 1.137 **cặp box ghép** dùng tính mean IoU 0,8383 và độ đồng thuận lớp 96,48%.
- Độ đồng thuận thuộc tính `occluded_level` chỉ 47,67%; `truncated` 87,07%: vấn đề hiệu chuẩn thuộc tính đáng chú ý hơn sai lớp.
- 999/1.260 box chưa ghép hai phía: **không** lấy số này làm số box sai thực tế nếu chưa xem ảnh.

## Phân loại 3.678 ứng viên bất đồng

| Loại ứng viên | Số ca | Hướng xử lý đúng |
|---|---:|---|
| Bỏ sót / thừa | 2.022 | Kiểm tra vật có thật hay không, đủ ngưỡng kích thước và nằm trong scope hay không; không chốt chỉ theo số đông |
| Thuộc tính khác | 663 | So `occluded_level`, `truncated` với định nghĩa v1.1 và mép ảnh |
| IoU thấp | 442 | Kiểm tra mép box, phần nhìn thấy và cắt ảnh (R-02) |
| Bất đồng ignore | 412 | Xem có cụm đủ điều kiện ignore hay phải vẽ từng vật (R-08) |
| Có `can_review` | 99 | Xác định lớp / đối tượng có thuộc scope (R-01/R-03), rồi gỡ cờ khi đã xác nhận |
| Khác lớp | 40 | Phân loại theo kết cấu xe và taxonomy (R-03) |

## Trường hợp có mã để giải thích

- **C001 / img_001.jpg, 263|273:** `person` `[109,113,89,130]` chỉ xuất hiện phía 263. Đây là *bất đồng hiện diện*, chưa chứng minh ai sai. Nếu người là thật và hợp lệ thì cần bổ sung nhãn còn thiếu; nếu không, cần bỏ box thừa.
- **C033 / img_001.jpg, 263|291:** Cùng `person`, box `[85,98,127,338]` và `[94,103,135,327]`, IoU **0,80**. Loại lỗi là *định vị*, không phải sai lớp; đối chiếu R-02 và R-15.
- **C048 / img_001.jpg, 263|291:** Cùng `person`, IoU **0,97** nhưng khác `truncated` (`263=yes`, `291=no`). IAA về hình học có thể rất cao trong khi thuộc tính chưa thống nhất. Đối chiếu quy định cắt mép ảnh.
- **C296 / img_004.jpg, 273|291:** IoU **0,96**, nhưng `273=other_vehicle`, `291=car`. Vấn đề chính là taxonomy / phân lớp theo cấu tạo, không phải vị trí box (R-03). Không chọn một bên khi chưa xem ảnh.
- **C372 / img_005.jpg, 263|291:** IoU **0,94**, nhưng `263=van`, `291=truck`; cần kiểm kết cấu cabin và khoang sau (R-03).
- **C608 / img_007.jpg, 263|273:** IoU **0,52** và khác lớp (`truck`/`car`), là ca có thể đồng thời lệch vùng bao và quy tắc phân loại.
- **C891 / img_011.jpg, 263|273:** Vùng `ignore_region` không ghép được với người còn lại; cần xác định cụm vật thuộc ngưỡng R-08 trước khi giữ hoặc bỏ vùng ignore.
- **C1000 / img_012.jpg, 277|291:** Cùng lớp `truck`, IoU **0,92**, nhưng khác mức che (`277=2`, `291=0`); cần dùng cùng một định nghĩa occlusion (R-14).

## Vì sao IAA cao không đủ chứng minh đúng?

IAA chỉ đo **mức giống nhau giữa các người gán trên box ghép được**. Hai người cùng bỏ sót vật có thể không làm giảm mean IoU. Hai người cùng gán nhầm tượng/phản chiếu cũng có thể thống nhất cao nhưng không tuân quy tắc. Vì vậy cần bộ ảnh đối chứng và kiểm bằng mắt trước khi Release.

## Các ảnh cần ưu tiên nếu tiếp tục calibration

Theo `image_scores.csv`, năm ảnh có nhiều ứng viên nhất là `img_032` (282), `img_034` (267), `img_039` (238), `img_021` (196), `img_006` (181). Bản đồ ưu tiên này là để tập trung review, **không** thay thế phiếu chốt ca.

## Vai trò mã 291

`submission/labels/291.json` có **40 ảnh và 311 annotation**; có guideline riêng và nhiều ca IAA liên quan, trong đó C033 (IoU), C048 (truncated), C296/C372 (sai lớp), C1000 (mức che). Đây là dấu vết cụ thể của đầu vào và những điểm cần cải thiện, **không** khẳng định 291 đã tự duyệt hoặc sửa tất cả các ca.

## Giới hạn

- Không có quyết định nào trong 3.678 dòng được xác nhận tại thời điểm đối chiếu.
- Những trường hợp nêu ở trên là **ví dụ để thảo luận và kiểm tiếp**, không được đếm là đã được giải quyết.
- Khi thay đổi guideline loại `semantic` cần rà lại toàn bộ 40 ảnh trước khi phát hành bản nhãn cuối.
