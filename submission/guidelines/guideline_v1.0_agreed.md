# Guideline gán nhãn — v1.0 (thống nhất nhóm) — Khongbittengi

- Phiên bản: **v1.0** (khóa cho lượt gán độc lập Pha 3)
- Ngày: 2026-11-16
- Nguồn tổng hợp: `guideline_154.md`, `guideline_164.md`, `guideline_169.md`, `guideline_286.md` (bản 286 trùng nội dung với bản 164)
- Người điều phối / chốt: `164` · Người review bản cuối: `286` · Thành viên: `154`, `169`
- Loại nhãn: bounding box trên ảnh 2D, CVAT, xuất COCO 1.0. Lớp và thuộc tính giữ đúng `taxonomy.json`; mục `rules` (ngưỡng) **không đổi**.

> Mọi quyết định trong bản này dẫn bằng mã `R-xx`. Khi ghi `can_review`, `image_issues.csv` hay `calibration_decisions.csv`, ghi kèm mã quy tắc.

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người và phương tiện thật** trên ảnh đường phố (camera giao thông, hỗ trợ lái / xe tự lái).

Lỗi nguy hiểm, xếp theo mức độ:

1. **Bỏ sót người thật** (người đi bộ, người lái/ngồi trên xe máy): mô hình học thói quen "không thấy" người.
2. **Gán vật không thật thành vật thật** (tượng, ma-nơ-canh, hình in, đồ chơi, mô hình, phản chiếu): mô hình báo động giả, phanh nhầm.
3. **Lẫn lớp giữa các phương tiện** (pickup ↔ car, van ↔ car): ít nguy hiểm hơn nhưng làm giảm đồng thuận lớp.

Nguyên tắc nền: **chỉ gán thứ nhìn thấy được trên ảnh 2D**. Phần bị che, phần ngoài khung không đoán thêm. Khi đã áp quy tắc mà vẫn còn hai cách hiểu thì tick `can_review` (R-18).

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Xe 4 bánh chở người, thân kín liền, tối đa ~9 chỗ: sedan, hatchback, SUV, wagon, taxi. | img_023 (sedan đen), img_024 (Pacer đen), img_031, img_034 (sedan đen), img_040 (taxi) | Pickup → `truck` (R-06). Xe thân hộp → `van`. Xe đồ chơi/mô hình img_038, thân xe rỗng img_024 → không gán (R-08). |
| `truck` | Xe chủ yếu chở hàng, **cabin tách khỏi thùng hàng**: xe tải, xe đầu kéo, xe chở ô tô, **pickup** (kể cả có nắp thùng/mui). | img_002 (xe chở ô tô), img_007 (Isuzu), img_001, img_012, img_028, img_036, img_037 | Tượng pickup bằng đá img_027 → không gán (R-08). Xe buýt/minibus → ngoài taxonomy (R-09). |
| `van` | Xe 4 bánh thân liền dạng hộp, cabin và khoang sau không tách, mũi ngắn (minivan, Hiace, xe van chở hàng). | img_005 (xe biển 51LD), img_016 (xe trắng sau xe máy), img_033 (mảnh xe trắng, `can_review`) | Minibus/Coaster có cửa xe khách, nhiều hàng ghế (img_009) → ngoài taxonomy. SUV → `car`. Pickup có mui (img_037) → `truck`. |
| `motorcycle` | Xe **2 bánh** có động cơ: xe máy, xe tay ga, mô tô; mô tô có thùng bên (sidecar) tính **một** xe. | img_006, img_008, img_010, img_021, img_026, img_029, img_022 (sidecar) | Xe đạp, xích lô (img_023, img_039) và tuk-tuk/xe lam 3 bánh (img_004, img_035) → ngoài taxonomy (R-09). |
| `person` | Người thật nhìn thấy trên ảnh: đi, đứng, ngồi, hoặc đang lái/ngồi trên **xe hở** bất kỳ (kể cả xe ngoài taxonomy). | img_014, img_034 (đi bộ), img_026 (2 người trên xe), img_023 (người đạp xích lô), img_011 (người hóa trang) | Tượng img_017, ma-nơ-canh img_033, người in trên biển img_018, áp phích/tranh tường img_026 → không gán (R-08). Người trong cabin kín → không gán (R-14). |
| `ignore_region` | Vùng chứa **nhiều vật thuộc taxonomy** không vẽ được từng box (quá dày, quá nhỏ, quá mờ); bỏ sót trong vùng không tính lỗi. | Rừng xe máy xa img_021, bãi xe dày img_030, cụm người chờ buýt img_032, đám đông sau rào img_019 | **Không** dùng cho vật ngoài taxonomy (buýt, tuk-tuk…) và **không** dùng cho vật giả. |

**Vật không thuộc lớp nào** (xe buýt, minibus, tuk-tuk/xe lam, xích lô, xe đạp, xe ba gác, động vật, đồ vật): **không vẽ box, không dùng `ignore_region`**. Người đang ngồi/lái trên các xe này vẫn gán `person` (R-09).
Đề xuất cho v1.1 (chưa áp dụng, cần cả nhóm + coach đồng ý vì đổi taxonomy): thêm lớp `bus` và `three_wheeler` nếu calibration cho thấy hai loại này gây nhiều bất đồng.

### Thuộc tính

**`occluded_level`** — tỉ lệ diện tích phần vật **nằm trong khung ảnh** bị vật khác che (ước bằng mắt).

| Mức | Nghĩa | Ngưỡng |
|---|---|---|
| `0` | Thấy rõ | bị che < 10% |
| `1` | Che ít | 10% đến < 50% |
| `2` | Che nhiều | ≥ 50%, vẫn nhận ra được lớp |
| `3` | Không ước lượng được | quá tối, quá mờ, hoặc chỉ thấy qua kính/lưới nên không đoán được phần bị che |

- Phần nằm **ngoài mép ảnh không tính** vào `occluded_level` (đã có `truncated`).
- Người lái / người ngồi trên xe **không tính** là che chiếc xe của chính họ; bánh xe chạm mặt đường không tính là che. Vật khác (xe bên cạnh, cột, người đi qua) thì tính.
- Hàng rào song thưa mà vẫn thấy rõ đường viền xe (img_018) → mức `1`.

**`truncated`** — `yes` khi box **chạm mép ảnh và** thấy rõ vật còn tiếp tục ra ngoài khung (phần mất khoảng ≥ 5% vật). Vật chỉ chạm mép mà đã thấy hết → `no`. Chỉ xét mép ảnh, không xét bị che.

**`can_review`** — tick khi đã áp hết quy tắc mà vẫn còn hai cách hiểu (lớp mơ hồ, không chắc thật/giả). Không tick chỉ vì box khó vẽ. Xem R-18.

## 3. Quy tắc vẽ box (mã quy tắc trọng tâm)

| Mã | Chủ đề | Quy tắc |
|---|---|---|
| **R-01** | Phần nhìn thấy | Box ôm sát **phần nhìn thấy** (lệch ≤ 2–3 px mỗi cạnh), không amodal, không kéo dài ra phần bị che/ngoài khung. Một vật bị cột/vật khác chia đôi vẫn là **một box** bao cả hai phần. Mục tiêu IoU ≥ 0.85 (vật cao ≥ 50 px), ≥ 0.70 (vật nhỏ hơn). |
| **R-02** | Người đi xe máy | **Hai loại box riêng, được chồng nhau**: mỗi người (lái, ngồi sau, trẻ em, người trong sidecar) là một `person`, bao từ đầu (kể cả mũ bảo hiểm) tới phần chân thấy được; chiếc xe là một `motorcycle` bao bánh trước → bánh/đuôi sau, tay lái, gương (và sidecar nếu có). Box xe **không** kéo lên để bao đầu/thân trên người. Người ngồi không tính là che xe. |
| **R-03** | Hàng hóa, vật kèm theo | Hàng **nhô ra ngoài thân xe/người** (ống dài, gánh hàng, bao chất cao, xe kéo thùng phía sau, tượng trên nóc/nhô cao khỏi thùng) **không** mở rộng box. Hàng nằm **trong** thùng pickup/xe tải thì thuộc box xe. |
| **R-04** | Vật bị che | Vẫn gán nếu nhận ra lớp; vẽ phần thấy được (R-01), đặt `occluded_level` theo bảng ở mục 2. |
| **R-05** | Cắt mép ảnh | Vẽ tới sát mép, `truncated = yes`. Vật chỉ còn một phần nhưng **nhận ra được lớp** vẫn gán. Không nhận ra lớp → R-16. |
| **R-06** | Ranh giới car / truck / van | Pickup (kể cả có nắp/mui thùng, đang chở đồ) = `truck` vì cabin tách thùng hàng. SUV = `car`. Thân hộp liền, mũi ngắn = `van`. Minibus/Coaster cửa kiểu xe khách = ngoài taxonomy (R-09). |
| **R-07** | Vật nhỏ | Box cao **< 20 px** (`min_box_height_px`) thì không vẽ box riêng. Vật lẻ → bỏ qua. Nhiều vật nhỏ thuộc taxonomy tụ thành cụm → một `ignore_region`. Đo theo **chiều cao** box. |
| **R-08** | Vật không thật | Tượng, ma-nơ-canh, đồ chơi, mô hình, thân xe rỗng trưng bày, người/xe trên biển quảng cáo, áp phích, tranh tường, **phản chiếu** trên gương, kính, mặt đường ướt, vũng nước → **không vẽ gì** (mẫu âm). Vật giả nằm trên vật thật thì box vật thật không ôm vật giả (R-03). |
| **R-09** | Ngoài taxonomy | Xe buýt, minibus, tuk-tuk/xe lam, xích lô, xe đạp, xe ba gác: **không box, không `ignore_region`**. Người trên các xe này (xe hở) vẫn gán `person`. |
| **R-10** | Đám đông / khối dày | Vật nào **tách được ranh giới**, thấy rõ ≥ 50% thân và cao ≥ 20 px → box riêng. Phần còn lại dính nhau → **một** `ignore_region` ôm cả cụm. `ignore_region` không trùm lên box riêng đã vẽ. |
| **R-11** | Xe chủ (ego) | Capô, kính, khung gương, cửa của xe chụp ảnh → không gán. |
| **R-12** | Mờ, tối | Nhận ra lớp và ước được biên → gán, bao phần đặc nhất, `occluded_level = 3`. Không nhận ra lớp: nằm trong cụm → `ignore_region`; vật lẻ → không gán, ghi `image_issues.csv`. |
| **R-13** | Xe chở xe | Xe chở là một `truck` bao cabin + giá chở. Mỗi ô tô trên giá là xe thật → một `car` riêng, `occluded_level` theo phần bị khung che. |
| **R-14** | Người trong xe | Người trên xe hở (xe máy, xe đạp, xích lô, tuk-tuk hở) → `person`. Người trong cabin kín, chỉ thấy qua kính (ô tô, cabin xe tải) → **không gán**. |
| **R-15** | Xe đỗ / trưng bày | Xe thật đang đỗ, trưng bày trong bảo tàng/showroom, xe diễu hành chạy được → gán bình thường theo lớp (vẫn áp R-07, R-10). |
| **R-16** | Mảnh vật quá nhỏ | Chỉ thấy gương, tay nắm, một mảnh không nhận ra lớp → không vẽ box, ghi ảnh + vùng vào `image_issues.csv`. |
| **R-17** | Ảnh 0 box | Ảnh không có vật hợp lệ vẫn giữ trong COCO với 0 annotation (img_027, img_038). |
| **R-18** | Dùng `can_review` | Tick khi còn hai cách hiểu sau khi áp quy tắc (thật/giả, lớp mơ hồ). Ca cần hỏi nhưng **không có box** để tick → ghi `image_issues.csv`. Không tick cho ca đã có quy tắc rõ. |

## 4. Bất đồng giữa các bản cá nhân và quyết định

| # | Điểm bất đồng | 154 | 164 / 286 | 169 | Chốt v1.0 | Lý do |
|---|---|---|---|---|---|---|
| 1 | Pickup | `truck` | `truck` | `car` (theo v0) | `truck` (R-06) | 3/4 bản đồng ý; cabin tách thùng hàng là dấu hiệu nhìn thấy được, dễ đồng thuận hơn "pickup thông thường". |
| 2 | Tuk-tuk / xe lam | ngoài lớp | ngoài lớp | `motorcycle` (theo v0) | Ngoài taxonomy (R-09) | Lớp `motorcycle` là xe 2 bánh; gộp 3 bánh làm lớp mơ hồ. |
| 3 | Xe buýt, tuk-tuk, xích lô, xe đạp | `ignore_region` | không gán | không gán | **Không gán** (R-09) | `ignore_region` theo taxonomy là vùng chứa vật *trong* taxonomy không tách được; dùng nó cho vật ngoài lớp làm hai người vẽ vùng ignore khác nhau, giảm IAA. |
| 4 | Ngưỡng `occluded_level` | 10/40/75%, mức 3 = che > 75% | 10/50%, mức 3 = không ước lượng được | chỉ định tính | Bảng 164 | Giữ đúng nghĩa v0 ("3 là không rõ"); 3 bậc % khó ước bằng mắt hơn 2 ngưỡng. |
| 5 | `truncated` | có phần ngoài mép ≥ ~5% | chạm mép + rõ còn tiếp tục | chạm mép | Gộp 154 + 164 | Thêm ngưỡng ~5% để vật chạm mép rất ít không bị đánh `yes`. |
| 6 | Tách vật trong đám đông | thấy ≥ 30% | thấy ≥ 50% và ≥ 20 px | chưa có ngưỡng | ≥ 50% + ≥ 20 px + tách được ranh giới (R-10) | Vật trong `ignore_region` không tính lỗi nên không mất người; ngưỡng 50% giúp hai người chọn cùng vật. |
| 7 | Vật chỉ còn một phần ở mép | < 15% thân → không box | < 20% nhưng nhận ra lớp → vẫn gán | — | Nhận ra lớp → gán (R-05); không nhận ra → R-16 | Ưu tiên không bỏ sót; tiêu chí "nhận ra lớp" dễ áp hơn ước % thân xe ngoài khung. |
| 8 | Người trong ô tô | không gán | — | gán nếu thấy rõ | Không gán (R-14) | Nhìn qua kính, biên mơ hồ; không phải người tham gia giao thông độc lập. |
| 9 | Xe vàng trên nóc img_024 | không gán | `car` (H) | — | **Không gán** (R-08) | Xem ảnh: là thân xe rỗng (không kính, không nội thất) đặt trên giá nóc — vật trưng bày, không phải xe hoạt động. |
| 10 | Mảnh xe trắng img_033 | `van`, `can_review` | không gán gì | — | `van`, `truncated = yes`, `can_review` | Thấy rõ gương + bản lề cửa của xe thật nhưng lẫn phản chiếu kính; gán để calibration có box mà chốt. |
| 11 | Cụm vật nhỏ < 20 px | bỏ qua, không ignore | cụm → `ignore_region` | cụm → có thể ignore | Lẻ bỏ qua, cụm → `ignore_region` (R-07) | Đúng nghĩa `ignore_region`; tránh phạt mô hình khi phát hiện đúng vật xa. |
| 12 | Box xe máy và người | xe không bao thân trên người | phần người đè lên xe vẫn nằm trong box xe | box riêng | R-02 | Hai cách thực chất giống nhau; ghi rõ giới hạn box xe (bánh, tay lái, gương) để một cách hiểu. |

## 5. Ca khó (40 ảnh)

Loại **Q** = áp quy tắc là quyết được. Loại **H** = còn mơ hồ, tick `can_review` / ghi `image_issues.csv`, đưa vào calibration.

| Ảnh | Tình huống | Rule ID | Quyết định | Loại |
|---|---|---|---|---|
| img_001, img_012 | Pickup, img_012 chở đồ trong thùng | R-06, R-03 | `truck`; hàng trong thùng thuộc box xe | Q |
| img_002 | Xe đầu kéo chở 8 ô tô | R-13, R-04 | 1 `truck` + mỗi ô tô 1 `car`, tầng dưới `occluded_level` 1–2 | Q |
| img_003 | Người gánh hàng | R-03 | `person`; gánh hàng không nằm trong box | Q |
| img_004, img_016, img_035 | Tuk-tuk / xe lam | R-09, R-14, R-03 | Xe không gán; người lái/ngồi `person`; xe máy, van, car xung quanh gán; bao hàng trên tuk-tuk không gán | Q |
| img_005 | Nhiều người trên xe máy chen sát, xe máy gần như bị che, van phía sau | R-02, R-04 | Mỗi người 1 `person`; xe máy nhận ra được 1 `motorcycle` (`occluded_level` 2); `van` 1–2 | Q |
| img_006 | Xe máy, xích lô | R-02, R-09 | Xe máy + người theo R-02; xích lô không gán, người đạp `person` | Q |
| img_007 | Xe tải Isuzu | R-06 | `truck` | Q |
| img_008, img_010 | Xe máy có người | R-02 | `person` + `motorcycle` | Q |
| img_009 | Minibus Coaster | R-06, R-09 | Không gán | H — ranh giới van/minibus, xác nhận ở calibration |
| img_011 | Người mặc đồ Predator; xe máy trang trí kiểu điêu khắc | R-08, R-18 | Người hóa trang `person`; xe `motorcycle` + `can_review` | H — xe chạy được hay tác phẩm |
| img_013, img_031 | Đường ướt ban đêm, phản chiếu | R-08, R-07, R-02 | Chỉ gán xe/người thật; phản chiếu không gán; xe buýt img_031 không gán (R-09); vật xa < 20 px theo R-07 | Q |
| img_014 | Người đi bộ rõ, xe máy phía sau mờ | R-12 | 2 `person` rõ; xe/người mờ nhận ra lớp → gán, `occluded_level = 3`; không nhận ra → R-12 | H — mức mờ |
| img_015 | Ô tô cắt hai mép, xe nền | R-05, R-07 | Hai xe hai bên `truncated = yes`; xe nền gán nếu ≥ 20 px | Q |
| img_017 | Tượng người cưỡi ngựa | R-08 | Không gán | Q |
| img_018 | Biển quảng cáo có người; ô tô sau hàng rào | R-08, R-04 | Người in không gán; mỗi ô tô `car`, `occluded_level` 1 (rào thưa) hoặc 2 | Q |
| img_019 | Xe diễu hành, linh vật chuột, đám đông sau rào | R-15, R-08, R-10 | Xe `car`; người lái `person`; linh vật gắn trên xe không gán; hàng người đầu tách được `person`, phía sau 1 `ignore_region` | H — linh vật có người bên trong không (ghi `image_issues.csv`) |
| img_020 | Ô tô trong gương chiếu hậu, khung gương xe chủ | R-08, R-11 | Không gán gì | Q |
| img_021 | Rừng xe máy giờ cao điểm | R-02, R-10 | Xe + người hàng đầu tách được gán riêng; khối phía sau 1 `ignore_region` | Q |
| img_022 | Mô tô sidecar, ảnh đen trắng | R-02, R-07 | 1 `motorcycle` bao cả sidecar; người lái + người trong sidecar `person` nếu cao ≥ 20 px | Q |
| img_023 | Xích lô, sedan đen, xe máy xa, người mép phải | R-09, R-14, R-02, R-05 | Xích lô không gán, người đạp `person`; sedan `car`; xe máy xa + người gán nếu ≥ 20 px; người mép phải `truncated = yes` | Q |
| img_024 | Bảo tàng xe; thân xe vàng rỗng trên nóc Pacer đen | R-15, R-08, R-03 | Pacer đen, xe cam, xe kem và các xe nền ≥ 20 px → `car`; thân xe vàng không gán; box Pacer không ôm xe vàng | Q |
| img_025 | Đường bờ biển, người ngồi ghế xa, xe máy kéo thùng | R-07, R-02, R-03 | Người ≥ 20 px `person`; xe máy `motorcycle` + `person`, thùng kéo không nằm trong box | Q |
| img_026 | 2 người trên xe ga, áp phích tàu biển phía sau | R-02, R-05, R-08 | 2 `person` + 1 `motorcycle` (`truncated = yes`, bánh bị cắt đáy); áp phích không gán | Q |
| img_027 | Tượng pickup bằng đá granite | R-08, R-17 | Không gán; ảnh giữ trong COCO | Q |
| img_028 | Pickup thật chở tượng ngựa, SUV đỏ phía sau | R-06, R-08, R-03 | Pickup `truck`, box không mở rộng theo phần tượng nhô cao; tượng không gán; SUV đỏ `car` | Q |
| img_029 | Xe ga chở ống dài; người dắt xe đạp | R-02, R-03, R-09 | `motorcycle` + `person`; ống không nằm trong box; xe đạp không gán, người `person` | Q |
| img_030 | Bãi xe máy dày, bảo vệ đứng giữa | R-10 | Bảo vệ `person`; xe tiền cảnh thấy ≥ 50% gán riêng; phần còn lại 1 `ignore_region` | Q |
| img_032 | ~30 người chờ buýt; xe xanh cắt mép trái; nón cảnh báo | R-10, R-07, R-05 | Người tách được ≥ 20 px `person`; cụm dính `ignore_region`; xe xanh `car`, `truncated = yes`; nón không gán | Q |
| img_033 | Ma-nơ-canh, hình in phụ nữ, phản chiếu kính, mảnh xe trắng có gương | R-08, R-05, R-18 | Ma-nơ-canh, hình in, phản chiếu không gán; xe trắng `van`, `truncated = yes`, `can_review` | H — xe thật hay phản chiếu |
| img_034 | Người đi bộ, sedan đen, xe máy xa, người trên xe máy mép phải | R-02, R-05, R-07 | Người đi bộ `person`; sedan `car`; xe máy mép phải `person` + `motorcycle`, `truncated = yes`; xe máy xa < 20 px không gán | Q |
| img_036 | Xe tải nhỏ mui bạt, dãy xe máy đỗ | R-06, R-10 | `truck`; xe máy đỗ tách được gán, dãy dính `ignore_region` | Q |
| img_037 | Pickup có mui, hai xe buýt, người đi bộ | R-06, R-09 | Pickup `truck`; buýt không gán; người `person` | Q |
| img_038 | Tủ kính xe mô hình, vịt cao su | R-08, R-17 | Không gán; giữ ảnh với 0 box (đối chứng âm) | Q |
| img_039 | Phố Huế: xe máy đỗ, người đi bộ, người đạp xe, xích lô mép phải | R-07, R-09, R-14 | Người và xe máy ≥ 20 px gán; xe đạp, xích lô không gán, người trên đó `person` | Q |
| img_040 | Người đội mũ bảo hiểm cận cảnh, taxi bị che, tuk-tuk, buýt, xe máy chỉ thấy gương | R-05, R-04, R-09, R-16 | Người `person` (`truncated = yes`); taxi `car`, `occluded_level = 1`, `truncated = yes`; tuk-tuk, buýt không gán; xe máy chỉ thấy gương không box, ghi `image_issues.csv` | Q |

## 6. QC và phiên bản

- Ngưỡng QC lấy nguyên `taxonomy.json` → `rules`: box cao tối thiểu 20 px; IoU ≥ 0.85 (vật ≥ 50 px) / ≥ 0.70 (vật nhỏ); IAA IoU 0.85, đồng thuận lớp 0.95; lỗi theo ảnh ≤ 2%; ghép box ở IoU 0.5. **Không sửa.**
- Ca loại **H**: lúc gán tick `can_review` hoặc ghi `image_issues.csv` kèm mã quy tắc; ở calibration **người điều phối `164` chốt** sau khi nghe nhóm, **`286` kiểm lại** bản cuối.
- v1.0 **khóa** trong Pha 3. Mọi thay đổi làm đổi nhãn (lớp, cách chọn vật, cách vẽ box, ngưỡng thuộc tính) → tăng phiên bản (v1.1…), ghi mã quy tắc và ca gây ra thay đổi.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản starter của giảng viên: 6 lớp, 3 thuộc tính, chưa có ngưỡng và quy tắc ca biên. Pickup ở `car`, tuk-tuk ở `motorcycle`. | — |
| v0.1 (cá nhân) | 4 bản độc lập: `154` (R-01…R-15, img_021–040), `164` (R-01…R-13, ngưỡng `occluded_level`/`truncated`), `169` (quy tắc chung, img_002/005/013), `286` (trùng `164`). | Pickup (img_001/012/037), người đi xe máy (img_005/026), vật giả (img_017/027/033/038), phản chiếu (img_020/031), tuk-tuk (img_004/035), đám đông (img_021/030/032) |
| **v1.0** | Thống nhất R-01…R-18. Pickup → `truck` (R-06); tuk-tuk, xích lô, xe đạp, buýt, minibus → không gán, không `ignore_region` (R-09); ngưỡng `occluded_level` 10%/50%, mức 3 = không ước lượng được; `truncated` cần chạm mép + mất ≥ ~5%; tách đám đông ≥ 50% thân (R-10); cụm vật < 20 px → `ignore_region` (R-07); người trong cabin kín không gán (R-14); thân xe vàng img_024 không gán (R-08); mảnh xe img_033 → `van` + `can_review`. | Giải quyết 12 bất đồng ở mục 4 (img_009, 024, 033, 035, 037, 040, 021, 030, 032) |
