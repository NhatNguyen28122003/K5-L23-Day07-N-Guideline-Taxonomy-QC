# Guideline v1.0 – Gán nhãn 40 ảnh đường phố

**Lab K5-L23 Day07** · Bộ ảnh `img_001` … `img_040` · Công cụ: CVAT (bounding box) · Phiên bản 1.0 (bản nháp để hiệu chuẩn)

> **Ghi chú:** taxonomy dưới đây là đề xuất dựa trên việc xem cả 40 ảnh. Nếu đề lab quy định sẵn danh sách lớp hoặc định dạng khác, hãy đổi tên lớp cho khớp nhưng giữ nguyên các quy tắc quyết định.

---

## 1. Mục tiêu và phạm vi

Bộ ảnh gồm cảnh giao thông đô thị châu Á (Việt Nam, Thái Lan, Đài Loan, Ấn Độ…) và một số ca đối chứng (tượng, đồ chơi, ma-nơ-canh, phản chiếu, biển quảng cáo). Mục tiêu: khoanh khung (bounding box) các phương tiện và người đi bộ thật, nhìn thấy trực tiếp trong ảnh, để huấn luyện/đánh giá bộ phát hiện đối tượng giao thông.

**Nguyên tắc gốc:** chỉ gán nhãn vật thể thật, đang hiện diện ngoài đời. Vật thể là hình vẽ, tượng, mô hình, đồ chơi, ảnh phản chiếu, hình trên biển quảng cáo thì **không** gán nhãn.

---

## 2. Danh sách lớp (taxonomy)

| ID | Lớp | Gồm | Không gồm |
|----|-----|-----|-----------|
| 1 | `motorcycle` | Xe máy, xe tay ga, mô tô, xe có thùng phụ (sidecar). Khung bao cả xe và người ngồi trên xe (người lái, người ngồi sau). | Xe đạp, xe đạp điện không có động cơ rõ ràng (xếp `bicycle`) |
| 2 | `car` | Xe con, SUV, taxi, van/minivan, bán tải (pickup) cỡ nhẹ kể cả có nắp thùng. | Xe tải thùng, xe đầu kéo |
| 3 | `truck` | Xe tải, xe tải nhỏ có thùng hàng cao hơn ca-bin, xe chở xe (car carrier), xe đầu kéo. | Pickup thùng hở |
| 4 | `bus` | Xe buýt, xe khách, minibus chở khách cỡ lớn. | Van/minivan (xếp `car`) |
| 5 | `three_wheeler` | Tuk-tuk, xe lam, auto-rickshaw, xe ba bánh có động cơ. | Xe ba gác đạp tay (xếp `bicycle`) |
| 6 | `bicycle` | Xe đạp, xích lô đạp, xe ba gác đạp. Khung bao xe và người đạp. | Xe có động cơ |
| 7 | `person` | Người không ngồi trên phương tiện: đi bộ, đứng, ngồi quán, người bán hàng. | Người đang ngồi trên xe (đã nằm trong khung xe), ma-nơ-canh, tượng, hình người trên biển quảng cáo |

Không có lớp "khác". Vật không thuộc 7 lớp trên thì bỏ qua, không tự thêm lớp.

---

## 3. Thuộc tính (attributes)

Mỗi khung gán các thuộc tính sau:

| Thuộc tính | Giá trị | Quy tắc |
|------------|---------|---------|
| `occlusion` | `0` thấy ≥ 80% · `1` thấy 40–79% · `2` thấy < 40% (chỉ gán nếu vẫn nhận ra loại) | Ước lượng bằng mắt phần thân thể/thân xe còn thấy |
| `truncated` | `yes` / `no` | `yes` nếu vật bị cắt bởi mép ảnh |
| `difficult` | `yes` / `no` | `yes` nếu nhòe chuyển động, thiếu sáng, ngược sáng, ướt/phản chiếu mạnh, hoặc quá nhỏ nhưng vẫn gán |
| `parked` | `yes` / `no` | `yes` nếu xe đỗ, không có người điều khiển |

**Thuộc tính cấp ảnh** (ghi ở bảng theo dõi, không gán trong CVAT): `time` (ngày / đêm), `image_type` (màu / đen trắng), `control_case` (có / không).

---

## 4. Quy tắc vẽ khung

1. **Khung chặt:** cạnh khung chạm sát phần nhìn thấy của vật, không chừa viền thừa quá 2 px, không cắt mất phần vật.
2. **Vật bị che:** vẽ khung theo phần nhìn thấy, không đoán phần bị che. Riêng xe bị che một phần nhưng nhìn rõ hình dạng, có thể kéo khung theo phần suy ra hợp lý của thân xe (tối đa 10%).
3. **Kích thước tối thiểu:** cạnh ngắn của khung < 20 px (trên ảnh cạnh dài 1024 px) thì không gán nhãn và không cần vùng bỏ qua.
4. **Vật bị cắt mép ảnh:** vẫn gán nếu còn thấy ≥ 30% và nhận ra loại; đánh `truncated = yes`.
5. **Đám đông / bãi xe dày đặc** (ví dụ dòng xe máy kín đường, bãi đỗ xe máy xếp sát, hàng người chờ xe buýt): gán nhãn từng vật ở tiền cảnh và trung cảnh nhìn thấy rõ ≥ 40%; phần phía sau chen chúc không tách nổi thì vẽ một khung vùng bỏ qua (ghi chú `ignore`) thay vì đoán.
6. **Mỗi vật một khung:** không vẽ hai khung chồng lên cùng một vật; không vẽ khung cho phần của vật (ví dụ chỉ bánh xe).
7. **Hàng hóa, đồ mang theo** (gánh hàng, bao tải, thanh dài chở trên xe, thùng hàng trên xe bán tải): không gán nhãn riêng và không tách khỏi khung vật chở nó. Khung bao gồm hàng hóa nếu hàng nằm trong thân xe; hàng nhô ra ngoài thì không bao phần nhô.
8. **Xe đang được chở trên xe khác** (xe chở xe): chỉ gán `truck` cho xe chở; xe được chở không gán.
9. **Nhìn thấy gương hoặc kính chiếu hậu:** vật trong gương hoặc trong vũng nước phản chiếu không gán.

---

## 5. Quy tắc phân loại khi mơ hồ

- **Pickup hay truck?** Có thùng hở và kích thước cỡ xe con → `car`. Thùng kín/cao hơn ca-bin hoặc xe tải nhỏ kiểu kei-truck → `truck`.
- **Minibus hay van?** Chở ≥ 15 chỗ, dáng buýt thu nhỏ → `bus`; còn lại → `car`.
- **Xe đạp điện / xe máy điện:** có động cơ, yên dạng xe máy → `motorcycle`; xe đạp có trợ lực, vẫn đạp bàn đạp → `bicycle`.
- **Taxi, xe ôm công nghệ:** phân loại theo loại xe (`car` hoặc `motorcycle`), áo/biển hiệu không đổi lớp.
- **Người dắt xe đạp/xe máy đi bộ:** nếu người đứng cạnh xe → `person` và xe là lớp tương ứng (hai khung riêng); nếu ngồi/ngồi gác chân lên xe → gộp vào khung xe.
- **Người đứng trên thùng xe hoặc ngồi trên xe tải:** coi như hành khách, không gán `person` riêng.

---

## 6. Các ca đối chứng (không gán nhãn)

Các trường hợp sau là bẫy thường gặp. Ghi vào bảng theo dõi là `control_case = có` và không vẽ khung cho phần giả:

| Trường hợp | Quy tắc |
|------------|---------|
| Tượng, điêu khắc (kể cả tượng xe tải, tượng ngựa, tượng người cưỡi ngựa) | Không gán |
| Đồ chơi, mô hình xe thu nhỏ (kệ xe mô hình, xe đồ chơi đặt trên nóc xe thật) | Không gán |
| Ma-nơ-canh | Không gán `person` |
| Hình người/xe trên biển quảng cáo, tranh tường, pano | Không gán |
| Phản chiếu (kính chiếu hậu, mặt đường ướt, cửa kính) | Không gán |
| Xe thật nhưng độc đáo (xe diễu hành trang trí, xe cổ trong bảo tàng, xe cổ trong ảnh đen trắng) | Vẫn gán theo loại xe, đánh `difficult = yes` nếu trang trí che dáng xe |
| Vật phụ trợ (nón giao thông, đèn, biển báo) | Không thuộc lớp nào, bỏ qua |

---

## 7. Quy ước chung

- Ảnh đen trắng, thiếu sáng, nhòe: vẫn gán nếu người gán tự tin nhận ra loại. Không chắc chắn → không gán, ghi chú trong cột `note` để thảo luận.
- Không sửa, đổi tên, cắt hay thêm/bớt ảnh. Tên file giữ nguyên `img_001.jpg` … `img_040.jpg`.
- Khuôn mặt và biển số trong ảnh là dữ liệu gốc: không che hay chỉnh sửa trong bước gán nhãn; việc xử lý riêng tư thuộc bước phát hành sau.
- Nhãn xuất theo định dạng mà đề lab yêu cầu (thường CVAT XML/COCO). Kiểm tra lại tên lớp và thuộc tính trước khi xuất.

---

## 8. Ca khó theo từng ảnh (dùng để hiệu chuẩn nhóm)

Bảng dưới là gợi ý dự kiến sau khi xem nhanh ảnh. Hai người gán nhãn độc lập rồi so khớp; chỗ nào lệch thì dùng cột này để thống nhất và cập nhật guideline.

| Ảnh | Nội dung chính | Điểm cần thống nhất |
|-----|----------------|---------------------|
| 001 | Quán ăn ngoài trời, pickup đỗ, người ngồi/đứng | Pickup → `car`, `parked = yes`; người ngồi quán → `person` |
| 002 | Xe chở xe | Gán `truck` cho xe chở; các xe trên giàn không gán |
| 003 | Người bán rong, xe đạp, gánh hàng | Gánh hàng không gán riêng; người đứng → `person` |
| 004 | Hai xe tuk-tuk đỗ ban đêm | `three_wheeler`, `parked = yes` |
| 005 | Cận cảnh xe máy, van phía sau | Người trong khung xe máy; khung bị cắt mép → `truncated` |
| 006 | Xe máy, xe đạp từ phía sau | Gộp người vào khung xe; xe đạp → `bicycle` |
| 007 | Xe bán tải và xe tải trong xưởng | Pickup → `car`; xe tải thùng → `truck` |
| 008 | Xe máy đỗ, tài xế áo cam, ban đêm | `difficult` do thiếu sáng; xe đỗ dày → vùng bỏ qua |
| 009 | Xe khách mini ở ngã tư | Phân loại `bus` hay `car` (xem mục 5) |
| 010 | Chợ, xe máy kèm người bán | Người đứng bán → `person`; xe máy có người ngồi → gộp |
| 011 | Điêu khắc kim loại, xe đạp xa | Điêu khắc không gán; xe đạp nhỏ xa dưới ngưỡng kích thước |
| 012 | Pickup chở hàng và xe con | Hàng trên thùng không gán riêng |
| 013 | Đường đêm gần như vắng | Có thể chỉ còn vật rất nhỏ, ghi `difficult` nếu vẫn gán |
| 014 | Người đi bộ rõ, xe máy nhòe | Người nhòe nhận ra loại vẫn gán, `difficult = yes` |
| 015 | Cận cảnh đường ướt, xe bị cắt mép | `truncated`, phản chiếu không gán |
| 016 | Tuk-tuk, van, xe tay ga, người | Nhiều lớp trong một ảnh, kiểm tra nhầm `three_wheeler` với `car` |
| 017 | Tượng ngựa có nón giao thông | Ca đối chứng, không gán |
| 018 | Biển quảng cáo có mặt người, xe xa sau hàng rào | Mặt người trên biển không gán; xe quá nhỏ bỏ qua |
| 019 | Xe diễu hành vàng trang trí, đám đông | Xe thật → `car` + `difficult`; đám đông → vùng bỏ qua, người đứng rõ phía trước → `person` |
| 020 | Kính chiếu hậu có phản chiếu đường | Ca đối chứng, không gán |
| 021 | Dòng xe máy dày đặc ban đêm | Gán tiền cảnh, phần chen chúc phía sau → vùng bỏ qua |
| 022 | Ảnh đen trắng xe mô tô sidecar trên đường tuyết | Xe thật → `motorcycle`, ghi `image_type = đen trắng` |
| 023 | Xích lô và xe con | Xích lô → `bicycle`; xe con → `car` |
| 024 | Xe cổ trong bảo tàng, xe đồ chơi trên nóc | Xe thật → `car`; mô hình trên nóc không gán |
| 025 | Đại lộ ven biển, người và xe xa | Phần lớn dưới ngưỡng kích thước, chỉ gán vật đủ lớn |
| 026 | Xe máy hai người trong hầm, tranh tường | Tranh tường không gán |
| 027 | Tượng xe bán tải đen trên bệ | Ca đối chứng, không gán |
| 028 | Pickup thật chở tượng ngựa | Pickup → `car`; tượng ngựa không gán |
| 029 | Xe đạp và xe máy chở thanh dài | Thanh dài không gán riêng, người đứng xa → `person` nếu đủ lớn |
| 030 | Bãi đỗ xe máy dày đặc | Gán hàng đầu, phần sau → vùng bỏ qua, `parked = yes` |
| 031 | Đường ướt đêm, đèn neon | `difficult`, phản chiếu không gán |
| 032 | Hàng người xếp chờ xe buýt | Vùng bỏ qua cho đám đông, xe con phía trước → `car` |
| 033 | Ma-nơ-canh, ảnh đen trắng | Ca đối chứng, không gán `person` |
| 034 | Người băng đường, xe máy và xe con | Gán người, xe máy, xe con |
| 035 | Xe ba bánh chở bao hàng cao | `three_wheeler`; hàng thuộc khung xe |
| 036 | Xe tải nhỏ thùng xanh | Kei-truck → `truck` hay `car` (xem mục 5) |
| 037 | Pickup có nắp thùng, xe buýt phía sau | Pickup → `car`; xe buýt bị che → `bus`, `occlusion` |
| 038 | Kệ xe mô hình và đồ chơi | Ca đối chứng, không gán |
| 039 | Phố nhỏ, xe máy ở xa | Phần lớn dưới ngưỡng kích thước |
| 040 | Taxi, tài xế đội mũ bảo hiểm, tuk-tuk phía sau | Người đội mũ gộp vào khung xe nếu ngồi trên xe; taxi → `car`; tuk-tuk → `three_wheeler` |

---

## 9. Quy trình kiểm soát chất lượng (QC) tóm tắt

1. Mỗi người gán nhãn độc lập trên cùng tập ảnh hiệu chuẩn, không trao đổi trong lúc gán.
2. So khớp bằng IoU ≥ 0,5 và cùng lớp; ghi lại các cặp lệch.
3. Họp thống nhất, ghi quyết định vào mục "Ca khó" và nâng phiên bản guideline (v1.1…).
4. Chỉ phát hành khi số liệu đồng thuận đạt ngưỡng của đề lab; nếu chưa đạt, quay lại bước 3.

---

## 10. Lịch sử phiên bản

| Phiên bản | Nội dung |
|-----------|----------|
| v1.0 | Bản đầu tiên, soạn từ việc xem cả 40 ảnh, chưa hiệu chuẩn |
