# Guideline gán nhãn — v1.1 — nhóm N (066 · 263 · 273 · 277 · 291)

> Bản điền đủ **5 phần** của khung guideline, tóm theo `submission/guidelines/guideline_v1.1_calibrated.md`.
> Mỗi quy tắc chỉ có một cách hiểu; mã **R-xx** dùng lại trong `calibration_decisions.csv` và phiếu QC.
> Chỗ đổi so với v1.0 đánh dấu **[v1.1]**.

- Phiên bản: **v1.1** (hiệu chỉnh từ v1.0 sau IAA Pha 4, thay đổi `semantic`)
- Ngày: 2026-11-16
- Người viết / duyệt: nhóm 066, 263, 273, 277, 291 / cả nhóm duyệt ở Pha 5
- Taxonomy đi kèm: `submission/taxonomy.json` `version = v1.0` (v1.1 không đổi lớp, thuộc tính hay ngưỡng)
- Công cụ: CVAT, xuất COCO 1.0

---

## 1. Mục tiêu

Nhãn bounding box dùng để huấn luyện và đánh giá mô hình **phát hiện người và phương tiện thật trên ảnh đường phố 2D** (camera giao thông, hỗ trợ lái). Mô hình phải báo được vật **thật, có thể di chuyển hoặc va chạm**.

Sai nguy hiểm nhất, theo thứ tự:

1. **Bỏ sót người thật**, nhất là người đi xe hoặc bị che một phần: mô hình không thấy người đi đường.
2. **Gán vật không thật thành vật thật** (tượng, ma-nơ-canh, ảnh in, phản chiếu, đồ chơi): mô hình báo động giả, phanh gấp.
3. **Nhầm lớp giữa các xe** (`car` / `truck` / `van` / `bus`): ít nguy hiểm hơn nhưng kéo đồng thuận lớp xuống dưới 0,95.

Khi phân vân có gán một **người thật** hay không: **gán và tick `can_review`**.

---

## 2. Lớp và thuộc tính

### Lớp

Chín lớp, mỗi box thuộc **đúng một** lớp. So với v0 có thêm `bus`, `bicycle`, `other_vehicle` (xuất hiện ở ≥ 4 ảnh, v0 không có lớp để gán).

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người tối đa 9 chỗ, có capo tách khoang khách: sedan, hatchback, wagon, SUV, MPV, taxi. | Sedan `img_002`; SUV `img_012`; taxi `img_040`; xe trưng bày thật `img_024` | Pickup → `truck` (R-03). Mô hình, đồ chơi `img_038` → không gán (R-01). |
| `truck` | Xe chở hàng có **cabin tách khỏi thùng hàng**: xe tải, pickup (kể cả có nắp thùng, khung giá nóc), đầu kéo, xe chở ô tô. | Pickup `img_001`, `img_028`, `img_037`; xe tải `img_007`, `img_036`; xe chở ô tô `img_002` | Thân liền một hộp → `van`. Tượng pickup `img_027` → không gán. |
| `van` | Xe thân **liền một hộp**, mũi ngắn, tối đa 16 chỗ, chở người hoặc hàng. | Toyota HiAce `img_005`; van trắng `img_016` | SUV/MPV có capo dài → `car`. Pickup có nắp thùng → `truck`. Từ 17 chỗ → `bus`. |
| `bus` | Xe buýt, xe khách, minibus **từ 17 chỗ**. | Xe khách `img_009`; xe buýt `img_031`, `img_037`, `img_040` | Van 16 chỗ → `van`. |
| `motorcycle` | Xe **hai bánh có động cơ**: xe máy, tay ga, mô tô, xe máy điện; kể cả sidecar hoặc thùng **gắn cố định**. | Tay ga `img_010`, `img_026`; sidecar `img_022`; xe điện `img_029` | Xe đạp → `bicycle`. Tuk-tuk, xe lam → `other_vehicle`. |
| `bicycle` | Xe đạp **hai bánh** chạy bằng sức người, kể cả xe trợ lực còn bàn đạp. | `img_003`, `img_006`, `img_029`, `img_039` | Xích lô, ba gác đạp → `other_vehicle`. |
| `other_vehicle` | Phương tiện có bánh **thật** không thuộc lớp trên: tuk-tuk, xe lam, xích lô, ba gác, xe ba bánh chở hàng. | Tuk-tuk `img_004`, `img_016`, `img_035`, `img_040`; xích lô `img_023` | Xe đẩy hàng, xe đẩy em bé → không gán. |
| `person` | **Người thật** nhìn thấy trực tiếp, mọi tư thế, kể cả người lái và hành khách trên **mọi** phương tiện. | Người đi bộ `img_014`, `img_034`; 2 người trên xe máy `img_026` | Tượng `img_017`, ma-nơ-canh `img_033`, người trên biển/poster `img_018`, `img_030` → không gán. |
| `ignore_region` | **Vùng** chứa **từ 5 vật** thuộc lớp mục tiêu chồng lấn đến mức không xác định được mép từng vật. | Dòng xe máy xa `img_021`; đám đông `img_019`; bãi xe `img_030` | Không dùng cho vật không chắc (dùng `can_review`), không dùng để che tượng, đồ chơi, phản chiếu. |

**Vật không thuộc lớp nào** (động vật, xe đẩy hàng, nón giao thông, biển báo, tượng, đồ chơi): **không vẽ box, không dùng `ignore_region`, không thêm lớp mới**. Người thật đi cùng các vật này vẫn gán `person`.

### Thuộc tính

Mọi box (trừ `ignore_region`) có đủ 3 thuộc tính. Không dùng cờ `Occluded` có sẵn của CVAT.

**`occluded_level`**: tỉ lệ diện tích vật **bị vật khác che**, so với hình dạng đầy đủ đáng lẽ nằm trong ảnh. Phần ngoài mép ảnh **không** tính ở đây (ghi bằng `truncated`).

| Mức | Ngưỡng | Ví dụ |
|---|---|---|
| `0` | che < 10 % | pickup `img_028`; người đi bộ `img_034` |
| `1` | 10 % đến < 35 % | xe máy bị chân người lái che `img_026`; xe sau rào thưa `img_018` |
| `2` | 35 % đến < 70 % | người ngồi sau bị người lái che `img_005`; ô tô tầng dưới giá chở `img_002` |
| `3` | ≥ 70 %, chỉ thấy phần nhỏ nhưng **vẫn nhận ra lớp** | người chỉ lộ đầu và vai trong dòng xe `img_021` |

**[v1.1] R-14 — chọn mức khi phân vân:**

1. Chỉ tính **vật khác** che. Mép ảnh, bóng tối, nhòe, bộ phận của chính vật đó **không** tính.
2. Chia vật thành 4 phần (trên/dưới × trái/phải), mỗi phần bị che hết ≈ 25 %; làm tròn **xuống**. Sát ranh giới 10 / 35 / 70 % → chọn **mức thấp hơn**.
3. Quy ước sẵn (khi không có vật che khác):

| Ca | `person` | `motorcycle` / `bicycle` |
|---|---|---|
| Người lái xe máy, nhìn ngang | `1` | `1` |
| Người lái xe máy, nhìn thẳng trước / sau | `1` | `2` |
| Người ngồi sau, bị người lái che nửa thân | `2` | không đổi |
| Người sau bàn, quầy che từ hông trở xuống | `2` | — |
| Xe sau hàng rào song thưa | — | `1` |

4. Không nhận ra lớp → **không gán**; không dùng mức `3` với nghĩa "không rõ". Vẫn phân vân → chọn mức thấp hơn, **không** tick `can_review`.

**`truncated`** **[v1.1]**

| Giá trị | Khi nào |
|---|---|
| `yes` | Box chạm mép ảnh (cách ≤ 2 px). Mặc định `yes` khi chạm mép. |
| `no` | Box không chạm mép, **hoặc** chạm mép nhưng nhìn rõ đường viền ngoài của vật (lốp, gót chân, nóc xe) nằm trong ảnh. |

Một vật có thể vừa `truncated=yes` vừa `occluded_level > 0`.

**`can_review`** (tick / không): tick khi **lớp** hoặc **việc có gán hay không** chưa chắc. Không tick vì box khó vẽ hay vì phân vân mức che. Vật không vẽ được box thì ghi vào `submission/qc/image_issues.csv`.

---

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| **R-01** Vật thật | Chỉ gán vật **thật, cỡ thật**. Không gán: tượng (`img_017`, `img_027`, ngựa `img_028`), ma-nơ-canh (`img_033`), người/xe trên biển, poster, tranh tường (`img_018`, `img_026`, `img_030`), **phản chiếu** trong gương, kính, mặt đường ướt (`img_013`, `img_015`, `img_020`, `img_031`), đồ chơi, mô hình (`img_038`). Xe thật đang trưng bày (`img_024`) vẫn gán. Xe trang trí lạ nhưng chạy được, có người lái (`img_019`) → lớp kết cấu gần nhất + `can_review`. Không chắc người hay tượng → `person` + `can_review`. **[v1.1]** 5 ảnh đối chứng `img_017`, `img_020`, `img_027`, `img_033`, `img_038` phải có **0 annotation**. |
| **R-02** Box sát phần nhìn thấy | Hình chữ nhật nhỏ nhất ôm **pixel nhìn thấy**, **không đoán** phần bị che. Vật bị che tách thành nhiều mảnh vẫn **một** box bao các mảnh. Sai số mép ≤ 2 px với vật lớn. Không bao bóng đổ. |
| **R-03** Lớp xe theo kết cấu | Chọn theo **kết cấu thân xe** (bảng ở mục 4), không theo màu, kích thước, mục đích. Pickup luôn `truck`. Không nhìn rõ kết cấu → lớp gần nhất + `can_review`. |
| **R-04** Người + xe | Mỗi người trên xe = **một box `person`** + **một box riêng cho xe**; hai box được chồng nhau. Box `person` gồm mũ, quần áo; box xe gồm thân, bánh, gương, tay lái, sidecar/thùng gắn cố định. Ví dụ `img_026` = 2 `person` + 1 `motorcycle`. |
| **R-05** Đồ mang theo, hàng chở | Không kéo box bao đồ cầm, gánh, chở **nhô ra khỏi thân**: quang gánh `img_003`, ống dài `img_029`, túi xách thõng `img_014`, hàng trên nóc `img_035`. Hàng **trong** thùng xe đã nằm trong box xe. |
| **R-06** Xe chở xe | Đầu kéo + rơ-moóc = **một** box `truck` theo khung và giá chở, không bao xe phía trên. Mỗi xe trên giá đủ điều kiện R-07 → box riêng, tính che bởi khung giá (`img_002`). |
| **R-07** Vật nhỏ | Chiều cao phần nhìn thấy **< 20 px** trên ảnh gốc → không vẽ. 20–49 px chỉ vẽ khi **nhận ra lớp**; nếu không thì bỏ hoặc gộp vào `ignore_region` theo R-08. |
| **R-08** Đám đông / dòng xe dày | **Từ 5 vật** chồng nhau không tách được mép → **một** `ignore_region` bao cả cụm. Vật tiền cảnh tách được vẽ box riêng **ngoài** vùng. Không đặt box vật bên trong vùng ignore. **[v1.1]** (a) không dùng ignore cho cụm mà mỗi vật cao ≥ 50 px; (b) vùng ôm sát cụm, cạnh dưới ở chân vật gần nhất của cụm; (c) mỗi cụm liền nhau một vùng; (d) ảnh không có cụm như vậy thì không có `ignore_region`. |
| **R-09** Một vật, một box | Mỗi vật độc lập một box; không gộp hai xe, không vẽ box cho bộ phận (chỉ bánh xe), không vẽ hai box cho một vật. |
| **R-10** Xe chủ (ego) | Phần xe của người chụp (gương, khung cửa, capo) không gán (`img_020`). |
| **R-11** Cắt mép | Vật bị cắt mép vẫn gán nếu phần trong ảnh nhận ra lớp và cao ≥ 20 px; `truncated=yes`. Xe chỉ lộ gương hoặc tay lái → không gán (`img_040`). |
| **R-12** Nhòe / tối / ảnh cũ | Vẽ khi **nhận ra lớp**; box theo phần đậm, rõ nhất, không bao bóng mờ. Không nhận ra lớp → không gán. Nhòe không phải che khuất. |
| **R-13** Ảnh không có vật | Ảnh không có vật cần gán vẫn **phải có mặt** trong COCO với 0 annotation. |
| **R-15 [v1.1]** Box người | Từ **đỉnh đầu / đỉnh mũ** đến **điểm thấp nhất nhìn thấy của cơ thể**, rộng đến hai tay hoặc vai; **không** gồm tay lái, thân xe, túi thõng. Người ngồi sau vẫn có box riêng theo phần nhìn thấy. |
| **R-16 [v1.1]** Ưu tiên gán người | Người thật cao ≥ 20 px, thấy đầu và thân → **bắt buộc** có box, kể cả ở trung cảnh, vỉa hè, trong quán, trừ khi nằm trong `ignore_region` hợp lệ. |
| **R-17 [v1.1]** Xe sau người lái | Xe nhìn thẳng trước/sau, gần như bị người lái che: vẫn vẽ box xe nếu thấy bánh hoặc đầu xe cao ≥ 20 px, chỉ bao phần xe nhìn thấy. Box `person` và box xe có IoU > 0,8 → xem lại cả hai. |

**Thứ tự tự kiểm khi gặp một vật:** Thật không? (R-01) → Lớp nào? (R-03) → Đủ lớn, nhận ra được? (R-07, R-11, R-12, R-16) → Tách được khỏi cụm? (R-08) → Vẽ box (R-02, R-04, R-05, R-06, R-09, R-15, R-17) → Thuộc tính (mục 2, R-14).

---

## 4. Ca khó

### Bảng phân lớp xe khi mơ hồ (R-03) — dừng ở dòng đầu tiên khớp

| Bước | Câu hỏi | Có → | Ví dụ |
|---|---|---|---|
| 1 | Ba bánh, hoặc xe chở người bằng sức người không phải xe đạp hai bánh? | `other_vehicle` | tuk-tuk `img_004`, xe lam `img_035`, xích lô `img_023` |
| 2 | Hai bánh? | động cơ → `motorcycle`; đạp → `bicycle` | `img_029` |
| 3 | Cabin tách khỏi thùng hàng (thấy khe hoặc vách)? | `truck` | `img_007`, `img_037` |
| 4 | Thân liền một hộp, ≥ 4 hàng cửa sổ hông **hoặc** dài ≥ 2,5 lần chiều cao? | `bus` | `img_009`, `img_031`, `img_037` |
| 5 | Thân liền một hộp, capo ngắn hơn ~1/4 chiều dài? | `van` | HiAce `img_005`, `img_016` |
| 6 | Còn lại (có capo rõ, ≤ 9 chỗ) | `car` | SUV `img_011`, MPV `img_016`, wagon `img_024` |

### Bảng ca theo ảnh

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_001 | Pickup chở sắt, người ngồi quán | R-03, R-16 | 1 `truck`; mỗi người tách được → `person`, che bởi bàn theo R-14 | Pickup luôn là truck. |
| img_002 | Đầu kéo chở 7–8 ô tô, 1 người | R-06, R-07 | 1 `truck` (không bao xe trên giá); mỗi xe nhận ra → `car`; 1 `person` | Xe trên giá là xe thật. |
| img_003 | Người gánh hàng; xe đạp, xe máy đỗ | R-05, R-03 | `person` không bao quang gánh; `bicycle`; `motorcycle` | Đồ nhô ra làm lệch IoU. |
| img_004 | Hai tuk-tuk ban đêm có người lái | R-03, R-04 | `other_vehicle` + `person` | Ba bánh. |
| img_005 | Nhiều người đi xe máy chồng nhau; van HiAce | R-04, R-11, R-14 | Mỗi người → `person`; mỗi xe → `motorcycle`; HiAce → `van`; xe góc trái → `car`, `truncated=yes` | HiAce 16 chỗ không phải bus. |
| img_006 | Dòng xe từ phía sau: xe máy, xe đạp, taxi | R-04, R-17 | Xe + người theo R-04; `bicycle` + `person`; taxi → `car` | |
| img_007 | Kho đêm: xe tải, pickup có khung giá | R-03 | Cả hai → `truck` | Khung giá không đổi bản chất pickup. |
| img_008 | Xe ôm tiền cảnh, bãi xe máy dày | R-04, R-08 | Tiền cảnh → `person` + `motorcycle`; cụm sau → `ignore_region` | |
| img_009 | Xe khách ~29 chỗ | R-03, R-07 | `bus`; xe máy xa chỉ gán nếu ≥ 20 px | > 16 chỗ. |
| img_010 | Người ngồi trên xe máy đỗ, xe đẩy hàng | R-04 | `person` + `motorcycle`; xe đẩy không gán | |
| img_011 | Điêu khắc / người hóa trang | R-01 | Điêu khắc không gán; nghi người thật → `person` + `can_review` | Reviewer xem ảnh gốc. |
| img_012 | SUV, pickup chở hàng, xe tải xa | R-03, R-05 | `car`; `truck` (không bao hàng nhô); `truck` | |
| img_013 | Đường ướt ban đêm, phản chiếu | R-01, R-12 | Không gán phản chiếu; vật xa gán khi ≥ 20 px và nhận ra | |
| img_014 | 2 người đi bộ rõ; vật nhòe | R-12, R-05 | 2 `person` (không bao túi); vật nhòe box phần đậm nhất | |
| img_015 | Xe cận cảnh cắt mép, nhòe, phản chiếu | R-11, R-12, R-01 | `car`, `truncated=yes`; phản chiếu không gán | |
| img_016 | Tuk-tuk, Vespa có người, van, MPV, phụ nữ cắt mép | R-03, R-04, R-11 | `other_vehicle`; Vespa theo R-04; `van`; MPV → `car`; phụ nữ `person`, `truncated=yes` | |
| img_017 | Tượng người cưỡi ngựa | R-01, R-13 | 0 annotation | Ảnh đối chứng. |
| img_018 | Biển quảng cáo có người; xe sau rào | R-01, R-14 | Người trên biển không gán; xe → `car`, `occluded_level=1` | |
| img_019 | Xe diễu hành có người lái; đám đông | R-01, R-08 | Xe → `car` + `can_review`; người lái `person`; đám đông → `ignore_region` | Xe chạy được nên là vật thật. |
| img_020 | Gương chiếu hậu phản chiếu xe | R-01, R-10, R-13 | 0 annotation | Phản chiếu + xe chủ. |
| img_021 | Dòng xe máy rất dày | R-04, R-08 | Cặp tiền cảnh vẽ riêng; khối xa → một `ignore_region` | |
| img_022 | Ảnh đen trắng, mô tô sidecar | R-04, R-12 | 1 `motorcycle` bao cả sidecar; `person` | Ảnh cũ vẫn là ảnh thật. |
| img_023 | Ô tô đen, xích lô có người đạp | R-03, R-04 | `car`; `other_vehicle`; `person` | Xích lô không phải xe đạp hai bánh. |
| img_024 | Bảo tàng xe; xe vàng trên nóc wagon | R-01, R-06 | Xe thật → `car`; xe vàng `car` (+ `can_review` nếu nghi mô hình) | Wagon không phải truck. |
| img_025 | Phố biển, taxi, xe máy kéo thùng | R-04, R-16 | Người ≥ 20 px → `person`; `car`; xe máy + thùng gắn liền → 1 `motorcycle` | |
| img_026 | Đôi nam nữ trên xe máy, tranh tường | R-04, R-01 | 2 `person` + 1 `motorcycle`; tranh tường không gán | Ảnh mẫu R-04. |
| img_027 | Tượng đài hình pickup | R-01, R-13 | 0 annotation | Ảnh đối chứng. |
| img_028 | Pickup cổ chở tượng ngựa; SUV đỏ | R-01, R-03 | `truck` (không bao ngựa); `car` | |
| img_029 | Xe máy điện chở ống dài; xe đạp | R-04, R-05 | `motorcycle` (không bao ống); `bicycle`; `person` | |
| img_030 | Bãi xe máy dày, poster mặt người | R-08, R-01 | Xe tiền cảnh `motorcycle`; khối sau `ignore_region`; poster không gán | |
| img_031 | Phố đêm mưa: SUV, xe con, xe buýt | R-03, R-12, R-01 | `car`; `bus`; phản chiếu không gán | |
| img_032 | Đám đông chờ xe buýt; xe cắt mép | R-08, R-11, R-16 | Người tách được `person`; đoạn ≥ 5 người chồng lấn `ignore_region`; xe `car`, `truncated=yes` | |
| img_033 | Ma-nơ-canh, ảnh in, phản chiếu kính | R-01, R-13 | 0 annotation | Ảnh đối chứng. |
| img_034 | Người băng đường, ô tô, xe máy | R-04, R-16, R-11 | Gán theo R-04; người cắt mép `truncated=yes` | |
| img_035 | Xe lam chở hàng cao, mô tô, ô tô | R-03, R-05 | `other_vehicle` (không bao hàng trên nóc); mô tô + người; `car` | |
| img_036 | Xe tải nhỏ phủ bạt; dãy xe máy đỗ | R-03, R-08 | `truck`; xe tách được `motorcycle`; dãy dày `ignore_region` | Cabin tách thùng. |
| img_037 | Pickup có nắp thùng; xe buýt; người | R-03 | `truck`; `bus`; `person` | Dễ nhầm `van`. |
| img_038 | Tủ kính mô hình ô tô | R-01, R-13 | 0 annotation | Mô hình học đây là nền. |
| img_039 | Phố nhỏ: xe máy, xe đạp, người | R-04, R-17, R-07 | Gán theo R-04/R-17; vật < 20 px bỏ | |
| img_040 | Người đội mũ cận cảnh chỉ thấy gương xe; taxi; tuk-tuk; xe buýt | R-11, R-03 | `person`, `truncated=yes`; xe của người đó không gán; `car`; `other_vehicle`; `bus` | |

Nếu chưa chốt, đánh dấu `can_review` trên box hoặc ghi vấn đề cấp ảnh vào `submission/qc/image_issues.csv`. Dùng mã quy tắc trong nhật ký QC.

---

## 5. QC và phiên bản

### Ngưỡng QC (lấy nguyên từ `taxonomy.json`, mục `rules`)

| Ngưỡng | Giá trị | Dùng ở đâu |
|---|---|---|
| `min_box_height_px` | 20 | R-07, R-11, R-16 |
| `large_px` | 50 | phân biệt vật lớn / nhỏ, R-08 (a) |
| `iou_thr_large` / `iou_thr_small` | 0,85 / 0,70 | box đạt khi IoU với box tham chiếu đủ ngưỡng |
| `match_iou` | 0,5 | ghép cặp box giữa người gán |
| `iaa_iou` / `iaa_class` | 0,85 / 0,95 | đồng thuận vị trí / lớp trên các cặp ghép được |
| `error_rate_img_max` | 2 % | quá ngưỡng → Rework |

### Ai quyết ca mơ hồ

- Lúc gán (Pha 3): **không hỏi nhau**; tick `can_review` hoặc ghi vấn đề cấp ảnh.
- Pha 5: nhóm chốt từng ca trong `calibration_decisions.csv` kèm rule ID; người kiểm lại **khác** người sửa.
- Đổi cách chọn vật, lớp, box hoặc thuộc tính → `semantic`, tăng phiên bản, kiểm lại cả lô (`pham_vi=lo`). Chỉ sửa câu chữ → `editorial`.
- **[v1.1] Kiểm trước khi chốt nhãn cuối:**
  1. 5 ảnh đối chứng `img_017`, `img_020`, `img_027`, `img_033`, `img_038` có 0 annotation.
  2. Không có box vật nào nằm trong `ignore_region` (R-08).
  3. Không có cặp `person`–xe IoU > 0,8 trên cùng người lái (R-17).
  4. Mọi box chạm mép ≤ 2 px có `truncated=yes`, trừ ca thấy rõ viền vật.

### Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của lab, 6 lớp, thuộc tính chưa có ngưỡng | — |
| v0.1–v0.2 (cá nhân) | 5 bản cá nhân 066, 263, 273, 277, 291 | Viết độc lập ở Pha 1 |
| v1.0 | Thêm `bus`, `bicycle`, `other_vehicle`; pickup = `truck`; người trên xe tách box; xe trên giá chở gán riêng; vật mô phỏng không gán, không ignore; `occluded_level` 10/35/70 %; R-01 … R-13 | Xe buýt `img_009`, `img_031`; tuk-tuk `img_004`, `img_035`; xe đạp `img_003`, `img_029`; pickup `img_037`; người + xe `img_026`; xe chở xe `img_002`; đối chứng `img_017`, `img_027`, `img_038` |
| **v1.1** (`semantic`) | Thêm R-14 (chọn `occluded_level`), R-15 (box người), R-16 (bắt buộc gán người ≥ 20 px), R-17 (xe bị người lái che); `truncated` mặc định `yes` khi chạm mép; R-08 thêm cách vẽ vùng ignore; R-03 thêm bảng phân lớp xe; R-01 thêm kiểm 5 ảnh đối chứng. Taxonomy giữ v1.0. | IAA Pha 4 (1137 cặp): cùng `occluded_level` 47,7 %, cùng `truncated` 87,1 %, IoU TB 0,838; 844 ca bỏ sót/thừa `person`; 412 ca ignore (`img_032`, `img_036`, `img_008`); 40 ca sai lớp (`img_005`, `img_016`, `img_039`); vật mô phỏng bị gán (`img_017`, `img_020`, `img_027`, `img_033`, `img_038`) |
