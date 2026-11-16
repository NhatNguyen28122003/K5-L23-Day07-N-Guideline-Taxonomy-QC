# Guideline gán nhãn — v0.2  — 273

- Phiên bản: v0.2-273 ( viết độc lập ở Pha 1)
- Ngày: 2026-11-16
- Người viết: 273 · Người duyệt: (nhóm, Pha 2)
> Quy ước đọc: **R-xx** là mã quy tắc để dẫn lại trong nhật ký QC.
> Cột "Loại ca" ở mục 4: **QT** = đã có quy tắc, gán thẳng; **RV** = tôi đã quyết tạm nhưng **cần reviewer/nhóm chốt**.

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người và phương tiện trên ảnh đường phố 2D**
(camera giao thông, hỗ trợ lái). Mô hình phải báo được vật **thật, có thể di chuyển hoặc va chạm**.

Sai nguy hiểm nhất, theo thứ tự:

1. **Bỏ sót người thật** (đặc biệt người đi xe máy, xe đạp, người bị che một phần) → mô hình không thấy người đi đường.
2. **Gán vật không thật thành vật thật** (tượng, ma-nơ-canh, ảnh in, phản chiếu, đồ chơi) → mô hình báo động giả, phanh gấp.
3. **Bỏ sót phương tiện** chỉ vì nó lạ (tuk-tuk, xích lô, xe lam) → mô hình không thấy chướng ngại. Vì vậy có lớp `other_vehicle`.
4. Nhầm lớp giữa các xe (`car` / `van` / `truck` / `bus`, `motorcycle` / `bicycle`) → ít nguy hiểm hơn nhưng làm giảm đồng thuận lớp (ngưỡng 0,95).

Vì vậy khi phân vân giữa "gán" và "không gán" một **người thật**, nghiêng về **gán + tick `can_review`**.

---

## 2. Lớp và thuộc tính

### 2.1 Lớp

9 lớp: 6 lớp của taxonomy v0 + 3 lớp mới `bus`, `bicycle`, `other_vehicle`. Mỗi box thuộc **đúng một** lớp.

| Lớp | Định nghĩa (một câu) | Ví dụ (ảnh) | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Xe ô tô chở người ≤ 9 chỗ: sedan, hatchback, wagon, SUV, MPV, taxi. | Mazda SUV `img_012`; taxi xanh `img_040`; sedan đen `img_023`; xe trưng bày `img_024`; xe trên giá chở `img_002` | Pickup → `truck` (R-03). Xe đồ chơi / mô hình `img_038` → không gán (R-01). |
| `truck` | Xe chở hàng có **cabin tách khỏi thùng hàng**: xe tải, pickup (kể cả pickup có nắp thùng), xe đầu kéo + rơ-moóc. | Pickup `img_001`, `img_012`, `img_028`, `img_037`; xe tải thùng `img_007`, `img_036`; xe chở ô tô `img_002` | Xe thùng kín liền khối (HiAce) → `van`. Tượng pickup `img_027` → không gán. |
| `van` | Xe thân **liền khối, một hộp** (cabin và khoang sau chung một vỏ), mũi ngắn, chở người/hàng, cỡ ≤ 16 chỗ. | Toyota HiAce `img_005` (biển 51LD-70.65); van trắng `img_016` | SUV/MPV có nắp capo dài → `car`. Pickup có nắp thùng `img_037` → `truck`. Xe khách 29 chỗ `img_009` → `bus`. |
| `bus` | Xe chở khách **≥ 17 chỗ**: thân dài một khối, một dãy cửa sổ hành khách dọc hai bên, có cửa lên xuống ở hông; gồm xe buýt, xe khách, xe khách cỡ trung (minibus coach). | Xe khách County `img_009`; xe buýt đêm `img_031`; xe buýt GO Transit `img_037`; xe buýt phía sau `img_040` | HiAce ≤ 16 chỗ `img_005` → `van`. Nhà chờ xe buýt `img_032` → không gán. Xe tải thùng kín không có cửa sổ hành khách → `truck`. |
| `motorcycle` | Xe **hai bánh có động cơ** (xe máy, xe tay ga, mô tô, xe điện kiểu tay ga), kể cả khi gắn thùng bên (sidecar) hoặc thùng hàng cố định. | Xe tay ga `img_010`, `img_026`; Vespa `img_016`; mô tô sidecar `img_022`; xe điện tay ga `img_029` | Xe đạp → `bicycle`. Xe ba bánh có mái (tuk-tuk, xe lam) → `other_vehicle`. |
| `bicycle` | Xe **hai bánh không động cơ**, có **khung xe đạp và bàn đạp** (kể cả xe đạp có giỏ, baga, chở hàng, xe đạp điện có bàn đạp và khung xe đạp). | Xe đạp đỗ `img_003`; dòng xe đạp `img_006`; người đạp xe chở ống `img_029`; xe đạp chở hàng của người bán `img_010` | Xe điện dạng tay ga (sàn để chân, yên dài, không đạp) → `motorcycle`. Xích lô 3 bánh `img_023` → `other_vehicle`. |
| `other_vehicle` | **Phương tiện giao thông thật, cỡ thật** chở người/hàng, chạy trên đường, **không thuộc 6 lớp xe trên**: tuk-tuk, xe lam / auto-rickshaw, xích lô, xe ba gác, máy kéo. | Tuk-tuk `img_004`, `img_016`, `img_040`; xe lam chở hàng `img_035`; xích lô `img_023` | Mô tô sidecar `img_022`, xe máy kéo thùng gắn liền `img_025` → `motorcycle`. Xe đẩy tay, gánh hàng, xe nôi → không gán. Xe đồ chơi `img_038`, tượng `img_027` → không gán. |
| `person` | **Người thật**, mọi tư thế: đi bộ, đứng, ngồi, lái hoặc ngồi trên bất kỳ xe nào. | Người đi bộ `img_014`, `img_034`; lái + người ngồi sau `img_026`; người ngồi quán `img_001`; người đạp xích lô `img_023` | Tượng `img_017`, ma-nơ-canh `img_033`, người trên biển quảng cáo `img_018`, người trên poster `img_030` → không gán (R-01). |
| `ignore_region` | **Vùng** (không phải vật) chứa nhiều vật thuộc các lớp trên mà không thể vẽ từng box; bỏ sót trong vùng này không tính lỗi. | Dòng xe máy dày phía xa `img_021`; đám đông sau rào `img_019`; bãi xe máy chồng lấp `img_030` | Không dùng để "đánh dấu vật không chắc" (dùng `can_review`), không dùng để che tượng / đồ chơi (những thứ đó chỉ đơn giản là không gán). |

**Cây chọn lớp phương tiện** (đi từ trên xuống, dừng ở câu "có" đầu tiên):

1. Có động cơ, **2 bánh** (kể cả khi gắn sidecar / thùng hàng cố định)? → `motorcycle`
2. Không động cơ, **2 bánh**, có bàn đạp và khung xe đạp? → `bicycle`
3. Xe 4 bánh trở lên chở khách **≥ 17 chỗ**? → `bus`
4. Cabin **tách** khỏi thùng hàng? → `truck`
5. Thân **một hộp**, mũi ngắn, ≤ 16 chỗ? → `van`
6. Ô tô chở người ≤ 9 chỗ có nắp capo? → `car`
7. Vẫn là phương tiện giao thông thật chở người/hàng? → `other_vehicle`
8. Còn lại (xe đẩy tay, xe nôi, động vật, tượng, đồ chơi) → **không gán**.

### 2.2 Thuộc tính

Mọi box (trừ `ignore_region`) có đủ 3 thuộc tính, **áp dụng giống nhau cho cả 3 lớp mới**.

**`occluded_level`** — tỉ lệ diện tích vật **bị vật khác che** (ước lượng bằng mắt, so với hình dạng đầy đủ của vật).
Phần nằm ngoài mép ảnh **không** tính vào đây (đã có `truncated`).

| Mức | Ngưỡng | Ví dụ |
|---|---|---|
| `0` | che < 10 % | pickup `img_028`; người đi bộ `img_034` |
| `1` | 10 % – < 35 % | xe máy bị chân người lái che `img_026`; xe sau hàng rào song thưa `img_018` |
| `2` | 35 % – < 70 % | người ngồi sau bị người lái che `img_005`; xe con tầng dưới giá chở `img_002`; người lái tuk-tuk sau khung xe `img_004` |
| `3` | ≥ 70 % (chỉ thấy một phần nhỏ nhưng **vẫn nhận ra lớp**) | người chỉ lộ đầu và vai trong dòng xe `img_021` |

Cách ước lượng nhanh: chia vật thành 4 phần (trên/dưới × trái/phải); mỗi phần bị che hết ≈ 25 %.
Không nhận ra lớp thì **không gán** (không dùng mức 3 cho "không rõ" như v0).

Quy tắc phụ:
- Hàng rào / lưới **thưa, nhìn xuyên qua được** vẫn thấy hình dạng vật → tối thiểu mức `1`.
- Người lái che xe, xe che chân người lái: **tính che cho cả hai** theo đúng tỉ lệ (áp dụng cho `motorcycle`, `bicycle`, `other_vehicle`).
- Khung xe, mái che, cột của chính chiếc xe che người ngồi bên trong (tuk-tuk, xe lam, xe buýt) → tính là che cho `person`.

**`truncated`**

| Giá trị | Khi nào |
|---|---|
| `yes` | Box chạm mép ảnh (cách ≤ 2 px) **và** một phần vật rõ ràng nằm ngoài khung. |
| `no` | Các trường hợp còn lại (kể cả vật chạm mép nhưng đã nằm trọn trong ảnh). |

**`can_review`** — tick khi **lớp** hoặc **việc có gán hay không** chưa chắc (ca RV ở mục 4). Không tick chỉ vì vẽ box khó.
Mỗi lần tick, ghi vào ghi chú cá nhân: ảnh, vật, lý do.

---

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| **R-01** Vật thật | Chỉ gán vật **thật, cỡ thật**. Không gán: tượng, tượng đài (`img_017`, `img_027`, ngựa `img_028`), ma-nơ-canh (`img_033`), người/xe in trên biển, poster, tranh tường (`img_018`, `img_030`, `img_026` tranh tường), **phản chiếu** trong gương, kính, mặt đường ướt (`img_013`, `img_015`, `img_020`, `img_031`), đồ chơi, mô hình (`img_038`). Xe **thật** đang trưng bày hoặc đỗ (`img_024`) vẫn gán. Xe trang trí kiểu giả tưởng nhưng **chạy được, có người lái** trên đường (`img_019`) vẫn gán theo hình dạng gần nhất. |
| **R-02** Box sát phần nhìn thấy | Box ôm sát **pixel nhìn thấy** của vật (modal), **không đoán** phần bị che. Vật bị che tách thành nhiều mảnh → **một** box bao các mảnh. Sai số mép ≤ 2 px với vật lớn. |
| **R-03** Pickup | Pickup (cabin + thùng hở hoặc có nắp thùng) luôn là `truck`, không phải `car`/`van`. |
| **R-04** Người + xe | Người trên xe hai bánh (`motorcycle`, `bicycle`) hoặc xe ba bánh (`other_vehicle`) = **box riêng cho từng người** (`person`) + **một box cho xe**. Box `person` gồm mũ, đồ mặc/đeo; box xe gồm thân, bánh, gương, tay lái, thùng/sidecar/ghế khách gắn cố định; hai box **được phép chồng nhau**. Người ngồi trên xe đang đỗ (`img_010`) cũng vậy. Ví dụ: `img_026` = 2 `person` + 1 `motorcycle`; `img_023` = 1 `person` + 1 `other_vehicle` (xích lô). |
| **R-05** Đồ mang theo, hàng chở | Không kéo box ra để bao **đồ cầm/gánh/chở nhô ra khỏi thân** người hoặc thân xe: đòn gánh và quang `img_003`, ống dài `img_029`, túi xách thõng `img_014`, hàng chất trên nóc xe lam `img_035`, mẹt hàng trên xe đạp `img_010`. Hàng nằm **trong** thùng xe thì đã nằm trong box xe. |
| **R-06** Xe chở xe | Xe đầu kéo + rơ-moóc nối liền = **một** box `truck` theo khung xe và giá chở, **không** kéo lên bao xe hàng ở trên. Mỗi xe con trên giá đủ điều kiện R-07 → box `car` riêng, tính che bởi khung giá (`img_002`). Ô tô thật đặt trên nóc xe khác (`img_024`) → hai box `car`. |
| **R-07** Vật nhỏ | Box có chiều cao **< 20 px** (ngưỡng `min_box_height_px`) → không vẽ. Box 20–50 px chỉ vẽ khi **nhận ra lớp**; nếu không, bỏ qua (hoặc gộp vào `ignore_region` theo R-08). |
| **R-08** Đám đông / dòng xe dày | Nếu ≥ 5 vật cùng lớp chồng nhau đến mức **không xác định được mép từng vật**, vẽ một `ignore_region` bao cả cụm. Vật **ở tiền cảnh**, tách được mép, vẫn vẽ box riêng **ngoài** vùng ignore. Không đặt box vật bên trong `ignore_region`. Ví dụ `img_019`, `img_021`, `img_030`, `img_036` (dãy xe máy đỗ phía sau). |
| **R-09** Ngoài taxonomy | Chỉ còn: xe đẩy tay, xe đẩy hàng không phải phương tiện giao thông, gánh hàng, xe nôi, động vật, vật không thật (R-01) → **không gán**, không dùng `ignore_region`. Người đi cùng các vật này → `person` nếu thấy đủ (R-07). |
| **R-10** Xe chủ (ego) | Phần xe của người chụp (gương, khung cửa, nắp capo, ghế xích lô) không gán (`img_020`; ảnh chụp từ xích lô `img_006`, `img_009`). |
| **R-11** Cắt mép | Vật bị cắt mép vẫn gán nếu phần trong ảnh **nhận ra lớp** và cao ≥ 20 px; đặt `truncated=yes`. Xe chỉ lộ **gương/tay lái** mà không thấy thân, bánh hoặc đầu xe → không gán (`img_040`, xe của người đội mũ đen). |
| **R-12** Nhòe chuyển động / tối | Vật nhòe hoặc trong ảnh đêm: vẽ khi **nhận ra lớp**; box theo phần đậm, rõ nhất của vệt nhòe, không bao bóng mờ. Nhòe đến mức không nhận ra lớp → không gán. (`img_014`, `img_015`, `img_013`, `img_031`.) |
| **R-13** Ảnh không có vật | Ảnh không có vật nào cần gán (`img_017`, `img_020`, `img_027`, `img_038`…) vẫn **phải có mặt** trong file COCO, với 0 annotation. |
| **R-14** Xe ba bánh và xe ghép | Tuk-tuk, xe lam, xích lô: **một** box `other_vehicle` bao cả đầu xe, thân, mái và ghế khách. Mô tô + sidecar, xe máy + thùng hàng gắn cố định: **một** box `motorcycle` (`img_022`, `img_025`). Rơ-moóc **rời**, kéo bằng dây/móc → không gộp vào box xe kéo. |
| **R-15** Ranh giới `bus` / `van` | Đếm cửa sổ hành khách một bên: **≥ 4 ô** hoặc thân dài ≥ 2 lần chiều cao, có cửa hông kiểu xe khách → `bus`; ít hơn → `van`. Không đếm được (bị che, góc chụp thẳng) → chọn theo kích thước so với ô tô con gần đó, tick `can_review`. |

Thứ tự tự kiểm khi gặp một vật: **Thật không? (R-01)** → **Là phương tiện / người không? (R-09)** → **Đủ lớn/nhận ra? (R-07, R-11, R-12)** → **Tách được không? (R-08)** → chọn lớp theo cây ở mục 2.1 (R-03, R-14, R-15) → vẽ box (R-02, R-04, R-05, R-06) → thuộc tính.

---

## 4. Ca khó

### 4.1 Ca đã có quy tắc (QT) và ca cần reviewer (RV)

| Ảnh | Tình huống | Rule ID | Quyết định của 273 | Loại ca | Lý do |
|---|---|---|---|---|---|
| img_001 | Pickup chở sắt, nhiều người ngồi quán phía sau | R-03, R-02 | 1 `truck`; mỗi người tách được mép → `person` (người bị bàn/ghế che ghi `occluded_level` theo tỉ lệ) | QT | Pickup luôn là truck; người thật ngồi vẫn là person. |
| img_002 | Xe đầu kéo chở 7–8 ô tô, 1 người đứng cạnh rơ-moóc | R-06, R-07 | 1 `truck` (cabin + rơ-moóc, không bao xe trên giá); mỗi xe con nhận ra được → `car` (tầng dưới thường `occluded_level` 2); 1 `person` | QT | Xe chở trên giá là ô tô thật, mô hình phải thấy; box truck không ăn vào chúng để IoU ổn định. |
| img_003 | Người bán hàng gánh quang; nhiều xe đạp, xe máy đỗ | R-05, R-04 | `person` không bao quang gánh; mỗi xe đạp tách được → `bicycle`; xe máy đỗ nhận ra → `motorcycle`; người dắt xe đạp → `person` | QT | Quang gánh nhô xa khỏi thân, đưa vào box làm IoU giữa hai người gán lệch lớn. |
| img_004 | Hai tuk-tuk có người lái | R-14, R-04 | Mỗi tuk-tuk → 1 `other_vehicle` (bao cả mái); người lái nhìn thấy → `person` (`occluded_level` 1–2 do khung xe) | QT | Tuk-tuk là phương tiện thật, chiếm làn; bỏ sót là nguy hiểm. |
| img_005 | Cận cảnh nhiều người đi xe máy, chồng nhau; van HiAce phía sau | R-04, R-11, R-15 | Mỗi người → `person` (người sau bị che 35–70 % → mức 2); mỗi xe máy nhận ra → `motorcycle` (nhiều xe chỉ lộ một phần → mức 2–3); HiAce → `van`; xe con trắng góc trái → `car`, `truncated=yes` | QT | Ảnh mẫu cho R-04: đếm người và xe riêng. |
| img_006 | Dòng xe nhìn từ sau (chụp từ xích lô): xe máy, xe đạp, taxi | R-04, R-10 | Xe máy + người → R-04; xe đạp + người đạp → `bicycle` + `person`; taxi trắng → `car` (`truncated=yes`); phần xích lô của người chụp (nếu lọt khung) không gán | QT | |
| img_007 | Kho đêm: xe tải Isuzu, pickup có khung giá nóc | R-03 | Isuzu → `truck`; Hilux có khung → `truck` | QT | Khung giá trên thùng không đổi bản chất pickup. |
| img_008 | Xe ôm áo cam, bãi xe máy đỗ dày | R-04, R-08 | Xe ôm tiền cảnh → `person` + `motorcycle`; xe đỗ tiền cảnh tách được → `motorcycle`; cụm xe/người xa phía sau không tách được → `ignore_region` | QT | |
| img_009 | Xe khách ~29 chỗ (Hyundai County); xe máy xa | R-15, R-07 | Xe khách → `bus`; xe máy xa nhỏ: chỉ gán nếu cao ≥ 20 px | **RV** | Dễ có người gán thành `van`. Theo R-15: ≥ 4 ô cửa sổ khách + cửa hông → `bus`. Cần nhóm xác nhận ngưỡng. |
| img_010 | Người ngồi trên xe máy đang đỗ; người bán hàng với xe đạp chở mẹt hàng | R-04, R-05 | Người ngồi → `person` + xe → `motorcycle`; xe đạp chở hàng → `bicycle` (không bao mẹt hàng nhô ra); người bán → `person` | **RV** | Xe đạp bị hàng che nhiều, có thể có người không nhận ra là xe đạp. |
| img_011 | Tượng / người hóa trang Predator; dãy mô tô làm từ phế liệu | R-01 | Predator: không gán; mô tô phế liệu: không gán; người thật ở xa (≥ 20 px) → `person` | **RV** | Không chắc Predator là tượng hay người mặc đồ; mô tô có thể là xe độ chạy được. Cần reviewer nhìn bản gốc. |
| img_012 | SUV, pickup chở hàng, xe tải phía xa | R-03, R-05 | SUV → `car`; pickup → `truck` (box không kéo lên bao túi hàng nhô khỏi thành thùng); xe tải xa → `truck` | QT | |
| img_013 | Đường ướt ban đêm, phản chiếu đèn, người và xe nhỏ | R-01, R-07, R-12 | Không gán phản chiếu trên mặt đường; xe/người xa chỉ gán khi ≥ 20 px và nhận ra | QT | |
| img_014 | 2 người đi bộ rõ nét + người và xe máy nhòe chuyển động | R-12, R-05 | 2 `person` (box không gồm túi xách thõng); người/xe nhòe: gán khi còn nhận ra lớp, box phần đậm nhất | **RV** | Hai người gán dễ lệch mép box vùng nhòe → nên có ví dụ box mẫu. |
| img_015 | Xe đỏ, xe bạc cận cảnh cắt mép; xe nhòe phía sau; mặt đường phản chiếu | R-11, R-12, R-01 | Xe đỏ, xe bạc → `car`, `truncated=yes`; xe nhòe → `car`; phản chiếu không gán | QT | |
| img_016 | Tuk-tuk xanh lá, tuk-tuk xanh dương, Vespa có người, van, pickup, phụ nữ cắt mép | R-14, R-03, R-04, R-11 | 2 tuk-tuk → `other_vehicle` (tuk-tuk xanh lá cắt mép phải → `truncated=yes`); người trong → `person`; Vespa → R-04; van → `van`; pickup → `truck`; phụ nữ → `person`, `truncated=yes` | QT | |
| img_017 | Tượng người cưỡi ngựa đội nón giao thông; chim | R-01, R-13 | Không gán gì; ảnh vẫn có trong COCO | QT | Ca đối chứng vật không thật. |
| img_018 | Biển quảng cáo có người; xe đỗ sau hàng rào | R-01, R-02 | Người trên biển không gán; từng xe nhận ra → `car`, `occluded_level` ≥ 1 vì rào song thưa | QT | |
| img_019 | Xe Dangermouse giả tưởng có người lái; nhân vật chuột; đám đông sau rào | R-01, R-08 | Xe → `car`; người lái → `person`; nhân vật chuột không gán; người tiền cảnh tách được → `person`; đám đông phía sau → `ignore_region` | **RV** | Xe chạy được trên đường nên coi là vật thật. Có thể chọn `other_vehicle` vì hình dạng lạ — cần chốt. Nhân vật chuột có thể là người mặc đồ. |
| img_020 | Gương chiếu hậu phản chiếu xe phía sau | R-01, R-10, R-13 | Không gán (phản chiếu + xe chủ) | QT | |
| img_021 | Dòng xe máy rất dày giờ cao điểm | R-04, R-08 | ~8–12 cặp người + xe tiền cảnh vẽ riêng; khối xe phía xa → một `ignore_region`; xe con trắng bên trái → `car` | QT | Vẽ từng người ở phía xa không khả thi và làm IAA vô nghĩa. |
| img_022 | Ảnh đen trắng cũ, mô tô có sidecar | R-14, R-04 | 1 `motorcycle` (bao cả sidecar); người trên xe nếu thấy → `person` | QT | Sidecar gắn cố định → không phải `other_vehicle`. |
| img_023 | Ô tô đen, xích lô có người đạp, xe máy xa | R-14, R-04 | Ô tô → `car`; xích lô → `other_vehicle`; người đạp → `person`; xe máy + người xa → R-04 nếu ≥ 20 px | QT | Xích lô 3 bánh, có ghế khách → không phải `bicycle`. |
| img_024 | Bảo tàng xe; ô tô vàng đặt trên nóc xe wagon đen | R-01, R-06, R-11 | Mọi xe thật nhận ra → `car`; xe vàng trên nóc → `car` riêng; xe cam góc dưới → `car`, `truncated=yes` | **RV** | Cần xác nhận xe vàng là xe thật cỡ thật, không phải mô hình. |
| img_025 | Phố biển: người đi bộ, taxi vàng, xe máy kéo thùng hàng | R-14, R-07 | Người ≥ 20 px → `person`; taxi → `car`; xe máy + thùng gắn liền → 1 `motorcycle`, người lái → `person` | **RV** | Thùng phía sau là gắn cố định (→ `motorcycle`) hay xe ba bánh kiểu sa-leng (→ `other_vehicle`)? Tôi coi là gắn cố định. |
| img_026 | Đôi nam nữ trên xe máy trong hầm, tranh tường phía sau | R-04, R-01 | 2 `person` + 1 `motorcycle`; tranh tường không gán | QT | Ảnh mẫu R-04. |
| img_027 | Tượng đài hình pickup ở nghĩa trang | R-01, R-13 | Không gán | QT | Ca đối chứng. |
| img_028 | Pickup cổ thật, tượng ngựa đứng trong thùng; SUV đỏ phía sau | R-01, R-03 | Pickup → `truck` (box theo xe, không kéo lên bao ngựa); ngựa không gán; SUV đỏ → `car` (bị pickup che → mức theo tỉ lệ) | QT | Xe thật, ngựa là tượng. |
| img_029 | Người chở ống dài trên xe điện; người đi xe đạp | R-04, R-05 | Xe điện tay ga → `motorcycle` (không bao ống); người lái → `person`; xe đạp → `bicycle` + người đạp → `person`; người đi bộ, người đạp xe xa ≥ 20 px → gán tương ứng | QT | Xe điện có sàn để chân, không đạp → `motorcycle` theo cây chọn lớp. |
| img_030 | Bãi xe máy đỗ dày, poster có mặt người | R-08, R-01 | Xe tiền cảnh tách được → `motorcycle`; khối xe phía sau → `ignore_region`; người mặc áo xanh → `person`; poster không gán | QT | |
| img_031 | Phố đêm mưa: SUV, xe con, xe buýt, xe máy | R-12, R-01 | Xe con → `car`; xe buýt → `bus`; xe máy + người nhận ra → R-04; phản chiếu không gán | QT | |
| img_032 | Đám đông chờ xe buýt; xe con cắt mép; nón giao thông | R-08, R-11 | Người đứng hàng ngang tách được mép → `person` riêng; đoạn chồng lấp ≥ 5 người không tách được → `ignore_region`; xe xanh → `car`, `truncated=yes`; nhà chờ không gán | **RV** | Ranh giới "tách được mép" dễ lệch giữa hai người gán ở đây. |
| img_033 | Ma-nơ-canh, người trong ảnh in khổ lớn, cửa xe phản chiếu trên kính | R-01 | Không gán gì | **RV** | Phần cửa xe bên phải có thể là xe thật đỗ sát kính; tôi nghiêng về phản chiếu. |
| img_034 | Người đi bộ băng qua đường, ô tô, xe máy và xe đạp xa | R-04, R-07, R-11 | Người → `person`; ô tô → `car`; xe máy/xe đạp + người ≥ 20 px → R-04; người đi xe máy cắt mép phải → `truncated=yes` | QT | |
| img_035 | 2 xe lam (một chở hàng chất cao), mô tô có người, ô tô | R-14, R-05, R-04 | Mỗi xe lam → `other_vehicle` (không bao hàng chất trên nóc); người lái xe lam nhìn thấy → `person`; mô tô + người → R-04; ô tô trắng → `car` | QT | |
| img_036 | Xe tải nhỏ phủ bạt; dãy xe máy đỗ phía sau | R-08, R-11 | Xe tải → `truck`; xe máy đỗ tách được → `motorcycle`; dãy dày → `ignore_region` | QT | Thùng phủ bạt vẫn là cabin tách thùng → truck. |
| img_037 | Pickup có nắp thùng kín; 2 xe buýt; người đi bộ | R-03, R-15 | Pickup → `truck`; mỗi xe buýt → `bus` (xe bên phải cắt mép/bị che → thuộc tính theo tỉ lệ); người → `person` | QT | Pickup dễ nhầm `van` vì thùng kín. |
| img_038 | Tủ kính đồ chơi, mô hình ô tô | R-01, R-13 | Không gán, không `ignore_region` | QT | Để mô hình học đây là nền. |
| img_039 | Phố nhỏ: xe máy đỗ, người đi bộ, xe máy và xe đạp chạy xa | R-04, R-07 | Gán theo R-04 (`motorcycle` / `bicycle` + `person`); vật < 20 px bỏ | QT | |
| img_040 | Người đội mũ bảo hiểm cận cảnh, chỉ thấy gương xe; taxi; tuk-tuk; SUV; xe buýt | R-11, R-14 | Người → `person`, `truncated=yes`; xe của người đó **không gán** (chỉ thấy gương); taxi → `car`, `truncated=yes`; tuk-tuk → `other_vehicle`; SUV → `car`; xe buýt → `bus` (bị che → mức theo tỉ lệ) | **RV** | Có người sẽ gán `motorcycle` vì biết chắc đó là xe máy — cần nhóm chốt R-11. |

### 4.2 Câu hỏi cho reviewer / họp Pha 2

1. Ngưỡng `bus` / `van` ở R-15 (≥ 4 ô cửa sổ khách, ≥ 17 chỗ) có ổn cho `img_009` không?
2. Xe giả tưởng `img_019` là `car` hay `other_vehicle`? Xe máy kéo thùng `img_025` là `motorcycle` hay `other_vehicle`?
3. Người mặc đồ hóa trang (`img_011`, `img_019`) là `person` hay không gán?
4. Xe chỉ lộ gương/tay lái (`img_040`): giữ R-11 (không gán) hay gán `motorcycle`, `occluded_level=3`?
5. Ngưỡng "≥ 5 vật chồng nhau" ở R-08 có hợp lý cho `img_032` không, hay đổi thành tiêu chí theo chiều cao box?
6. `other_vehicle` gom nhiều loại xe khác hình dạng (tuk-tuk, xích lô, xe lam): có cần tách nhỏ hơn ở phiên bản sau không, nếu IAA lớp này thấp?

---

## 5. QC và phiên bản

### 5.1 Ngưỡng áp dụng (lấy nguyên từ `taxonomy.json` mục `rules`, không sửa)

| Ngưỡng | Giá trị | Dùng ở đâu |
|---|---|---|
| `min_box_height_px` | 20 | R-07, R-11 |
| `large_px` | 50 | phân biệt vật lớn / nhỏ |
| `iou_thr_large` / `iou_thr_small` | 0,85 / 0,70 | box đúng khi IoU với box tham chiếu đạt ngưỡng |
| `match_iou` | 0,5 | ghép cặp box giữa hai người gán |
| `iaa_iou` / `iaa_class` | 0,85 / 0,95 | đồng thuận vị trí / lớp trên các cặp ghép được |
| `error_rate_img_max` | 2 % | quá ngưỡng → Rework |

Thêm lớp làm `taxonomy.json` phải có đủ 9 lớp (`bus`, `bicycle`, `other_vehicle` kèm định nghĩa một câu như bảng 2.1),
**giữ nguyên mục `rules`**, rồi chạy lại `python scripts/make_cvat_labels.py` và tạo lại Project CVAT.

### 5.2 Ai quyết ca mơ hồ

- Lúc gán (Pha 3): **không hỏi nhau**; tick `can_review` hoặc ghi vấn đề cấp ảnh (`image,annotator,region,note`).
- Pha 5: nhóm chốt từng ca trong `calibration_decisions.csv`, ghi rule ID; người kiểm lại **khác** người sửa.
- Quy tắc đổi cách chọn vật / lớp / box → `semantic`, tăng phiên bản và kiểm lại cả lô (`pham_vi=lo`).
  Chỉ sửa câu chữ → `editorial`.

### 5.3 Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v0.1-273 | Định nghĩa lại 5 lớp kèm ví dụ/phản ví dụ; ngưỡng % cho `occluded_level` (bỏ mức "không rõ"); định nghĩa `truncated`; thêm R-01 … R-13; bảng 40 ảnh | Pickup/van (`img_037`), người + xe máy (`img_026`, `img_005`), xe chở xe (`img_002`), vật không thật (`img_017`, `img_027`, `img_033`, `img_038`), phản chiếu (`img_020`), đám đông (`img_021`, `img_032`), ngoài taxonomy (`img_004`, `img_035`) |
| v0.2-273 | Thêm lớp `bus`, `bicycle`, `other_vehicle`; cây chọn lớp phương tiện; mở rộng R-04 cho xe đạp và xe ba bánh; thu hẹp R-09; thêm R-14 (xe ba bánh / xe ghép), R-15 (ranh giới bus/van); cập nhật bảng ca khó | Xe buýt / xe khách xuất hiện ở `img_009`, `img_031`, `img_037`, `img_040`; xe đạp ở `img_003`, `img_006`, `img_010`, `img_029`; tuk-tuk, xe lam, xích lô ở `img_004`, `img_016`, `img_023`, `img_035`, `img_040` — bỏ không gán thì mô hình học chúng là nền |
