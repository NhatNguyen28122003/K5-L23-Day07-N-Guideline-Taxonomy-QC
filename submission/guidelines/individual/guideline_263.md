# Guideline gán nhãn cá nhân — Thuy

- Phiên bản `v0.2-thuy`
- Ngày 16/11/2026
- Phạm vi 40 ảnh `img_001.jpg` đến `img_040.jpg` trong [CVAT Job 17](http://localhost:8080/tasks/17/jobs/17)
- Người duyệt: nhóm ở Pha 2

## 1. Mục tiêu

Tạo nhãn bounding box cho mô hình phát hiện người và phương tiện trong ảnh đường phố 2D. Ưu tiên phát hiện người thật, kể cả người đang đi xe hoặc chỉ lộ một phần. Khung dư nền làm giảm độ chính xác định vị. Nhãn trên tượng, ảnh in hay phản chiếu tạo mẫu dương sai.

## 2. Lớp và thuộc tính

| Lớp | Tiêu chí | Ví dụ | Dễ nhầm |
|---|---|---|---|
| `car` | Ô tô con, hatchback, sedan hoặc SUV thiết kế chở khách | Sedan đỏ ở `img_002`, taxi ở `img_040` | Xe bán tải có thùng hàng |
| `truck` | Xe tải hoặc bán tải có khoang hàng riêng sau cabin | Xe chở ô tô ở `img_002`, xe tải xanh ở `img_036` | Van có thân liền khoang |
| `van` | Xe thân liền, dáng thùng kín cỡ nhỏ dùng chở khách hoặc hàng | Van bạc ở `img_016` | Xe buýt, bán tải lắp nắp thùng |
| `motorcycle` | Xe máy hoặc mô tô, kể cả mô tô có sidecar gắn liền | Xe tay ga ở `img_010`, mô tô sidecar ở `img_022` | Xe đạp, xích lô, tuk-tuk |
| `person` | Người thật nhìn thấy trực tiếp, gồm người đi bộ, người điều khiển và hành khách | Người đi bộ ở `img_034`, hai người trên xe ở `img_026` | Ma-nơ-canh, tượng, ảnh in |
| `ignore_region` | Vùng có cụm mục tiêu quá nhỏ hoặc quá dày để phân tách đáng tin cậy | Cụm xe ở xa trong `img_021` | Khung của một đối tượng đã nhận dạng được |

Xe buýt, tuk-tuk, xe đạp và xích lô nằm ngoài sáu lớp của Job 17. Người thật trên các xe này vẫn thuộc `person` nếu đủ điều kiện. Ghi ca cần bổ sung lớp vào nhật ký cấp ảnh để nhóm quyết định ở Pha 2.

| Trường CVAT | Giá trị và cách chọn |
|---|---|
| `Occluded` | Bật khi vật khác che một phần đối tượng. Tắt khi phần đối tượng nằm trong ảnh được thấy đầy đủ. |
| `occluded_level` | `0` nếu che 0%, `1` nếu che trên 0% đến 30%, `2` nếu che trên 30% đến 70%, `3` nếu che trên 70% mà lớp vẫn nhận ra được. Ước lượng theo diện tích hình dáng đối tượng đáng lẽ xuất hiện bên trong ảnh. |
| `truncated` | `yes` nếu mép ảnh cắt đối tượng, `no` nếu đối tượng nằm trọn trong ảnh. Một đối tượng có thể vừa bị cắt vừa bị che. |
| `can_review` | Bật khi còn nghi ngờ về lớp, ranh giới khung hoặc mức che. Nếu chưa thể đặt khung đáng tin cậy, ghi vấn đề ở cấp ảnh. |

## 3. Quy tắc vẽ box

- **R-01 — Ôm phần nhìn thấy.** Dùng hình chữ nhật nhỏ nhất chứa mọi phần nhìn thấy thuộc cùng một đối tượng. Hai phần lộ ra hai bên vật che vẫn dùng một khung. Dừng ở mép ảnh và bỏ phần hàng hóa rời, bóng đổ khỏi khung.
- **R-02 — Che khuất và cắt mép.** Vật che có thể là người, xe, cột hoặc hàng rào. Bật `Occluded` khi mức che lớn hơn 0. Đánh giá phần cắt ở mép bằng `truncated`. Ảnh nhòe do chuyển động chỉ cần `can_review` nếu hình dáng khó xác định.
- **R-03 — Người trên xe.** Vẽ một khung cho mỗi người đủ điều kiện và một khung riêng cho xe thuộc taxonomy. Hai khung có thể chồng nhau. Người bị thân xe hoặc người khác che được đánh mức che theo phần cơ thể bị khuất.
- **R-04 — Xe chở xe.** Vẽ một khung cho xe tải chở xe, gồm cabin và giàn chở, rồi thêm một khung cho từng xe thật nhận dạng được trên giàn. Thanh giàn che xe con được tính vào mức che của xe con.
- **R-05 — Hình ảnh gián tiếp và vật mô phỏng.** Bỏ qua tượng, ma-nơ-canh, đồ chơi, ảnh in, phản chiếu và bộ phận của xe chụp ảnh. Giữ ảnh trong bộ dữ liệu với 0 nhãn nếu cả ảnh chỉ có các trường hợp này.
- **R-06 — Vật nhỏ và đám đông.** Dùng chiều cao khung phần nhìn thấy ở ảnh gốc. Gán từng vật từ 20 px nếu phân biệt được. Vật dưới 20 px đứng riêng được bỏ qua. Dùng `ignore_region` ôm cụm quá nhỏ hoặc chồng lấn dày, thu vùng ignore để tránh che mất các vật rõ ở tiền cảnh.
- **R-07 — Xe ngoài lớp và xe cải tạo.** Dựa vào kết cấu thân xe để chọn lớp. Bán tải có thùng hàng riêng vẫn xếp `truck` khi lắp nắp thùng. Bật `can_review` nếu ảnh không cho thấy rõ kết cấu. Ghi xe buýt và tuk-tuk ở cấp ảnh vì Job 17 chưa có lớp tương ứng.

## 4. Ca khó từ bộ 40 ảnh

| Ảnh | Tình huống | Quy tắc | Quyết định và lý do |
|---|---|---|---|
| `img_002` | Giàn chở ô tô che nhiều xe | R-02, R-04 | Một khung xe tải và tám khung xe con. Đánh dấu rà lại xe tầng dưới chỉ lộ một phần nhỏ. |
| `img_004`, `img_035` | Tuk-tuk và người điều khiển | R-03, R-07 | Gán người nhận dạng được. Ghi tuk-tuk ở cấp ảnh để nhóm xem xét thêm lớp. |
| `img_009`, `img_037`, `img_040` | Có xe buýt | R-07 | Ghi xe buýt ở cấp ảnh vì thiếu lớp `bus`. |
| `img_014` | Người và xe nhòe chuyển động | R-01, R-02 | Giữ khung cho vật còn nhận dạng được. Bật `can_review` nếu biên khó xác định. |
| `img_017`, `img_020`, `img_027`, `img_033`, `img_038` | Tượng, phản chiếu, ma-nơ-canh hoặc đồ chơi | R-05 | Giữ ảnh với 0 nhãn. |
| `img_018` | Chân dung trên biển và xe sau hàng rào | R-02, R-05 | Bỏ chân dung in. Gán từng xe nhìn rõ và bật `Occluded` do hàng rào che. |
| `img_019` | Xe diễu hành và đám đông | R-06, R-07 | Tạm gán `car` và bật `can_review` cho xe cải tạo. Dùng ignore cho vùng đám đông khó tách. |
| `img_021`, `img_030`, `img_032` | Nhiều xe máy hoặc người chồng lấn | R-06 | Gán riêng các vật phân biệt được, dùng ignore cho phần dày đặc. |
| `img_026`, `img_034` | Người đi xe và xe có thể che nhau | R-02, R-03 | Gán riêng từng người và xe, xét che khuất và cắt mép cho từng khung. |
| `img_037` | Bán tải gắn nắp thùng cao | R-07 | Tạm gán `truck` theo thùng hàng riêng và bật `can_review`. |
| `img_039` | Người đi xe đạp | R-03, R-07 | Gán người đi xe. Xe đạp nằm ngoài taxonomy của job. |

## 5. QC và phiên bản

Rà cả 40 ảnh theo thứ tự. Kiểm tra đối tượng sót, khung trùng, lớp sai, khung dư nền và sự nhất quán giữa `Occluded`, `occluded_level` và `truncated`. Kiểm tra riêng năm ảnh dự kiến có 0 nhãn và toàn bộ khung có `can_review`.

Áp dụng ngưỡng trong `submission/taxonomy.json`. Ghép khung để so sánh ở IoU từ 0,50. Ngưỡng chất lượng khung là IoU từ 0,85 cho khung cao từ 50 px và từ 0,70 cho khung cao 20–49 px. Mục tiêu IAA là IoU ≥ 0,85 và đồng thuận lớp ≥ 0,95. Ngưỡng lỗi theo ảnh là tối đa 2%. Ghi mã ảnh, đối tượng, loại lỗi và mã quy tắc khi phát hiện sai lệch. Reviewer của nhóm chốt các ca mơ hồ trước khi dùng bản thống nhất để gán lại hoặc chấm QC.

| Phiên bản | Thay đổi | Căn cứ |
|---|---|---|
| `v0.2-thuy` | Quy tắc khung phần nhìn thấy, mức che, cắt mép, vật nhỏ và xe ngoài lớp | Rà soát 40 ảnh Job 17 |

