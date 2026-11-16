# Guideline gán nhãn — v0.1 (bản tham khảo cá nhân)

- **Phiên bản:** v0.1 — đề xuất
- **Ngày:** 16/11/2026
- **Mã thành viên:** [mã 3 chữ số]
- **Công cụ:** CVAT
- **Loại nhãn:** Bounding Box
- **Định dạng xuất:** COCO 1.0

## 1. Mục tiêu

Xây dựng bộ quy tắc gán nhãn nhất quán cho 40 ảnh đường phố, phục vụ huấn luyện và đánh giá mô hình phát hiện người và phương tiện giao thông.

Guideline cần giảm các lỗi thường gặp như nhầm loại phương tiện, bỏ sót đối tượng, gộp nhiều đối tượng thành một box hoặc gán nhãn lên vật không phải đối tượng thật.

Ưu tiên tính nhất quán giữa các người gán nhãn. Khi xuất hiện tình huống mơ hồ, cần ghi nhận để thảo luận thay vì tự suy đoán.

## 2. Lớp và thuộc tính

### 2.1. Định nghĩa lớp

| Class | Định nghĩa | Ví dụ / trường hợp dễ nhầm |
|---|---|---|
| `car` | Ô tô con, sedan, hatchback, SUV và taxi dạng ô tô con | Không nhầm với xe bán tải hoặc xe tải |
| `truck` | Xe tải, xe bán tải, xe chở hàng có thùng hoặc khoang hàng riêng | Xe tải chở ô tô, pickup, xe tải nhỏ |
| `van` | Xe có thân kín liền khối, sử dụng để chở người hoặc hàng hóa quy mô nhỏ | Xe van, không bao gồm xe buýt lớn |
| `motorcycle` | Xe máy, mô tô và xe tay ga; đề xuất bao gồm mô tô có sidecar | Không nhầm với xe đạp hoặc xe ba bánh chở khách dạng tuk-tuk |
| `person` | Người thật xuất hiện trong ảnh, bao gồm người đi bộ, ngồi, lái xe hoặc ngồi sau xe | Không gán tượng, ma-nơ-canh, người trên áp phích |
| `ignore_region` | Vùng không thể gán từng đối tượng một cách đáng tin cậy theo quy tắc | Nhóm vật quá nhỏ hoặc khu vực quá dày đặc để tách vật |

Đối tượng không thuộc taxonomy thì không tự tạo class mới. Ghi nhận trường hợp gây tranh cãi để nhóm thống nhất.

### 2.2. Thuộc tính

**`occluded_level` — mức độ che khuất (đề xuất)**

- `0`: Không bị vật khác che đáng kể.
- `1`: Bị che nhẹ, ước tính trên 0% đến 25%.
- `2`: Bị che trung bình hoặc nhiều, trên 25% đến 75%.
- `3`: Bị che trên 75%, hoặc không thể đánh giá đáng tin cậy phần bị che.

Đây là ngưỡng đề xuất; nhóm cần chốt trước khi gán độc lập.

**`truncated`**

- `no`: Đối tượng không bị cắt bởi mép ảnh.
- `yes`: Một phần đối tượng nằm ngoài khung ảnh.

**`can_review`**

Đánh dấu khi chưa chắc về class, ranh giới box, thuộc tính hoặc tính hợp lệ của đối tượng.

## 3. Quy tắc vẽ bounding box

**R-01 — Chỉ gán đối tượng thuộc phạm vi**

Chỉ gán người thật và phương tiện thuộc các lớp đã khai báo. Không tự suy diễn lớp cho vật không có trong taxonomy.

**R-02 — Một đối tượng, một box**

Mỗi vật thể độc lập phải có box riêng, kể cả khi hai vật cùng loại đứng sát hoặc che nhau. Không gộp hai xe máy thành một box.

**R-03 — Box ôm sát phần nhìn thấy**

Box được vẽ sát phạm vi nhìn thấy của đối tượng. Không tự mở rộng box dựa trên phần bị che mà không có bằng chứng hình ảnh.

**R-04 — Người và xe được gán độc lập**

Người lái xe, người ngồi sau và phương tiện là các đối tượng khác nhau. Ví dụ một xe máy chở hai người có thể cần ba box: hai `person`, một `motorcycle`.

**R-05 — Xe chở và xe được chở**

Khi xe tải đang vận chuyển các ô tô khác, gán xe tải và từng ô tô nhìn thấy được bằng các box riêng. Không coi toàn bộ cụm là một chiếc xe.

**R-06 — Đối tượng thật và hình mô phỏng**

Không gán người hoặc xe xuất hiện dưới dạng hình in, tượng, ma-nơ-canh, đồ chơi hoặc phản chiếu qua gương. Xe có thiết kế lạ nhưng thực sự đang hoạt động vẫn có thể thuộc class phù hợp.

**R-07 — Vật thể quá nhỏ**

Áp dụng ngưỡng chiều cao box tối thiểu 20 px theo taxonomy của lab. Những vật nhỏ hơn ngưỡng không được tự tạo box đối tượng thông thường; xác định vùng `ignore_region` thích hợp nếu cần.

Đo kích thước trên ảnh gốc, không theo mức phóng to của CVAT.

**R-08 — Vật bị che và bị cắt mép**

Vật bị che vẫn được gán nếu nhận diện được. Nếu đối tượng bị cắt bởi mép ảnh thì đặt `truncated=yes`. Che khuất bởi xe khác không đồng nghĩa với bị cắt mép.

**R-09 — Phân loại phương tiện theo cấu trúc**

Phân biệt `truck` và `van` dựa trên cấu tạo thân xe và khoang hàng, không chỉ dựa vào kích thước hoặc màu sắc. Xe pickup có nắp che khoang hàng vẫn ưu tiên `truck`.

**R-10 — Mật độ đối tượng lớn**

Vẫn gán riêng từng người hoặc phương tiện nếu nhận diện được và đạt ngưỡng kích thước. Chỉ đề xuất `ignore_region` cho những khu vực thực sự không thể tách đối tượng đáng tin cậy; không bỏ qua cả đám đông chỉ vì có nhiều vật.

**R-11 — Ảnh mờ, thiếu sáng và phản chiếu**

Ảnh chuyển động hoặc thiếu sáng vẫn được gán nếu đối tượng và phạm vi nhìn thấy đủ rõ. Không dùng `occluded_level` để biểu diễn độ mờ chuyển động.

**R-12 — Trường hợp chưa xác định**

Ưu tiên ghi ảnh, vị trí, cách hiểu đang áp dụng và lý do chưa chắc chắn. Box chưa chắc được đánh dấu `can_review`; vấn đề không có box phù hợp được ghi ở cấp ảnh.

## 4. Các tình huống khó trong 40 ảnh

Các trường hợp dưới đây là phương án phân loại đề xuất để thảo luận và kiểm chứng.

| Ảnh | Tình huống | Rule ID | Quyết định đề xuất và lý do |
|---|---|---|---|
| `img_002` | Xe tải chở nhiều ô tô | R-05 | Gán `truck` cho xe chuyên chở, `car` riêng cho từng ô tô nhìn thấy |
| `img_004` | Tuk-tuk ba bánh | R-01 | Đề xuất ngoài taxonomy, không ép thành `car` hoặc `motorcycle`; cần thống nhất |
| `img_005` | Nhiều người bị xe và người khác che | R-04, R-08 | Tách các `person` nhận diện được, không gộp theo nhóm |
| `img_009` | Xe buýt nhỏ | R-01 | Cần chốt ranh giới `van` và bus; đề xuất không ép xe buýt vào `van` |
| `img_014` | Xe máy chuyển động gây nhòe | R-11 | Chỉ gán nếu nhận diện được loại xe và biên đủ đáng tin cậy |
| `img_017` | Tượng người cưỡi ngựa | R-06 | Không gán `person` vì không phải người thật |
| `img_018` | Người trên biển quảng cáo và xe sau hàng rào | R-06, R-08 | Không gán người trên biển; xe thật sau hàng rào có thể được gán phần nhìn thấy |
| `img_019` | Ô tô thiết kế giả tưởng trong đoàn diễu hành | R-06 | Đề xuất `car` nếu là xe thật có khả năng vận hành, không phải mô hình tĩnh |
| `img_020` | Ô tô phản chiếu trong gương chiếu hậu | R-06 | Không gán xe trong phản chiếu |
| `img_021` | Hàng loạt xe máy và người trong cảnh ùn tắc | R-02, R-10 | Tách từng đối tượng rõ ràng, xem xét `ignore_region` cho vùng quá dày đặc |
| `img_022` | Mô tô có thùng phụ sidecar | R-09 | Đề xuất một box `motorcycle` cho cụm xe và sidecar |
| `img_023` | Xích lô ba bánh | R-01 | Xích lô ngoài taxonomy; người điều khiển được gán `person` nếu đủ rõ |
| `img_024` | Ô tô trưng bày và xe nhỏ trên nóc | R-06 | Xe trưng bày thật và mô hình cần phân biệt; đánh dấu review cho xe trên nóc |
| `img_026` | Hai người trên cùng xe máy | R-04 | Hai box `person` và một box `motorcycle` |
| `img_027` | Tượng hoặc mô hình xe bán tải | R-06 | Không gán `truck` nếu xác nhận là vật trưng bày không phải xe vận hành |
| `img_028` | Xe bán tải chở tượng ngựa | R-06, R-09 | Gán xe `truck`, không gán tượng ngựa |
| `img_030` | Nhiều xe máy đỗ chồng lấn | R-02, R-08 | Mỗi xe nhận diện được có box riêng |
| `img_032` | Đám đông đứng chờ bên đường | R-10 | Gán người đủ rõ; khu vực không tách được cần xử lý thống nhất |
| `img_033` | Ma-nơ-canh và hình quảng cáo trong cửa kính | R-06 | Không gán ma-nơ-canh hay người trong ảnh in |
| `img_035` | Tuk-tuk chở hàng cồng kềnh | R-01 | Không tự chuyển tuk-tuk thành `truck` chỉ vì chở hàng |
| `img_037` | Pickup có khoang che kín | R-09 | Đề xuất `truck` do kết cấu xe bán tải |
| `img_038` | Nhiều ô tô mô hình trong cửa hàng | R-06 | Không gán `car` cho đồ chơi |
| `img_040` | Taxi, tuk-tuk và người ở tiền cảnh | R-01, R-08 | Taxi sedan thuộc `car`; tuk-tuk cần quy tắc riêng; người thật vẫn được gán |

## 5. Kiểm soát chất lượng và phiên bản

### 5.1. Ngưỡng kiểm tra

Áp dụng các ngưỡng có sẵn trong `taxonomy.json`, không tự ý sửa:

- Chiều cao box tối thiểu: 20 px.
- Box cao từ 50 px: IoU yêu cầu từ 0.85.
- Box nhỏ hơn 50 px: IoU yêu cầu từ 0.70.
- IAA IoU trung bình: ít nhất 0.85.
- IAA đồng thuận class: ít nhất 95%.
- Tỷ lệ ảnh lỗi tối đa để Release: 2%.

### 5.2. Xử lý bất đồng

Khi hai người gán nhãn khác nhau, kiểm tra theo thứ tự: khác class, thiếu hoặc thừa đối tượng, sai vị trí box và sai thuộc tính.

Nếu bất đồng xuất phát từ guideline không rõ thì cần bổ sung quy tắc. Nếu guideline đã rõ nhưng người gán áp dụng sai thì cần sửa nhãn và kiểm tra lại.

Không thay đổi guideline đã khóa trong lúc gán nhãn độc lập.

### 5.3. Lịch sử phiên bản

| Phiên bản | Thay đổi | Trạng thái |
|---|---|---|
| v0 | Guideline khởi đầu của lab | Có sẵn |
| v0.1 | Các phương án bổ sung để thảo luận | Chưa thống nhất |
| v1.0 | Bản nhóm thống nhất | Chưa thực hiện |
| v1.1 | Bản hiệu chỉnh sau calibration | Chưa thực hiện |

Mọi quyết định sửa đổi cần có mã quy tắc, ảnh minh họa, người quyết định và bằng chứng kiểm tra lại. Không ghi kết quả IAA hoặc QC trước khi thực sự chạy các bước tương ứng.