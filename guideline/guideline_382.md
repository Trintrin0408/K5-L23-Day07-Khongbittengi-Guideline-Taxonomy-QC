# Guideline gán nhãn — v0.1 — ảnh img_001 đến img_020

> Bản nháp cá nhân trong phạm vi được giao: chỉ gồm `img_001` đến `img_020`.

- Phiên bản: v0.1
- Ngày: 2026-11-16
- Người viết / duyệt: điền mã thành viên / nhóm sau
- Phạm vi rà soát: chỉ `img_001`–`img_020`
- Công cụ: CVAT, bounding box, xuất COCO 1.0

## 1. Mục tiêu

Gán nhãn các **người** và **phương tiện giao thông thực** nhìn thấy trong ảnh đường phố để huấn luyện mô hình phát hiện đối tượng. Mỗi vật thể thực được gán một box riêng; không gán nhãn cho chữ, biển quảng cáo, hình in, tượng, đồ trưng bày hoặc ảnh phản chiếu.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa | Ví dụ trong 20 ảnh | Không gán vào lớp này |
|---|---|---|---|
| `car` | Ô tô con, sedan, hatchback, SUV hoặc ô tô trang trí nhưng là phương tiện thật có bánh/chassis. | Ô tô trên xe chở xe ở `img_002`, ô tô ở `img_012`, `img_015`, `img_018`, xe diễu hành thật ở `img_019`. | Hình ô tô trên biển quảng cáo, mô hình hoặc xe chỉ là đạo cụ/trang trí không phải phương tiện thật. |
| `truck` | Xe tải có thùng/chở hàng, xe tải mui hở, xe bán tải có thùng hàng, xe chở xe. | Xe bán tải ở `img_001` và `img_012`, xe tải/xe chở ô tô ở `img_002`, `img_007`. | Ô tô con có cốp sau; hàng hóa hoặc thùng hàng tách rời không được gán box riêng. |
| `van` | Xe thân kín dùng chở người/hàng, gồm van và minibus cỡ nhỏ; không dùng cho xe buýt lớn. | Xe minibus/van ở `img_009`, `img_016`. | Xe buýt lớn; xe buýt lớn được loại theo quy tắc vì taxonomy không có lớp `bus`. |
| `motorcycle` | Xe máy, scooter, mô tô và phương tiện cơ giới hai hoặc ba bánh như tuk-tuk. | Xe máy/tuk-tuk ở `img_003`, `img_004`, `img_005`, `img_006`, `img_008`, `img_010`, `img_016`. | Xe đạp, xe máy trong tranh/biển quảng cáo, hoặc tác phẩm điêu khắc không phải phương tiện đang sử dụng. |
| `person` | Người thật nhìn thấy trong ảnh, gồm người đi bộ, lái xe, người ngồi sau xe và người trong xe. | Người đi bộ ở `img_001`, `img_003`, `img_014`; người lái/ngồi trên xe ở `img_005`, `img_006`, `img_010`. | Người trong ảnh quảng cáo, tượng, hình nộm, mascot không thấy người thật bên trong. |
| `ignore_region` | Một vùng chứa vật ngoài phạm vi hoặc cụm quá mơ hồ khiến không thể gán riêng từng vật. | Có thể dùng quanh khu trưng bày tượng ở `img_011` hoặc tượng kỵ sĩ ở `img_017` nếu reviewer yêu cầu. | Không dùng để phủ nền, đường, bầu trời hoặc toàn bộ ảnh; vật rõ ràng vẫn phải gán nhãn riêng. |

### Thuộc tính

| Thuộc tính | Giá trị | Quy tắc |
|---|---|---|
| `occluded_level` | `0` | Vật gần như không bị che, phần nhìn thấy từ 90% trở lên. |
| `occluded_level` | `1` | Bị che nhẹ, khoảng 70–89% vật vẫn nhìn thấy. |
| `occluded_level` | `2` | Bị che đáng kể, khoảng 30–69% vật nhìn thấy nhưng vẫn nhận dạng được. |
| `occluded_level` | `3` | Chỉ còn dưới 30% vật nhìn thấy nhưng đủ dấu hiệu để xác định đúng lớp; nếu không đủ chắc thì không gán hoặc bật `can_review`. |
| `truncated` | `no` | Vật không bị cắt bởi biên ảnh. Bị che bởi vật khác không phải là truncated. |
| `truncated` | `yes` | Box chạm/cắt biên trái, phải, trên hoặc dưới ảnh; ví dụ ô tô ở hai mép `img_015`. |
| `can_review` | bật | Chỉ bật khi còn nghi ngờ về lớp, biên box, vật thật hay tượng/đạo cụ, hoặc vật bị mờ/che đến mức cần reviewer. |

Ngưỡng tối thiểu theo taxonomy: không tạo box có chiều cao nhỏ hơn **20 px**. Các vật quá nhỏ ở xa được bỏ qua; không phóng đoán box từ nền hoặc bóng.

## 3. Quy tắc vẽ box

1. Vẽ một box cho mỗi vật thể thực, ôm sát **phần nhìn thấy** của vật.
2. Không mở rộng box vào phần bị che khuất hoặc phần không nhìn thấy. Hai vật chồng lên nhau vẫn phải có hai box riêng nếu nhận dạng được.
3. Người và phương tiện là hai đối tượng độc lập: người lái xe máy được gán `person`, xe máy được gán `motorcycle`.
4. Xe chở xe: gán một box `truck` cho xe chuyên chở và một box `car` cho từng ô tô được chở, nếu từng ô tô nhìn thấy đủ rõ (`img_002`).
5. Hàng hóa, rau củ, thùng carton, biển báo, cọc tiêu, dây điện và vật dụng trong/ngoài xe không thuộc taxonomy thì bỏ qua.
6. Xe đạp không có lớp trong taxonomy nên bỏ qua; không gán nhầm sang `motorcycle`.
7. Vật bị cắt ở biên ảnh vẫn gán box phần nhìn thấy và đặt `truncated=yes`.
8. Vật bị nhòe do chuyển động vẫn gán nếu nhận dạng được lớp và box tối thiểu 20 px; nếu không chắc thì bật `can_review`.
9. Không gán vật chỉ xuất hiện trong gương, mặt kính phản chiếu, màn hình, biển quảng cáo hoặc ảnh in. Không nhân đôi một vật thật và ảnh phản chiếu của nó.
10. Tượng, mô hình, đồ chơi, tác phẩm nghệ thuật và mascot không phải người thật/xe thật thì không gán vào `person` hoặc lớp phương tiện. Nếu vùng đó gây nhầm lẫn cho reviewer, dùng `ignore_region` hoặc `can_review` theo quyết định của nhóm.
11. Tuk-tuk là phương tiện cơ giới ba bánh nên gán `motorcycle`, một box cho mỗi tuk-tuk. Xe bán tải có thùng chở hàng, kể cả có phủ bạt hoặc gắn khoang kín, vẫn gán `truck`.
12. Minibus thân kín cỡ nhỏ gán `van`. Xe buýt lớn không có lớp tương ứng trong taxonomy nên loại theo quy tắc; không tự tạo lớp `bus`.

## 4. Ca khó và quyết định tham chiếu

| Ảnh | Tình huống | Rule ID | Quyết định |
|---|---|---|---|
| `img_001` | Xe bán tải có thùng hở, nhiều người phía sau. | R-01 | Xe bán tải là `truck`; gán riêng từng `person` nhìn thấy đủ rõ. Không gán biển quảng cáo. |
| `img_002` | Xe chở nhiều ô tô. | R-02 | Gán `truck` cho xe chở và `car` cho từng ô tô trên xe; người đứng bên phải gán `person` nếu đạt 20 px. |
| `img_003` | Nhiều xe máy, xe đạp và người bị chồng lấp. | R-03 | Xe máy và người gán riêng; xe đạp bỏ qua; dùng `occluded_level` theo phần bị che. |
| `img_004` | Hai tuk-tuk ba bánh. | R-04 | Mỗi tuk-tuk là một `motorcycle`; gán người lái nếu nhìn thấy đủ rõ. Đây là ca ánh xạ lớp theo quy tắc, không cần reviewer nếu tuk-tuk nhận dạng rõ. |
| `img_005` | Nhiều người ngồi trên xe máy, có xe van phía sau. | R-05 | Mỗi người và mỗi xe máy là một box riêng; xe thân kín phía sau là `van`. |
| `img_006` | Đám đông xe máy và xe đạp. | R-06 | Gán các `motorcycle` và `person` nhận dạng được; xe đạp bỏ qua; không gộp cả đám đông thành một box. |
| `img_007` | Xe bán tải và xe tải thùng trong không gian hẹp. | R-07 | Xe bán tải có thùng hàng và xe tải thùng đều là `truck`; box theo phần nhìn thấy. |
| `img_008` | Nhiều xe máy/ba bánh trong cảnh đêm, che khuất mạnh. | R-08 | Gán từng phương tiện/người đủ rõ; đặt `occluded_level=2/3` khi phù hợp và bật `can_review` nếu không chắc. |
| `img_009` | Xe minibus/van trên đường. | R-09 | Xe thân kín cỡ nhỏ là `van`; không tạo lớp `bus` mới. Đây là ca ánh xạ lớp theo quy tắc. |
| `img_010` | Xe máy dừng bên người bán hàng. | R-10 | Gán từng xe máy và người nhìn thấy; hàng hóa/xe đạp không gán. |
| `img_011` | Khu trưng bày có tượng lớn và tác phẩm xe bằng kim loại. | R-11 | Loại tượng và xe điêu khắc theo quy tắc; không gán `person`/`motorcycle`. Người thật phía xa chỉ gán nếu đạt 20 px. Không cần reviewer nếu xác định rõ là tác phẩm trưng bày. |
| `img_012` | Pickup chở nhiều thùng/hàng và ô tô bên trái. | R-12 | Pickup là `truck`, ô tô bên trái là `car`; không gán thùng, túi hoặc hàng hóa. |
| `img_013` | Cảnh đường ban đêm, người/xe ở xa và nhỏ. | R-13 | Chỉ gán vật đạt 20 px và nhận dạng được; vật quá nhỏ bỏ qua. |
| `img_014` | Người đứng rõ, xe/người đi ngang bị nhòe chuyển động. | R-14 | Gán vật nhòe nếu nhận dạng được; box theo biên nhìn thấy; bật `can_review` khi lớp hoặc biên không chắc. |
| `img_015` | Ô tô bị cắt ở mép trái/phải ảnh. | R-15 | Gán phần nhìn thấy của từng `car`, đặt `truncated=yes`; xe không bị cắt đặt `no`. |
| `img_016` | Tuk-tuk, scooter, van và người đi đường. | R-16 | Tuk-tuk/scooter là `motorcycle`; van là `van`; người là `person`; gán độc lập. |
| `img_017` | Tượng kỵ sĩ cưỡi ngựa, không có người thật. | R-17 | Loại theo quy tắc: không gán tượng thành `person`; không có lớp `horse`. Có thể dùng `ignore_region` cho vùng tượng nếu cần. |
| `img_018` | Người trên billboard và ô tô thật ở phía dưới. | R-18 | Người trên quảng cáo không gán; chỉ gán ô tô thật nhìn thấy đạt ngưỡng. |
| `img_019` | Xe trang trí trong đoàn diễu hành và người thật. | R-19 | Nếu là xe thật có bánh/chassis và đang chạy, gán `car`; người thật gán `person`; mascot/đạo cụ không gán. Bật `can_review` nếu không xác định được xe thật hay mô hình. |
| `img_020` | Xe/người chỉ xuất hiện trong gương chiếu hậu. | R-20 | Loại theo quy tắc: không gán vật trong phản chiếu; không nhân đôi đối tượng. Chỉ gán vật thật bên ngoài gương nếu có và đạt ngưỡng. |

## 5. QC và phiên bản

- Trước khi gửi nhãn, kiểm tra mọi box có đúng một lớp, ôm sát vật và có đủ thuộc tính.
- Kiểm tra riêng các nhóm ca: xe chở xe (`img_002`), xe ba bánh (`img_004`, `img_016`), tượng/tác phẩm (`img_011`, `img_017`), vật bị nhòe (`img_014`), cắt biên (`img_015`) và phản chiếu (`img_020`).
- Nếu hai người không thống nhất, giữ box nhưng bật `can_review` và ghi lại ảnh/vùng trong nhật ký QC.
- **Ca loại theo quy tắc trong phạm vi:** `img_011`, `img_017`, `img_020` — tượng, tác phẩm hoặc phản chiếu; không tạo box đối tượng và không bật `can_review` nếu bản chất đã rõ.
- **Ca ánh xạ lớp theo quy tắc trong phạm vi:** `img_004`, `img_009`, `img_016` — tuk-tuk → `motorcycle`, minibus/van → `van`.
- **Chỉ hỏi reviewer** khi sau khi áp dụng các quy tắc trên vẫn không xác định được vật thật, lớp, số box hoặc phần nhìn thấy; không dùng reviewer để thay thế một quy tắc đã chốt.
- Khi thay đổi quyết định về lớp, số box hoặc phạm vi box, tăng phiên bản guideline và ghi rõ Rule ID bị ảnh hưởng.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0.1 | Bổ sung quy tắc cho ảnh `img_001`–`img_020` | Làm rõ xe chở xe, xe ba bánh, minibus, tượng, phản chiếu, xe cắt biên và vật nhòe |
