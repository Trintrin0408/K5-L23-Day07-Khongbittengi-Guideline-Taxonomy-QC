# Guideline gán nhãn — v0.1 (bản cá nhân) — 164

- Phiên bản: v0.1-164 (sửa từ v0 starter, Pha 1, viết độc lập)
- Ngày: 2026-11-16
- Người viết / duyệt: 164 / (chờ nhóm duyệt ở Pha 2)
- Loại nhãn: bounding box trên ảnh 2D, CVAT, xuất COCO 1.0. Danh sách lớp và thuộc tính giữ như `taxonomy` v0.

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người và phương tiện thật đang tham gia giao thông** trên ảnh đường phố (camera giao thông / hỗ trợ lái).

Các lỗi nguy hiểm, xếp theo mức độ:
1. **Bỏ sót người thật** (nhất là người đi bộ, người lái xe máy). Mô hình học được thói quen "không thấy" người.
2. **Gán vật không thật là vật thật** (tượng, ma-nơ-canh, ảnh in, đồ chơi, phản chiếu). Mô hình sẽ báo động giả.
3. **Lẫn lớp giữa các phương tiện** (pickup ↔ car, van ↔ car). Ít nguy hiểm hơn nhưng làm giảm độ đồng thuận.

Nguyên tắc chung: **gán cái mà một camera thật cần nhận ra là vật thật trong cảnh**, chỉ dựa vào phần nhìn thấy trên ảnh.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Xe 4 bánh chở người, thân kín, tối đa ~9 chỗ: sedan, hatchback, SUV, wagon, taxi. | img_023 (xe đen), img_015, img_024, img_031, img_032 | Pickup có thùng hở → `truck` (R-05). Xe thùng dạng hộp → `van`. Mô hình xe đồ chơi img_038 → không gán (R-08). |
| `truck` | Xe chủ yếu để chở hàng, có cabin tách khỏi thùng hàng: xe tải, xe đầu kéo, **pickup** (kể cả pickup có nắp thùng). | img_002 (xe chở ô tô), img_007 (Isuzu), img_012, img_036, img_037, img_001, img_028 | Tượng pickup bằng đá img_027 → không gán. Xe buýt/minibus → ngoài taxonomy (R-09). |
| `van` | Xe 4 bánh thân liền dạng hộp, không có cabin tách rời, mũi xe ngắn (minivan, xe Hiace). | img_005 (xe biển 51LD), img_016 (xe trắng sau xe máy) | Minibus nhiều hàng ghế, cửa lên xuống kiểu xe khách (img_009) → ngoài taxonomy. SUV → `car`. |
| `motorcycle` | Xe 2 bánh có động cơ: xe máy, xe tay ga, mô tô; xe có thùng bên (sidecar) tính là một xe. | img_006, img_008, img_010, img_026, img_029 (xe tay ga), img_022 (sidecar) | Xe đạp, xích lô (img_023, img_006) → không gán. Tuk-tuk / xe lam 3 bánh có mái (img_004, img_035) → ngoài taxonomy (R-09). |
| `person` | Người thật, đứng, đi, ngồi hoặc đang lái/ngồi trên bất kỳ phương tiện nào. | img_014, img_034, img_003, người lái xe máy img_005, người đạp xích lô img_023, người mặc trang phục hóa trang img_011 | Tượng img_017, ma-nơ-canh img_033, người in trên biển quảng cáo img_018, tranh tường img_026 → không gán (R-08). |
| `ignore_region` | Vùng có nhiều vật thuộc taxonomy nhưng không vẽ từng box được (quá dày, quá nhỏ, quá mờ). | Đám đông sau rào img_019, khối xe máy xa img_021, bãi xe dày img_030 | Không dùng để bao vật ngoài taxonomy hay vật giả. |

**Vật không thuộc lớp nào** (xe đạp, xích lô, tuk-tuk, xe buýt, minibus, động vật, đồ vật): **không gán box**, không dùng `ignore_region`. Người đang ngồi trên chúng vẫn gán `person` (R-09).

### Thuộc tính

**`occluded_level`**: tỉ lệ phần vật **nằm trong khung ảnh** bị vật khác che (ước lượng bằng mắt theo diện tích).

| Mức | Nghĩa | Ngưỡng |
|---|---|---|
| `0` | Thấy rõ | bị che < 10% |
| `1` | Che ít | 10% đến < 50% |
| `2` | Che nhiều | ≥ 50% (vẫn nhận ra được loại vật) |
| `3` | Không ước lượng được | quá tối, quá mờ, hoặc chỉ thấy qua kính/lưới nên không đoán được phần bị che |

- Phần vật nằm **ngoài mép ảnh không tính** vào `occluded_level` (đã có `truncated`).
- Người lái / người ngồi trên xe **không tính** là che chiếc xe của chính họ. Nếu tính, gần như mọi xe máy có người đều thành mức 1–2 và thuộc tính mất ý nghĩa. Vật khác (xe bên cạnh, cột, người đi qua) thì tính.
- Hàng rào song thưa (img_018): tính như che mức `1` nếu vẫn thấy rõ đường viền xe.

**`truncated`**: `yes` khi box chạm mép ảnh **và** rõ ràng vật còn tiếp tục ra ngoài khung (ví dụ img_015 hai xe hai bên, img_040 taxi). Vật chỉ chạm mép mà đã thấy hết → `no`.

**`can_review`**: tick khi đã áp dụng hết quy tắc mà vẫn còn hai cách hiểu. Ghi ảnh + lý do vào nhật ký. Không tick chỉ vì box khó vẽ.

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| R-01 | Box ôm sát **phần nhìn thấy** của vật (sai lệch ≤ 2–3 px mỗi cạnh). Không đoán và kéo dài box ra phần bị che. |
| R-02 | **Người đi xe máy = 2 box riêng**: một `person` cho mỗi người (lái và người ngồi sau, kể cả trẻ em) và một `motorcycle` cho chiếc xe. Box người bao đầu (kể cả mũ bảo hiểm) tới phần chân nhìn thấy. Box xe bao từ bánh trước tới bánh/đuôi sau, gương và tay lái; phần người ngồi đè lên xe vẫn nằm trong box xe. img_005 có 4–5 người trên các xe máy → mỗi người 1 box. |
| R-03 | **Hàng hóa mang theo không nằm trong box**: ống dài trên xe máy img_029, gánh hàng img_003, hàng chất trên xe img_035. Riêng hàng trong thùng pickup (img_012) và thùng xe tải thì nằm trong box xe vì thuộc thân xe. |
| R-04 | **Vật bị che**: chỉ vẽ phần thấy được, đặt `occluded_level` theo ngưỡng. Một vật bị chia làm hai phần (ví dụ cột che giữa xe) vẫn **một box** bao cả hai phần. |
| R-05 | **Pickup = `truck`**, kể cả pickup có nắp thùng (img_037) hay chở đồ (img_012, img_028). Lý do: có cabin tách thùng hàng, dùng để chở hàng. |
| R-06 | **Cắt mép**: vẽ tới mép ảnh, `truncated = yes`. Vật còn thấy < 20% thân nhưng nhận ra được loại thì vẫn gán. |
| R-07 | **Vật nhỏ**: box cao < **20 px** (theo `min_box_height_px`) thì không vẽ box riêng. Nhiều vật nhỏ tụ lại → một `ignore_region`. Vật lẻ < 20 px → bỏ qua. |
| R-08 | **Vật không thật không gán**: tượng (img_017, img_027), ma-nơ-canh (img_033), người/xe in trên biển hay tranh (img_018, img_026), đồ chơi, mô hình (img_038), **ảnh phản chiếu** trong gương, kính, vũng nước (img_020, img_013, img_031). Xe thật đang trưng bày (img_024) hay xe diễu hành chạy được (img_019) **là xe thật** → gán. |
| R-09 | **Ngoài taxonomy**: xe đạp, xích lô, tuk-tuk/xe lam, xe buýt, minibus không gán box. Người trên các xe này vẫn gán `person`. |
| R-10 | **Đám đông / khối xe dày**: vật nào tách được (thấy rõ ≥ 50% thân, cao ≥ 20 px) thì vẽ box riêng. Phần còn lại dày quá, chồng chéo không tách được thì bao bằng **một** `ignore_region`. Không để `ignore_region` trùm lên box riêng đã vẽ. |
| R-11 | **Xe chủ (ego)**: phần xe của người chụp (khung gương, cửa, nắp capo) không gán (img_020). |
| R-12 | **Mờ chuyển động**: vẫn gán nếu nhận ra loại vật và ước được biên (img_014). Biên quá nhòe thì bao theo phần đặc nhất và `occluded_level = 3`. Không nhận ra loại thì đưa vào `ignore_region`. |
| R-13 | **Xe chở xe** (img_002): xe tải là một box `truck` bao cả cabin và giá chở. Mỗi ô tô trên giá là xe thật → gán `car` riêng, với `occluded_level` theo phần bị khung che. |

## 4. Ca khó

Loại **Q** = áp dụng quy tắc là quyết được. Loại **H** = cần hỏi reviewer/nhóm, tạm thời tick `can_review`.

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do | Loại |
|---|---|---|---|---|---|
| img_005 | Nhiều người đi xe máy chen sát nhau, xe máy gần như bị người che hết | R-02, R-04 | Mỗi người 1 `person`. Mỗi xe máy thấy được (đầu xe, gương, tay lái) 1 `motorcycle`, `occluded_level` 2. Xe `van` phía sau `occluded_level` 1–2. | Người là ưu tiên. Xe vẫn nhận ra được nên không bỏ. | Q |
| img_026 | Hai người trên một xe máy, nền là tranh tường cảng | R-02, R-08 | 2 `person` + 1 `motorcycle`. Tranh tường không gán. | Áp dụng thẳng R-02, R-08. | Q |
| img_029 | Người chở ống dài trên xe tay ga, cạnh người đạp xe đạp | R-02, R-03, R-09 | Xe tay ga: `motorcycle` + `person`. Ống **không** nằm trong box. Xe đạp không gán, người đạp `person`. | Hàng hóa không phải thân xe. Xe đạp ngoài taxonomy. | Q |
| img_022 | Mô tô có thùng bên (sidecar), ảnh đen trắng | R-02 | Một `motorcycle` bao cả xe và thùng bên. Người ngồi `person`. | Sidecar gắn liền, cùng di chuyển như một xe. | H — hỏi nhóm có tách sidecar không |
| img_002 | Xe đầu kéo chở 8 ô tô | R-13, R-05 | 1 `truck` + mỗi ô tô 1 `car` (tầng dưới `occluded_level` 1–2 vì khung che). | Ô tô trên giá là xe thật, camera vẫn phải nhận ra. | H — nhóm có thể muốn không gán xe được chở |
| img_001, img_012, img_028, img_037 | Pickup | R-05 | `truck`. | v0 không nói pickup thuộc lớp nào, hai người dễ đọc ra `car` và `truck`. | Q |
| img_036 | Xe tải nhỏ thùng phủ bạt | R-05 | `truck`. | Cabin tách thùng hàng. | Q |
| img_009 | Minibus (Coaster) | R-09 | Không gán. | Không phải van (có cửa và nhiều hàng ghế kiểu xe khách). | H — ranh giới van / minibus |
| img_004, img_016, img_035, img_040 | Tuk-tuk, xe lam 3 bánh | R-09 | Không gán xe. Người lái/ngồi `person`. Xe máy, van, car xung quanh vẫn gán. | Không phải xe máy 2 bánh, cũng không phải ô tô. | H — tuk-tuk rất phổ biến, nhóm có thể muốn thêm lớp |
| img_017 | Tượng người cưỡi ngựa | R-08 | Không gán. | Vật không thật. | Q |
| img_027 | Tượng pickup bằng đá ở nghĩa trang | R-08 | Không gán. | Vật không thật, dù hình dạng giống hệt xe. | Q |
| img_028 | Tượng ngựa đặt trên pickup thật | R-05, R-08 | Pickup `truck`, tượng ngựa không gán. Xe đỏ phía sau `car`. | Phương tiện thật, vật trang trí thì không. | Q |
| img_033 | Ma-nơ-canh và phản chiếu trong kính cửa hàng | R-08 | Không gán gì. | Không có người thật. Phản chiếu không gán. | Q |
| img_018 | Biển quảng cáo có ảnh người; ô tô sau hàng rào | R-08, R-04 | Người trên biển không gán. Mỗi ô tô `car`, `occluded_level` 1 (rào song thưa) hoặc 2 nếu xe khác che. | Người in không phải người thật. | Q |
| img_020 | Ô tô trong gương chiếu hậu; khung gương của xe chủ | R-08, R-11 | Không gán gì. | Ảnh phản chiếu và xe chủ. | Q |
| img_038 | Cửa hàng mô hình xe đồ chơi | R-08 | Không gán. | Đồ chơi. Ca đối chứng âm. | Q |
| img_024 | Xe vàng đặt trên nóc wagon trong bảo tàng xe | R-08, R-04 | Mọi xe trưng bày đều là xe thật → `car`. Xe trên nóc → `car`. | Xe thật, chỉ đang đỗ. | H — xe trưng bày trong nhà có thuộc mục tiêu "giao thông" không |
| img_019 | Xe diễu hành Dangermouse, linh vật chuột, đám đông sau rào | R-08, R-10 | Xe `car`. Người lái `person`. Linh vật không gán. Hàng người đầu tách được → `person`, phần đám đông phía sau → 1 `ignore_region`. | Xe chạy được là xe thật. Linh vật là vật trang trí. | H — linh vật có người bên trong không? |
| img_011 | Người mặc đồ Predator; xe máy trang trí kiểu điêu khắc | R-08 | Người hóa trang → `person`. Xe máy trang trí → `motorcycle`, tick `can_review`. | Người thật trong trang phục vẫn là người. Không chắc xe là xe chạy được hay tác phẩm. | H |
| img_021 | Dòng xe máy rất dày | R-02, R-10, R-07 | Xe và người ở hàng gần (tách được) vẽ box riêng. Khối phía xa → 1 `ignore_region`. | Vẽ từng box ở phía xa không nhất quán giữa hai người gán. | Q |
| img_030 | Bãi xe máy đỗ dày, chồng chéo | R-10 | Xe tiền cảnh thấy ≥ 50% → box riêng. Phần còn lại → `ignore_region`. Người mặc áo xanh → `person`. | Như trên. | Q |
| img_032 | Hàng người chờ xe buýt ở xa | R-07, R-10 | Người cao ≥ 20 px tách được → `person`. Cụm sát nhau không tách được → `ignore_region`. Xe `car` tiền cảnh `truncated = yes`. | Theo ngưỡng 20 px. | Q |
| img_014 | Người đi bộ rõ, xe máy phía sau mờ chuyển động | R-12 | 2 `person` rõ. Xe máy/người mờ nhận ra được → gán, `occluded_level = 3`. Mờ quá → `ignore_region`. | Vẫn là vật thật trong cảnh. | H — mức mờ nào thì bỏ |
| img_015 | Ô tô cắt hai mép, xe nền mờ | R-06, R-12 | Hai xe hai bên `truncated = yes`. Xe nền gán nếu cao ≥ 20 px. | Theo R-06. | Q |
| img_013, img_031 | Đường ướt ban đêm, phản chiếu trên mặt đường | R-08, R-07 | Chỉ gán xe/người thật. Phản chiếu không gán. Vật xa < 20 px bỏ qua hoặc gom `ignore_region`. | Phản chiếu không phải vật. | Q |

## 5. QC và phiên bản

- Ngưỡng QC giữ đúng `rules` của taxonomy: box cao tối thiểu 20 px; IoU ≥ 0.85 (vật ≥ 50 px) / ≥ 0.70 (vật nhỏ); IAA IoU 0.85, đồng thuận lớp 0.95; tỉ lệ lỗi ảnh ≤ 2%; ghép box ở IoU 0.5.
- Ca loại **H** ở mục 4: tạm tick `can_review`, đưa ra họp Pha 2. **Người điều phối (164) chốt** sau khi nghe cả nhóm, người review bản cuối (286) kiểm lại.
- Mọi thay đổi làm đổi nhãn (lớp, cách chọn vật, cách vẽ box, ngưỡng thuộc tính) → tăng phiên bản và ghi ca gây ra thay đổi.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản starter của giảng viên | — |
| v0.1-164 | Thêm định nghĩa lớp có ví dụ/phản ví dụ; ngưỡng % cho `occluded_level`; định nghĩa `truncated`; 13 quy tắc vẽ box (R-01…R-13); bảng ca khó | Pickup (img_001/012/037), người đi xe máy (img_005/026), vật không thật (img_017/027/033/038), phản chiếu (img_020), tuk-tuk (img_004/035), đám đông (img_019/021/030) |
