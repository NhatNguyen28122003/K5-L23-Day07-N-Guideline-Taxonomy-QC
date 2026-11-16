# Guideline gán nhãn — v1.1 (sau calibration) — nhóm 066 · 263 · 273 · 277 · 291

- Phiên bản: **v1.1** (hiệu chỉnh từ v1.0 sau IAA Pha 4)
- Ngày: 2026-11-16
- Người viết: nhóm · Người duyệt: cả nhóm (Pha 5)
- Taxonomy đi kèm: `submission/taxonomy.json` `version = v1.0`. **Không đổi lớp, thuộc tính hay ngưỡng**, nên không có `taxonomy_v1.1.json`.
- Quy ước: **R-xx** là mã quy tắc, dùng lại trong `calibration_decisions.csv` và phiếu QC. Chỗ đổi so với v1.0 đánh dấu **[v1.1]**.
- Căn cứ: `submission/qc/iaa_summary.md` (40 ảnh, 5 người, 1137 cặp box ghép). IoU TB 0,838 (< 0,85), cùng lớp 96,5 %, cùng `occluded_level` 47,7 %, cùng `truncated` 87,1 %; 3678 ca, gồm 2022 bỏ sót/thừa, 663 sai thuộc tính, 442 IoU thấp, 412 bất đồng ignore, 99 `can_review`, 40 sai lớp.

---

## 1. Mục tiêu

Nhãn bounding box dùng để huấn luyện và đánh giá mô hình **phát hiện người và phương tiện thật trên ảnh đường phố 2D** (camera giao thông, hỗ trợ lái). Mô hình phải báo được vật **thật, có thể di chuyển hoặc va chạm**.

Sai nguy hiểm nhất, theo thứ tự:

1. **Bỏ sót người thật**, nhất là người đi xe hoặc bị che một phần: mô hình không thấy người đi đường.
2. **Gán vật không thật thành vật thật** (tượng, ma-nơ-canh, ảnh in, phản chiếu, đồ chơi): mô hình báo động giả.
3. **Nhầm lớp giữa các xe** (`car` / `truck` / `van` / `bus`): ít nguy hiểm hơn nhưng kéo đồng thuận lớp xuống dưới 0,95.

Khi phân vân có gán một **người thật** hay không: **gán và tick `can_review`**.

---

## 2. Lớp và thuộc tính

### 2.1 Lớp

Chín lớp, mỗi box thuộc **đúng một** lớp. So với v0, nhóm thêm `bus`, `bicycle`, `other_vehicle`, vì 4/5 bản cá nhân gặp xe buýt, xe đạp, tuk-tuk ở ít nhất 4 ảnh và không có lớp nào để gán.

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người tối đa 9 chỗ, có mũi xe tách khoang khách: sedan, hatchback, wagon, SUV, MPV, taxi. | Sedan đỏ `img_002`; SUV `img_012`; taxi `img_040`; xe trưng bày thật `img_024` | Pickup → `truck` (R-03). Mô hình, đồ chơi `img_038` → không gán (R-01). |
| `truck` | Xe chở hàng có **cabin tách khỏi thùng hàng**: xe tải, pickup (kể cả có nắp thùng hoặc khung giá nóc), xe đầu kéo, xe chở ô tô. | Pickup `img_001`, `img_028`, `img_037`; xe tải `img_007`, `img_036`; xe chở ô tô `img_002` | Xe thân liền một hộp → `van`. Tượng pickup `img_027` → không gán. |
| `van` | Xe thân **liền một hộp** (cabin và khoang sau chung vỏ), mũi ngắn, tối đa 16 chỗ, chở người hoặc hàng. | Toyota HiAce `img_005`; van trắng `img_016` | SUV/MPV có capo dài → `car`. Pickup có nắp thùng → `truck`. Xe khách từ 17 chỗ → `bus`. |
| `bus` | Xe buýt, xe khách, minibus **từ 17 chỗ** (thường có 2 trục, cửa giữa hoặc nhiều hàng cửa sổ). | Xe khách ~29 chỗ `img_009`; xe buýt `img_031`, `img_037`, `img_040` | Van 16 chỗ → `van`. |
| `motorcycle` | Xe **hai bánh có động cơ**: xe máy, tay ga, mô tô, xe máy điện; kể cả sidecar hoặc thùng hàng **gắn cố định**. | Tay ga `img_010`, `img_026`; mô tô sidecar `img_022`; xe máy điện `img_029` | Xe đạp → `bicycle`. Tuk-tuk, xe lam → `other_vehicle`. |
| `bicycle` | Xe đạp **hai bánh** chạy bằng sức người, kể cả xe đạp trợ lực còn bàn đạp. | Xe đạp `img_003`, `img_006`, `img_029`, `img_039` | Xích lô, xe ba gác đạp → `other_vehicle`. Xe máy điện có yên xe máy → `motorcycle`. |
| `other_vehicle` | Phương tiện có bánh **thật** không thuộc các lớp trên: tuk-tuk, xe lam, auto-rickshaw, xích lô, xe ba gác, xe ba bánh chở hàng. | Tuk-tuk `img_004`, `img_016`, `img_035`, `img_040`; xích lô `img_023` | Xe đẩy hàng, xe đẩy em bé → không gán (không phải phương tiện giao thông). |
| `person` | **Người thật** nhìn thấy trực tiếp, mọi tư thế: đi bộ, đứng, ngồi, người lái và hành khách trên **mọi** phương tiện. | Người đi bộ `img_014`, `img_034`; 2 người trên xe máy `img_026`; người ngồi quán `img_001` | Tượng `img_017`, ma-nơ-canh `img_033`, người trên biển quảng cáo `img_018`, poster `img_030` → không gán. |
| `ignore_region` | **Vùng** (không phải vật) chứa **từ 5 vật** thuộc lớp mục tiêu chồng lấn đến mức không xác định được mép từng vật; bỏ sót trong vùng này không tính lỗi. | Dòng xe máy xa `img_021`; đám đông sau rào `img_019`; bãi xe máy `img_030` | Không dùng cho vật không chắc (dùng `can_review`). Không dùng để che tượng, đồ chơi, phản chiếu (chỉ cần không gán). |

**Vật không thuộc lớp nào** (động vật, xe đẩy hàng, nón giao thông, biển báo, tượng, đồ chơi): **không vẽ box, không dùng `ignore_region`**. Người thật đi cùng các vật này vẫn gán `person`.

### 2.2 Thuộc tính

Mọi box (trừ `ignore_region`) có đủ 3 thuộc tính. Cờ `Occluded` có sẵn của CVAT bị script bỏ qua, nên chỉ dùng `occluded_level`.

**`occluded_level`**: tỉ lệ diện tích vật **bị vật khác che**, so với hình dạng đầy đủ đáng lẽ nằm trong ảnh. Phần nằm ngoài mép ảnh **không** tính ở đây mà ghi bằng `truncated`.

| Mức | Ngưỡng | Ví dụ |
|---|---|---|
| `0` | che < 10 % | pickup `img_028`; người đi bộ `img_034` |
| `1` | 10 % đến < 35 % | xe máy bị chân người lái che `img_026`; xe sau hàng rào thưa `img_018` |
| `2` | 35 % đến < 70 % | người ngồi sau bị người lái che `img_005`; ô tô tầng dưới giá chở `img_002` |
| `3` | ≥ 70 %, chỉ thấy một phần nhỏ nhưng **vẫn nhận ra lớp** | người chỉ lộ đầu và vai trong dòng xe `img_021` |

- Cách ước lượng nhanh: chia vật thành 4 phần (trên/dưới × trái/phải); mỗi phần bị che hết ≈ 25 %.
- Không nhận ra lớp thì **không gán**. Không dùng mức `3` cho nghĩa "không rõ".
- Hàng rào hoặc lưới **thưa, nhìn xuyên được**: tối thiểu mức `1`.
- Người lái che xe máy và xe máy che chân người lái: tính che **cho cả hai box**.
- Nhòe chuyển động **không** phải che khuất (R-12).

**[v1.1] R-14: cách chọn mức khi phân vân.** IAA chỉ cùng `occluded_level` ở 47,7 % cặp box (263|291: 29,6 %, 277|291: 28,5 %), nhiều nhất ở người và xe máy trong dòng xe (`img_005`, `img_021`, `img_034`, `img_039`). Từ v1.1:

1. **Chỉ tính vật khác che** (người, xe, cột, rào, hàng hóa của xe khác). Mép ảnh, bóng tối, nhòe, chính bộ phận của vật (tay người che mặt mình) **không** tính.
2. Ước lượng theo **4 phần** như trên, rồi làm tròn **xuống** mức gần nhất. Nằm sát ranh giới 10 / 35 / 70 % thì chọn **mức thấp hơn**.
3. Quy ước sẵn cho ca hay gặp (chỉ áp dụng khi không có vật che khác):

| Ca | `person` | `motorcycle` / `bicycle` |
|---|---|---|
| Người lái xe máy, nhìn ngang | `1` (chân bị thân xe che) | `1` (yên, thân bị người che) |
| Người lái xe máy, nhìn thẳng trước / sau | `1` | `2` (thân xe gần như sau người) |
| Người ngồi sau, bị người lái che nửa thân | `2` | không đổi mức xe |
| Người sau bàn, quầy che từ hông trở xuống | `2` | — |
| Xe sau hàng rào song thưa | — | `1` |

4. Vẫn phân vân sau 3 bước → chọn mức thấp hơn **và không** tick `can_review` (`can_review` chỉ dành cho lớp và việc có gán hay không).

**`truncated`**

| Giá trị | Khi nào |
|---|---|
| `yes` | Box chạm mép ảnh (cách ≤ 2 px) **và** một phần vật rõ ràng nằm ngoài ảnh. |
| `no` | Các trường hợp còn lại, kể cả vật chạm mép nhưng nằm trọn trong ảnh. |

Một vật có thể vừa `truncated=yes` vừa `occluded_level > 0`.

**[v1.1]** Cùng `truncated` chỉ đạt 87,1 % (263|291: 79,6 %). Từ v1.1: **box chạm mép (≤ 2 px) thì mặc định `truncated=yes`**. Chỉ chọn `no` khi nhìn rõ đường viền ngoài của vật (lốp xe, gót chân, nóc xe) nằm trong ảnh. Không cần chắc phần ngoài ảnh "rõ ràng", vì hai người gán không cùng thấy được phần không có trong ảnh.

**`can_review`**: tick khi **lớp** hoặc **việc có gán hay không** còn chưa chắc. Không tick chỉ vì box khó vẽ. Vật không vẽ được box thì ghi vào `submission/qc/image_issues.csv` (`image,annotator,region,note`) sau khi khóa nhãn.

---

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| **R-01** Vật thật | Chỉ gán vật **thật, cỡ thật**. Không gán: tượng, tượng đài (`img_017`, `img_027`, ngựa `img_028`), ma-nơ-canh (`img_033`), người/xe trên biển, poster, tranh tường (`img_018`, `img_026`, `img_030`), **phản chiếu** trong gương, kính, mặt đường ướt (`img_013`, `img_015`, `img_020`, `img_031`), đồ chơi, mô hình (`img_038`). Xe **thật** đang trưng bày hoặc đỗ (`img_024`) vẫn gán. Xe trang trí lạ nhưng **chạy được, có người lái** (`img_019`) gán theo kết cấu gần nhất và tick `can_review`. Người thật mặc đồ hóa trang là `person`; không chắc là người hay tượng thì gán `person` + `can_review`. **[v1.1]** IAA cho thấy 2 người vẫn gán vật mô phỏng: 36 `car` trên tủ mô hình `img_038`, `person` trên tượng `img_017` và ma-nơ-canh `img_033`, `truck` trên tượng `img_027`, `car` phản chiếu trong gương `img_020`. Từ v1.1, 5 ảnh **đối chứng** này phải có **0 annotation**, và reviewer kiểm riêng chúng trước khi chốt nhãn (mục 5.2). |
| **R-02** Box sát phần nhìn thấy | Box là hình chữ nhật nhỏ nhất ôm **pixel nhìn thấy** của vật, **không đoán** phần bị che. Vật bị che tách thành nhiều mảnh vẫn dùng **một** box bao các mảnh. Sai số mép ≤ 2 px với vật lớn. Không bao bóng đổ. |
| **R-03** Phân lớp xe theo kết cấu | Chọn lớp theo **kết cấu thân xe**, không theo màu, kích thước hay mục đích. Pickup (cabin + thùng hở hoặc có nắp) luôn là `truck`. Thân liền một hộp ≤ 16 chỗ → `van`; ≥ 17 chỗ → `bus`. Ba bánh hoặc xe chở người bằng sức người không phải xe đạp hai bánh → `other_vehicle`. Không nhìn rõ kết cấu → lớp gần nhất + `can_review`. **[v1.1]** Có 40 ca sai lớp, gồm `bus`/`van` ở `img_005`, `car`/`van` ở `img_016`, `img_011`, `car`/`truck` ở `img_007`, `img_024`, `bus`/`truck` ở `img_031`, `img_037`. Xem bảng phân lớp xe ở mục 4.2. |
| **R-04** Người + xe | Người trên xe = **một box `person` cho từng người** + **một box riêng cho xe** (`motorcycle`, `bicycle`, `other_vehicle`…). Hai box được chồng nhau. Box `person` gồm mũ, quần áo, đồ đeo; box `motorcycle` gồm thân, bánh, gương, tay lái, sidecar/thùng gắn cố định. Người ngồi trên xe đang đỗ (`img_010`) cũng vậy. Ví dụ `img_026` = 2 `person` + 1 `motorcycle`. |
| **R-05** Đồ mang theo, hàng chở | Không kéo box ra để bao đồ cầm, gánh, chở **nhô ra khỏi thân** người hoặc xe: quang gánh `img_003`, ống dài `img_029`, túi xách thõng `img_014`, hàng chất trên nóc `img_035`. Hàng nằm **trong** thùng xe đã nằm trong box xe. |
| **R-06** Xe chở xe | Xe đầu kéo + rơ-moóc nối liền = **một** box `truck` theo khung xe và giá chở, không kéo lên bao xe ở trên. Mỗi xe trên giá đủ điều kiện R-07 → box riêng theo lớp của nó, tính che bởi khung giá (`img_002`). Ô tô thật đặt trên nóc xe khác (`img_024`) → hai box. |
| **R-07** Vật nhỏ | Box có chiều cao phần nhìn thấy **< 20 px** trên ảnh gốc (`min_box_height_px`) → không vẽ. Box 20–49 px chỉ vẽ khi **nhận ra lớp**; nếu không thì bỏ, hoặc gộp vào `ignore_region` theo R-08. Đo trên ảnh gốc, không theo mức phóng của CVAT. |
| **R-08** Đám đông / dòng xe dày | Nếu **từ 5 vật** chồng nhau đến mức không xác định được mép từng vật, vẽ **một** `ignore_region` bao cả cụm. Vật tiền cảnh tách được mép vẫn vẽ box riêng **ngoài** vùng ignore; thu vùng ignore cho khỏi che vật rõ. Không đặt box vật bên trong `ignore_region`. Ví dụ `img_019`, `img_021`, `img_030`, `img_032`, `img_036`. **[v1.1]** Có 412 ca vùng ignore không khớp nhau (`img_032`: 98, `img_036`: 37, `img_008`: 30). Từ v1.1: (a) **không** dùng ignore cho cụm mà mỗi vật đều cao **≥ 50 px**, vì vật đó phải vẽ riêng; (b) vùng ignore ôm sát cụm, cạnh dưới đặt ở **chân vật gần nhất trong cụm**, không kéo xuống mặt đường hay lên trời; (c) mỗi cụm liền nhau dùng **một** vùng, không chia nhỏ; (d) ảnh không có cụm ≥ 5 vật chồng lấn thì **không có** `ignore_region`. |
| **R-09** Một vật, một box | Mỗi vật độc lập có box riêng, kể cả khi đứng sát cùng loại. Không gộp hai xe thành một box, không vẽ box cho bộ phận (chỉ bánh xe), không vẽ hai box cho cùng một vật. |
| **R-10** Xe chủ (ego) | Phần xe của người chụp (gương, khung cửa, nắp capo) không gán (`img_020`). |
| **R-11** Cắt mép | Vật bị cắt mép vẫn gán nếu phần trong ảnh **nhận ra lớp** và cao ≥ 20 px; đặt `truncated=yes`. Xe chỉ lộ **gương hoặc tay lái**, không thấy thân, bánh hoặc đầu xe → không gán (`img_040`). |
| **R-12** Nhòe / tối / ảnh cũ | Vật nhòe chuyển động, ảnh đêm hoặc ảnh đen trắng: vẽ khi **nhận ra lớp**; box theo phần đậm, rõ nhất của vệt nhòe, không bao bóng mờ. Nhòe đến mức không nhận ra lớp → không gán. Không dùng `occluded_level` để biểu diễn độ nhòe. (`img_013`, `img_014`, `img_015`, `img_022`, `img_031`) |
| **R-13** Ảnh không có vật | Ảnh không có vật cần gán (`img_017`, `img_020`, `img_027`, `img_038`) vẫn **phải có mặt** trong COCO với 0 annotation. |
| **R-15 [v1.1]** Box người | Box `person` đi từ **đỉnh đầu hoặc đỉnh mũ bảo hiểm** đến **điểm thấp nhất nhìn thấy của cơ thể** (bàn chân, hoặc chân đặt trên bàn để chân), rộng đến hai tay hoặc vai, **không** gồm tay lái, thân xe, túi xách thõng. Người ngồi sau bị người lái che vẫn có box riêng theo phần nhìn thấy (R-02). Lý do: 260/442 ca IoU thấp là `person`. |
| **R-16 [v1.1]** Ưu tiên gán người | Người thật cao **≥ 20 px** nhận ra được (thấy đầu và thân) **phải** có box riêng, kể cả ở trung cảnh, trên vỉa hè hay trong quán, trừ khi nằm trong `ignore_region` hợp lệ theo R-08. Lý do: `person` chiếm 844/2022 ca bỏ sót/thừa, dồn ở `img_034`, `img_039`, `img_032`, `img_021`, `img_025`. |
| **R-17 [v1.1]** Xe nhìn thẳng sau người lái | Xe máy hoặc xe đạp nhìn thẳng trước/sau, gần như bị người lái che hết: vẫn vẽ box xe nếu **thấy bánh xe hoặc đầu xe** cao ≥ 20 px; box xe chỉ bao phần xe nhìn thấy, không bao người. Không vẽ hai box gần trùng nhau rồi đổi lớp: nếu box `person` và box xe có IoU > 0,8 thì xem lại cả hai. Lý do: 11/40 ca sai lớp là `motorcycle`/`person` (`img_039`, `img_034`, `img_009`). |

**Thứ tự tự kiểm khi gặp một vật:** Thật không? (R-01) → Lớp nào? (R-03) → Đủ lớn, nhận ra được? (R-07, R-11, R-12, R-16) → Tách được khỏi cụm? (R-08) → Vẽ box (R-02, R-04, R-05, R-06, R-09, R-15, R-17) → Thuộc tính (mục 2.2, R-14).

---

## 4. Ca khó

### 4.1 Quyết định nhóm cho các điểm 5 bản cá nhân khác nhau

| Điểm bất đồng | Các phương án cá nhân | Quyết định v1.0 | Rule ID | Lý do |
|---|---|---|---|---|
| Xe buýt, xe đạp, tuk-tuk, xích lô | 066: `ignore_region`; 263, 273, 291: không gán; 277: thêm lớp `bus`, `bicycle`, `three_wheeler` | Thêm 3 lớp `bus`, `bicycle`, `other_vehicle` | R-03 | Xuất hiện ở ≥ 4 ảnh; bỏ trống hoặc ép lớp làm mô hình học sai nền; `ignore_region` không phải để đánh dấu lớp còn thiếu. |
| Pickup | 263, 273, 291, 066: `truck`; 277: `car` | `truck` | R-03 | Cabin tách thùng là tiêu chí kết cấu dễ kiểm hơn kích thước. |
| Người trên xe | 263, 273, 291, 066: box riêng; 277: gộp vào box xe | Box `person` riêng cho từng người + box xe | R-04 | Ưu tiên số 1 là không bỏ sót người; gộp làm mất người trong dữ liệu. |
| Xe trên xe chở ô tô | 263, 273, 291: gán từng xe; 277: không gán | Gán từng xe nhận ra được | R-06 | Đó là xe thật, mô hình phải thấy. |
| Van | 263, 273, 291, 066: giữ `van`; 277: gộp vào `car` | Giữ `van` (thân liền một hộp) | R-03 | Taxonomy lab có `van`; tiêu chí một hộp đủ rõ. |
| Vật mô phỏng (tượng, đồ chơi, phản chiếu) | 066: `ignore_region`; còn lại: không gán | Không gán, không ignore | R-01, R-13 | `ignore_region` chỉ dành cho cụm vật thật không tách được. |
| Ngưỡng `occluded_level` | 263: 0/30/70 %; 273: 10/35/70 %; 291: 0/25/75 %; 066, 277: chưa có ngưỡng | 10 / 35 / 70 % | mục 2.2 | Ngưỡng 10 % tránh tranh cãi khi chỉ che một góc nhỏ; có ví dụ ảnh cho từng mức. |
| Vật nhỏ | 277: cạnh ngắn < 20 px; còn lại: chiều cao < 20 px | Chiều cao < 20 px | R-07 | Khớp `min_box_height_px` của taxonomy. |
| Kéo box theo phần bị che | 277: cho kéo tối đa 10 %; còn lại: không đoán | Không đoán phần bị che | R-02 | Box modal cho IoU ổn định giữa người gán. |

### 4.2 [v1.1] Bảng phân lớp xe khi mơ hồ

Xét theo thứ tự, dừng ở dòng đầu tiên khớp.

| Bước | Câu hỏi | Có → | Ví dụ |
|---|---|---|---|
| 1 | Ba bánh, hoặc xe chở người bằng sức người không phải xe đạp hai bánh? | `other_vehicle` | tuk-tuk `img_004`, xe lam `img_035`, xích lô `img_023` |
| 2 | Hai bánh? | có động cơ → `motorcycle`; đạp → `bicycle` | `img_029` |
| 3 | Cabin tách khỏi thùng hàng (nhìn thấy khe hoặc vách giữa cabin và thùng)? | `truck`, kể cả pickup có nắp thùng, khung giá | `img_007`, `img_024` (xe đen chở xe vàng là wagon → `car`, không phải truck), `img_037` |
| 4 | Thân liền một hộp, từ 4 hàng cửa sổ hông trở lên **hoặc** dài gấp ≥ 2,5 lần chiều cao? | `bus` | `img_009`, `img_031`, `img_037` (xe buýt phía sau, không phải truck) |
| 5 | Thân liền một hộp, mũi ngắn (capo ngắn hơn ~1/4 chiều dài xe)? | `van` | HiAce `img_005` (16 chỗ → `van`, không phải `bus`); van trắng `img_016` |
| 6 | Còn lại (có capo rõ, tối đa 9 chỗ) | `car` | MPV, SUV `img_011`, `img_016` (MPV có capo → `car`) |

### 4.3 [v1.1] Kết quả calibration: lỗi do người hay do guideline

| Nhóm ca IAA | Số ca | Nguyên nhân | Xử lý | Rule ID | Phạm vi |
|---|---|---|---|---|---|
| Vật mô phỏng bị gán (`img_017`, `img_020`, `img_027`, `img_033`, `img_038`) | phần lớn trong 144 ca `img_038` | **Người** áp dụng sai R-01 (guideline đã rõ) | Xóa box ở bản cuối; không đổi quy tắc, chỉ thêm kiểm đối chứng | R-01, R-13 | `anh` |
| `occluded_level` lệch | 595 | **Guideline** mơ hồ ở ranh giới mức | Thêm R-14; rà lại `occluded_level` của **cả lô** | R-14 | `lo` |
| `truncated` lệch | 68 | **Guideline** mơ hồ ("phần rõ ràng ngoài ảnh") | Mặc định `yes` khi chạm mép; rà cả lô | mục 2.2 | `lo` |
| Bỏ sót / thừa `person` | 844 | Cả hai: người trung cảnh bị bỏ, box người lái lệch | Thêm R-15, R-16; rà cả lô | R-15, R-16 | `lo` |
| Vùng ignore không khớp | 412 | **Guideline** chưa nói cách vẽ cạnh vùng | Bổ sung R-08 (a–d) | R-08 | `lo` |
| Sai lớp xe | 29 | **Guideline** thiếu tiêu chí `van`/`bus`/`car` | Bảng 4.3 | R-03 | `anh` |
| `motorcycle`/`person` ghép nhầm | 11 | **Guideline** chưa nói box xe khi bị người che hết | Thêm R-17 | R-17 | `anh` |
| Pickup gán `car` (`img_007`, `img_024`), người trên xe gộp | — | **Người** dùng bản cá nhân thay vì v1.0 | Sửa nhãn theo R-03, R-04 | R-03, R-04 | `anh` |
| Xe trên giàn `img_002` (một người chỉ gán 2 `car`, người khác 8–9) | — | **Người** bỏ sót | Gán đủ theo R-06 | R-06 | `anh` |

### 4.4 Bảng ca theo ảnh

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_001 | Pickup chở sắt, người ngồi quán | R-03, R-02 | 1 `truck`; mỗi người tách được → `person`, che bởi bàn ghế theo tỉ lệ | Pickup luôn là truck. |
| img_002 | Xe đầu kéo chở 7–8 ô tô, 1 người | R-06, R-07 | 1 `truck` (không bao xe trên giá); mỗi xe nhận ra → `car` (tầng dưới thường mức 2); 1 `person` | Xe trên giá là xe thật. |
| img_003 | Người gánh hàng; xe đạp, xe máy đỗ | R-05, R-03 | `person` không bao quang gánh; xe đạp → `bicycle`; xe máy → `motorcycle` | Quang gánh nhô xa làm lệch IoU. |
| img_004 | Hai tuk-tuk ban đêm có người lái | R-03, R-04 | Tuk-tuk → `other_vehicle`; người lái → `person` | Lớp mới v1.0. |
| img_005 | Nhiều người đi xe máy chồng nhau; van HiAce | R-04, R-11 | Mỗi người → `person`; mỗi xe máy → `motorcycle`; HiAce → `van`; xe trắng góc trái → `car`, `truncated=yes` | Ảnh mẫu R-04. |
| img_006 | Dòng xe từ phía sau: xe máy, xe đạp, taxi | R-04, R-03 | Xe máy + người theo R-04; xe đạp → `bicycle` + người đạp → `person`; taxi → `car` | |
| img_007 | Kho đêm: xe tải Isuzu, pickup có khung giá | R-03 | Cả hai → `truck` | Khung giá không đổi bản chất pickup. |
| img_008 | Xe ôm áo cam, bãi xe máy dày | R-04, R-08 | Xe ôm tiền cảnh → `person` + `motorcycle`; cụm phía sau → `ignore_region` | |
| img_009 | Xe khách ~29 chỗ | R-03, R-07 | → `bus`; xe máy xa chỉ gán nếu ≥ 20 px | > 16 chỗ, 2 trục. |
| img_010 | Người ngồi trên xe máy đỗ, người bán hàng | R-04 | `person` + `motorcycle`; xe đẩy hàng không gán | |
| img_011 | Điêu khắc / người hóa trang; mô tô phế liệu | R-01 | Điêu khắc không gán; không chắc là người thật → `person` + `can_review` | Cần reviewer xem ảnh gốc. |
| img_012 | SUV, pickup chở hàng, xe tải xa | R-03, R-05 | SUV → `car`; pickup → `truck` (không bao hàng nhô); xe tải → `truck` | |
| img_013 | Đường ướt ban đêm, phản chiếu | R-01, R-07, R-12 | Không gán phản chiếu; vật xa chỉ gán khi ≥ 20 px và nhận ra | |
| img_014 | 2 người đi bộ rõ; người + xe nhòe | R-12, R-05 | 2 `person` (không bao túi xách); vật nhòe gán khi nhận ra, box phần đậm nhất | |
| img_015 | Xe cận cảnh cắt mép, xe nhòe, phản chiếu | R-11, R-12, R-01 | Xe → `car`, `truncated=yes`; phản chiếu không gán | |
| img_016 | Tuk-tuk, Vespa có người, van, pickup, phụ nữ cắt mép | R-03, R-04, R-11 | Tuk-tuk → `other_vehicle`; Vespa → R-04; van → `van`; pickup → `truck`; phụ nữ → `person`, `truncated=yes` | |
| img_017 | Tượng người cưỡi ngựa đội nón giao thông | R-01, R-13 | Không gán; ảnh có 0 nhãn | Ca đối chứng. |
| img_018 | Biển quảng cáo có người; xe sau hàng rào | R-01, R-02 | Người trên biển không gán; xe thật → `car`, `occluded_level ≥ 1` | |
| img_019 | Xe diễu hành giả tưởng có người lái; đám đông | R-01, R-08 | Xe → `car` + `can_review`; người lái → `person`; đám đông khó tách → `ignore_region` | Xe chạy được nên là vật thật. |
| img_020 | Gương chiếu hậu phản chiếu xe | R-01, R-10, R-13 | Không gán | Phản chiếu + xe chủ. |
| img_021 | Dòng xe máy rất dày | R-04, R-08 | Cặp người + xe tiền cảnh vẽ riêng; khối phía xa → một `ignore_region` | |
| img_022 | Ảnh đen trắng, mô tô sidecar | R-04, R-12 | 1 `motorcycle` bao cả sidecar; người → `person` | Ảnh đen trắng vẫn là ảnh thật. |
| img_023 | Ô tô đen, xích lô có người đạp | R-03, R-04 | Ô tô → `car`; xích lô → `other_vehicle`; người đạp → `person` | Xích lô không phải xe đạp hai bánh. |
| img_024 | Bảo tàng xe; xe vàng trên nóc wagon | R-01, R-06, R-11 | Xe thật → `car`; xe vàng → `car` + `can_review` nếu nghi là mô hình | |
| img_025 | Phố biển, taxi, xe máy kéo thùng | R-04, R-07 | Người ≥ 20 px → `person`; taxi → `car`; xe máy + thùng gắn liền → 1 `motorcycle` | |
| img_026 | Đôi nam nữ trên xe máy, tranh tường | R-04, R-01 | 2 `person` + 1 `motorcycle`; tranh tường không gán | Ảnh mẫu R-04. |
| img_027 | Tượng đài hình pickup | R-01, R-13 | Không gán, không ignore | Ca đối chứng. |
| img_028 | Pickup cổ chở tượng ngựa; SUV đỏ phía sau | R-01, R-03 | Pickup → `truck` (không bao ngựa); ngựa không gán; SUV → `car` | |
| img_029 | Xe máy điện chở ống dài; người đi xe đạp | R-04, R-05 | Xe điện → `motorcycle` (không bao ống); xe đạp → `bicycle`; người → `person` | |
| img_030 | Bãi xe máy dày, poster mặt người | R-08, R-01 | Xe tiền cảnh → `motorcycle`; khối phía sau → `ignore_region`; poster không gán | |
| img_031 | Phố đêm mưa: SUV, xe con, xe buýt | R-03, R-12, R-01 | Xe con → `car`; xe buýt → `bus`; phản chiếu không gán | |
| img_032 | Đám đông chờ xe buýt; xe con cắt mép | R-08, R-11 | Người tách được → `person`; đoạn ≥ 5 người chồng lấp → `ignore_region`; xe → `car`, `truncated=yes` | |
| img_033 | Ma-nơ-canh, ảnh in, phản chiếu trên kính | R-01 | Không gán gì | |
| img_034 | Người băng đường, ô tô, xe máy | R-04, R-07, R-11 | Gán theo R-04; người cắt mép → `truncated=yes` | |
| img_035 | Xe lam chở hàng cao, mô tô, ô tô | R-03, R-05, R-04 | Xe lam → `other_vehicle` (không bao hàng nhô trên nóc); mô tô + người theo R-04; ô tô → `car` | |
| img_036 | Xe tải nhỏ phủ bạt; dãy xe máy đỗ | R-03, R-08 | Xe tải → `truck`; xe tách được → `motorcycle`; dãy dày → `ignore_region` | Cabin tách thùng. |
| img_037 | Pickup có nắp thùng; xe buýt; người | R-03 | Pickup → `truck`; xe buýt → `bus`; người → `person` | Dễ nhầm `van`. |
| img_038 | Tủ kính mô hình ô tô | R-01, R-13 | Không gán, không ignore | Mô hình học đây là nền. |
| img_039 | Phố nhỏ: xe máy, xe đạp, người | R-04, R-07 | Gán theo R-04; vật < 20 px bỏ | |
| img_040 | Người đội mũ cận cảnh chỉ thấy gương xe; taxi; tuk-tuk; xe buýt | R-11, R-03 | Người → `person`, `truncated=yes`; xe của người đó không gán; taxi → `car`; tuk-tuk → `other_vehicle`; xe buýt → `bus` | |

---

## 5. QC và phiên bản

### 5.1 Ngưỡng (lấy nguyên từ `taxonomy.json` mục `rules`, không sửa)

| Ngưỡng | Giá trị | Dùng ở đâu |
|---|---|---|
| `min_box_height_px` | 20 | R-07, R-11 |
| `large_px` | 50 | phân biệt vật lớn / nhỏ |
| `iou_thr_large` / `iou_thr_small` | 0,85 / 0,70 | box đạt khi IoU với box tham chiếu đủ ngưỡng |
| `match_iou` | 0,5 | ghép cặp box giữa các người gán |
| `iaa_iou` / `iaa_class` | 0,85 / 0,95 | đồng thuận vị trí / lớp trên các cặp ghép được |
| `error_rate_img_max` | 2 % | quá ngưỡng → Rework |

### 5.2 Ai quyết ca mơ hồ

- Lúc gán (Pha 3): **không hỏi nhau**; tick `can_review` hoặc ghi vấn đề cấp ảnh.
- Pha 5: nhóm chốt từng ca trong `calibration_decisions.csv`, ghi rule ID; người kiểm lại **khác** người sửa.
- Quy tắc đổi cách chọn vật, lớp, box hoặc thuộc tính → `semantic`, tăng phiên bản, kiểm lại cả lô (`pham_vi=lo`). Chỉ sửa câu chữ → `editorial`.
- **[v1.1] Kiểm trước khi chốt nhãn cuối:**
  1. 5 ảnh đối chứng `img_017`, `img_020`, `img_027`, `img_033`, `img_038` có 0 annotation.
  2. Không có box vật nào nằm trong `ignore_region` (R-08).
  3. Không có cặp `person`–xe có IoU > 0,8 trên cùng người lái (R-17).
  4. Mọi box chạm mép ≤ 2 px có `truncated=yes`, trừ ca thấy rõ viền vật (mục 2.2).
- Thay đổi v1.1 là **`semantic`** (đổi cách chọn `occluded_level`, `truncated`, box người, vùng ignore), nên `calibration_decisions.csv` phải có ít nhất một quyết định `pham_vi=lo`, và nhóm kiểm lại cả 40 ảnh.

### 5.3 Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của lab, 6 lớp | — |
| v0.1–v0.2 (cá nhân) | 5 bản cá nhân 066, 263, 273, 277, 291 | Viết độc lập ở Pha 1 |
| v1.0 | Thêm lớp `bus`, `bicycle`, `other_vehicle`; chốt pickup = `truck`, người trên xe tách box, xe trên giá chở gán riêng, vật mô phỏng không gán; ngưỡng `occluded_level` 10/35/70 %; quy tắc R-01 … R-13 | Xe buýt `img_009`, `img_031`, `img_037`, `img_040`; tuk-tuk `img_004`, `img_016`, `img_035`; xe đạp `img_003`, `img_006`, `img_029`; pickup `img_037`; người + xe `img_026`; xe chở xe `img_002`; đối chứng `img_017`, `img_027`, `img_038` |
| **v1.1** (`semantic`) | Thêm R-14 (chọn `occluded_level` khi phân vân + bảng quy ước), R-15 (box người), R-16 (bắt buộc gán người ≥ 20 px), R-17 (xe bị người lái che); `truncated` mặc định `yes` khi chạm mép; R-08 thêm cách vẽ vùng ignore; R-03 thêm bảng phân lớp xe (4.2); R-01 thêm kiểm 5 ảnh đối chứng. Taxonomy giữ v1.0. | IAA Pha 4: cùng `occluded_level` 47,7 %, cùng `truncated` 87,1 %, IoU TB 0,838, 844 ca bỏ sót/thừa `person`, 412 ca ignore (`img_032`, `img_036`, `img_008`), 40 ca sai lớp (`img_005`, `img_016`, `img_039`), vật mô phỏng bị gán (`img_017`, `img_020`, `img_027`, `img_033`, `img_038`) |
