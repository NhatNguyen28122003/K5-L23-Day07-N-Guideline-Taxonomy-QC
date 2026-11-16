# Guideline gán nhãn — bản cá nhân — 066

- Phiên bản: cá nhân v0.1 (dựa trên taxonomy v0)
- Ngày: 2026-11-16
- Người viết: 066
- Trạng thái: đề xuất cá nhân, chưa phải quy tắc đã thống nhất của nhóm

## 1. Mục tiêu

Gán bounding box trên ảnh đường phố để huấn luyện mô hình phát hiện người và phương tiện thuộc các lớp có trong taxonomy v0. Chỉ gán các đối tượng nhìn thấy và nhận diện được; không tự thêm lớp mới hoặc gán một vật vào lớp gần giống chỉ vì hình dáng của nó.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa dùng khi gán | Ví dụ | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Ô tô con dùng để di chuyển trên đường. | Xe con, SUV, xe con màu đỏ ở img_028. | Xe đồ chơi, xe trong tranh hoặc xe tượng. |
| `truck` | Xe tải thật dùng để chở hàng. | Xe tải có thùng hàng; xe bán tải cổ ở img_028 theo yêu cầu gán thủ công. | Bia mộ hoặc tượng có hình xe tải; tuk-tuk/xe tải ba bánh chưa có quy tắc lớp thống nhất. |
| `van` | Xe thùng kín cỡ nhỏ. | Van chở hàng hoặc chở khách. | Xe buýt, xe tải lớn. |
| `motorcycle` | Xe máy hoặc mô tô thật. | Xe máy trong bãi xe. | Xe máy đồ chơi hoặc hình xe trên vật trang trí. |
| `person` | Người thật, gồm người đi bộ hoặc ngồi trên phương tiện. | Người nhìn thấy trong đám đông. | Ma-nơ-canh, tượng người, hình in hoặc phản chiếu. |
| `ignore_region` | Vùng có vật không thuộc lớp mục tiêu hoặc cụm quá dày không thể gán từng vật nhất quán. Dùng để chỉ vùng được bỏ qua, không xem đó là một vật thể. | Xe buýt/xe đạp chưa có lớp; bia mộ hình xe tải; cụm phương tiện quá dày. | Không dùng thay cho box của một đối tượng thuộc lớp đã có và nhận diện được. |

**Vật ngoài taxonomy:** không gán ép vào lớp gần nhất. Dùng `ignore_region` cho vật/vùng ngoài phạm vi các lớp hiện có. Đề xuất này cần nhóm xác nhận trước khi dùng làm quy tắc chung.

### Thuộc tính

| Thuộc tính | Cách ghi | Quy tắc cá nhân |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Theo mức che khuất trong CVAT. V0 chưa định nghĩa ngưỡng phần trăm giữa các mức; cần nhóm thống nhất ngưỡng trước khi chuẩn hóa cách gán. |
| `truncated` | `no`, `yes` | Chọn `yes` khi đối tượng bị mép ảnh cắt; nếu không thì `no`. |
| `can_review` | tick / không | Tick khi còn nghi ngờ về lớp, ranh giới box hoặc quyết định có nên gán. |

## 3. Quy tắc vẽ box

1. Với đối tượng thuộc lớp mục tiêu, vẽ box sát phần đối tượng nhìn thấy; không ước lượng phần bị vật khác che khuất.
2. Nếu đối tượng chạm hoặc bị cắt bởi mép ảnh, đánh dấu `truncated=yes`.
3. Tách người và phương tiện thành các box riêng khi cả hai đều nhìn thấy và nhận diện được.
4. Không gán vật mô phỏng hoặc hình ảnh của vật thật như vật thể thật: tượng, bia mộ tạo hình xe, xe đồ chơi, ma-nơ-canh, hình in và phản chiếu không nhận `car`, `truck` hay `person`. Theo đề xuất cá nhân này, đánh dấu vùng liên quan bằng `ignore_region`.
5. Với cảnh dày đặc, gán các đối tượng riêng lẻ còn phân biệt rõ; đặt `ignore_region` lên cụm mà không thể tách và gán nhất quán. Không dùng vùng ignore để che các đối tượng rõ ràng đã gán được.
6. Vật cao dưới ngưỡng `min_box_height_px = 20` trong taxonomy được xử lý theo quy tắc vật nhỏ/ignore của nhóm; không tự đổi ngưỡng taxonomy.
7. Xe buýt, xe đạp và xe ba bánh chưa có lớp trong taxonomy v0. Không gán chúng vào `truck`, `van` hoặc `motorcycle` chỉ vì gần giống; dùng `ignore_region` theo lựa chọn cá nhân ở trên.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định cá nhân | Lý do |
|---|---|---|---|---|
| img_027 | Bia mộ đá hình xe tải | R-01 | Không gán `truck`; dùng `ignore_region` cho bia mộ. | Vật thật là bia mộ, không phải phương tiện. |
| img_028 | Xe bán tải cổ chở ngựa tượng và xe đỏ phía sau | R-02 | Theo yêu cầu xử lý thủ công: xe bán tải là `truck`, xe đỏ phía sau là `car`; ngựa tượng không gán như vật thật, dùng `ignore_region` nếu cần đánh dấu vùng ngoài phạm vi. | Xe nằm ngoài phần đã được giao ban đầu; cần người phụ trách gán thủ công và kiểm tra lại. |
| img_029 | Xe đạp | R-03 | Không gán xe đạp; dùng `ignore_region` cho xe đạp. Gán người nếu nhìn thấy rõ. | Taxonomy v0 chưa có lớp xe đạp. |
| img_030 | Bãi xe máy dày đặc | R-04 | Gán xe/người nổi bật, tách được; dùng `ignore_region` cho cụm không thể phân biệt từng vật. | Không bỏ qua các đối tượng riêng lẻ còn nhận diện được. |
| img_031 | Xe buýt | R-03 | Không gán xe buýt; dùng `ignore_region`. | Taxonomy v0 chưa có lớp xe buýt. |
| img_032 | Đám đông chờ xe buýt | R-04 | Gán người nhìn thấy rõ; dùng `ignore_region` cho phần đám đông quá dày, không thể tách ổn định. | Giữ nhất quán giữa các vùng có mật độ khác nhau. |
| img_033 | Ma-nơ-canh và phản chiếu trên kính | R-01 | Không gán `person`; dùng `ignore_region` cho ma-nơ-canh/phản chiếu nếu chúng nằm trong vùng cần bỏ qua. | Không phải người thật đang hiện diện trong cảnh. |
| img_034 | Xe đạp | R-03 | Không gán xe đạp; dùng `ignore_region`. Gán người nếu nhìn thấy rõ. | Taxonomy v0 chưa có lớp xe đạp. |
| img_035 | Tuk-tuk/xe ba bánh | R-03 | Không gán vào `truck` hoặc `motorcycle`; dùng `ignore_region`. | Taxonomy v0 chưa quy định lớp cho xe ba bánh. |
| img_037 | Xe buýt | R-03 | Không gán xe buýt; dùng `ignore_region`. | Taxonomy v0 chưa có lớp xe buýt. |
| img_038 | Tủ trưng bày xe đồ chơi | R-01 | Không gán xe đồ chơi là phương tiện thật; dùng `ignore_region` cho phần trưng bày. | Đồ vật mô phỏng, không phải phương tiện giao thông thật. |
| img_039 | Phố đông có xe đạp và người | R-03/R-04 | Dùng `ignore_region` cho xe đạp và vùng quá dày; gán riêng người/phương tiện thuộc lớp có sẵn nếu phân biệt được. | Xe đạp ngoài taxonomy; các vật rõ ràng vẫn cần được gán. |
| img_040 | Xe tải ba bánh | R-03 | Không tự gán `truck`; dùng `ignore_region`. | Chưa có quy tắc xác định xe ba bánh thuộc lớp nào. |

## 5. QC và phiên bản

- Dùng ngưỡng QC của taxonomy v0: IoU box lớn (cao từ 50 px) `0.85`, box nhỏ hơn `0.7`; IAA IoU trung bình `0.85`, đồng thuận lớp `95%`; lỗi theo ảnh tối đa `2%` để cân nhắc Release.
- Ghi ca chưa chắc bằng `can_review` hoặc trong `submission/qc/image_issues.csv`; không xem nhãn của người khác trước khi khóa lượt gán độc lập.
- Các quyết định R-01 đến R-04 ở trên là lựa chọn cá nhân của mã 066. Nhóm cần thống nhất trước khi đưa vào guideline chung hoặc thay taxonomy.
- Chưa thay đổi schema hay các ngưỡng trong `submission/taxonomy.json`.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| cá nhân v0.1 | Làm rõ vật mô phỏng, vật ngoài taxonomy và cảnh đông; ghi quyết định cho các ca khó đã nêu. | Các tình huống img_027–img_040 và lựa chọn cá nhân của annotator 066. |
