# Guideline gán nhãn — v0.1 (bản khởi đầu)

> **Đây là bản do giảng viên viết nhanh. Nó CHƯA đủ dùng.**
> Đừng sửa file này. Hãy copy sang `submission/guidelines/individual/` rồi sửa bản của bạn
> (xem `GUIDE.md`, Pha 1).

- Phiên bản: v0.1
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình phát hiện **người và phương tiện** trên ảnh đường phố
(ví dụ cho camera giao thông hoặc xe tự lái).

## 2. Lớp và thuộc tính

### Lớp (mỗi vật thuộc đúng một lớp)

| Lớp | Định nghĩa |
|---|---|
| `car` | Ô tô con, sedan, hatchback, SUV, taxi và pickup thông thường. |
| `truck` | Xe tải chở hàng, xe tải thùng, xe tải nhẹ và xe vận chuyển ô tô. |
| `van` | xe van, minivan, xe chở khách hoặc hàng hóa có thân dạng van. |
| `motorcycle` | Xe máy, scooter, mô tô và xe ba bánh cơ giới dạng tuk-tuk. |
| `person` | 

Người thật, bao gồm người đi bộ, người lái xe, người ngồi trên phương tiện. |
| `ignore_region` |Vùng chứa đối tượng không thể phân tách hoặc nhận diện đáng tin cậy. |

### Thuộc tính

| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | Mức bị che: 0 là thấy rõ, 1 là bị che ít, 2 là bị che nhiều, 3 là không rõ. |
| `truncated` | `no`, `yes` | Vật bị cắt ở mép ảnh. |
| `can_review` | tick / không | Tick khi bạn không chắc, để người khác xem lại. |

## 3. Quy tắc vẽ box

## 3.1. Quy tắc chung

- Vẽ box ôm sát phần vật thể nhìn thấy.
- Không bao gồm bóng đổ, phản chiếu hoặc vệt chuyển động.
- Không vẽ box vượt ngoài ảnh.
- Không gộp nhiều đối tượng cùng lớp vào một box. 
- Vật thể bị che khuất vẫn được gán nếu nhận diện được.
- Nếu chỉ nhìn thấy một phần quá nhỏ và không thể xác định lớp, không tự suy đoán.

## 3.2. Người và phương tiện
- Người lái xe: box person. 
- Người ngồi sau: box person riêng. 
- Xe máy: box motorcycle riêng.
- Người đi xe đạp: chỉ gán person.
- Người trên xe xích lô: gán person, không gán phương tiện nếu xe không thuộc taxonomy.
- Người trong ô tô chỉ gán nếu nhìn thấy đủ rõ để xác định phần cơ thể.

## 3.3. Đối tượng chồng lấn
- Mỗi đối tượng nhận diện được phải có box độc lập.
- Các box được phép chồng lên nhau.
- Không gộp hai người hoặc hai xe chỉ vì chúng đứng sát nhau.
- Với vùng quá đông không thể phân tách, sử dụng ignore_region theo quy định của dự án.

## 3.4. Đối tượng cắt mép
- Box dừng tại mép ảnh.
- Đặt truncated = yes.
- Không suy đoán kích thước phần nằm ngoài ảnh.

## 4. Ca khó

## 4.1. Xe tải chở nhiều ô tô — img_002

Quan sát: Một xe tải chuyên dụng chở nhiều ô tô trên hai tầng.

Quyết định:
- Xe vận chuyển chính: một box truck.
- Mỗi ô tô được chở: một box car riêng.
- Box xe tải ôm phần cấu trúc xe vận chuyển.
- Box ô tô ôm phần từng ô tô nhìn thấy.
- Các box có thể chồng lấn.
- Không gộp tất cả phương tiện thành một box.

Lý do: Xe tải và các ô tô được chở là những đối tượng riêng biệt. Gộp chúng sẽ làm mất thông tin instance và sai số lượng phương tiện.

Trạng thái: Tự xử lý theo quy tắc.

## 4.2. Người bị che — img_005, img_026, img_040

Quan sát:
- img_005: nhiều người ngồi sát nhau trên xe máy.
- img_026: người lái bị người ngồi sau che một phần.
- img_040: người đội mũ bảo hiểm đứng trước taxi.

Quyết định:
- Mỗi người nhận diện được có một box person.
- Box ôm phần cơ thể nhìn thấy.
- Đánh dấu occluded_level theo mức che.
- Không gộp hai người vào một box.
- Không suy ra có xe máy chỉ vì một người đội mũ bảo hiểm.

Lý do: Tránh bỏ sót người và tránh dự đoán đối tượng không có bằng chứng trực quan.

Trạng thái: Tự xử lý; chỉ hỏi reviewer khi không xác định được đó có phải người thật hoặc không phân biệt được các cá thể.

4.3. Đối tượng quá nhỏ — img_013, img_025, img_031

Quan sát: Người và phương tiện xuất hiện rất xa trong ảnh đường phố.

Quyết định:
- Đối tượng có chiều lớn nhất dưới 20 px: không gán box riêng.
- Đối tượng đủ kích thước và đủ rõ: gán theo lớp.
- Không dùng vị trí trên đường để suy đoán lớp.
- Vùng nhiều đối tượng nhỏ không thể phân biệt có thể được đề xuất làm ignore_region.

Lý do: Hạn chế nhãn không đáng tin cậy và giảm sai lệch giữa các annotator.

Trạng thái: Hỏi reviewer để xác nhận ngưỡng 20 px và cách áp dụng ignore_region.

## 5. QC và phiên bản

- Mọi thay đổi làm đổi nhãn đều tăng phiên bản guideline.
- Ca mơ hồ: tick `can_review`.
