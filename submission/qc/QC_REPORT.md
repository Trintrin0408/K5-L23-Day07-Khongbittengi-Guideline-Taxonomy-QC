# Báo cáo QC — Khongbittengi

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Thực hành đặc tả và kiểm chất lượng nhãn bbox 2D trên 40 ảnh tuyển chọn. Độ đại diện cho dữ liệu vận hành chưa được đánh giá.
- Nguồn và phép sử dụng dữ liệu: Xem tác giả, nguồn và giấy phép của từng ảnh trong `data/ATTRIBUTION.csv`.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json`. SHA-256 của COCO nhãn cuối là `7a9f4932c7e234565669a88030856338f58bb47092584ab805141e6508dc6b96`, khớp bản chụp của vòng 2.
- Guideline và taxonomy khi gán / khi bàn giao: Gán độc lập theo guideline v1.0 và taxonomy v1.0. Bản nhãn sau calibration dùng guideline v1.1 (editorial) và giữ taxonomy v1.0.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt trên 40 ảnh, 4 thành viên, mỗi người 20 ảnh.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 246 cặp ghép 1–1, IoU trung bình 0,780, đồng thuận lớp 245/246 (99,6%). Nguồn: `submission/qc/iaa_summary.md`.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 102/206 box chưa ghép ở hai phía khi cộng các cặp; 0 box được bỏ qua do ignore; 82 ca bất đồng vùng ignore.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Guideline v1.1 mục 7 ghi trường hợp van trong `img_005.jpg` được cả hai gán `truncated=no` dù nhóm xác định là `yes` theo R-05. Chưa có xác nhận đối chiếu mẫu của coach.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Nhật ký có 590 dòng trên 34/40 ảnh. Reviewer 286 đã xác nhận 5 box qua 10 dòng liên quan. 580 dòng khác có kết quả đối chiếu dữ liệu, gồm C124 và C264 vừa được sửa hình học. Các dấu vết đối chiếu có hỗ trợ AI được ghi riêng trong `kiem_lai`.
- Tác động v1.1 lên nhãn và ảnh: Các quyết định ghi phạm vi theo ảnh. Bản v1.1 làm rõ cách áp dụng các quy tắc hiện có và giữ nguyên lớp, thuộc tính và ngưỡng.
- Tham chiếu bằng chứng khi Rework: Giữ phiếu Release của phiên bản trước trong `submission/qc/reviews/round-01/`. Bản sau sửa C124 và C264 nằm ở `submission/qc/reviews/round-02/`. Ba vòng trước lần chạy lại nằm trong `submission/qc/reviews/history-20261117-before-restart/`.

### Kết quả xác nhận 5 box có cờ ở lần chạy trước

Reviewer `286` xác nhận 5 ca ngày 17/11/2026. Các ID dưới đây tham chiếu nhãn vòng 1 trong thư mục lịch sử và COCO cũ `labels_calibrated_901d7068c987807e.json`. Bản COCO sau khi chốt 5 ca là `labels_calibrated_d5734a9c824e5a44.json`. Manifest hiện tại trỏ đến phiên bản khớp nhật ký mới nhất.

| Ca cờ (ca liên quan) | Ảnh | ID vòng 1 | ID COCO cũ | Kết quả |
|---|---|---:|---:|---|
| C213 (C212) | `img_011.jpg` | 155 | 153 | Giữ xe máy độ đang trưng bày theo mô tả ảnh nguồn. R-15. |
| C234 (C233) | `img_013.jpg` | 165 | 163 | Bỏ box vì vùng tối giữa các lá cờ thiếu đường nét để nhận ra người. R-12. |
| C243 (C238) | `img_014.jpg` | 169 | 167 | Giữ xe máy mờ giữa hai người đi bộ, occluded_level=3, truncated=no. R-12. |
| C244 (C239) | `img_014.jpg` | 170 | 168 | Giữ xe máy mờ bị mép trái cắt, occluded_level=3, truncated=yes. R-05 / R-12. |
| C522 (C520) | `img_033.jpg` | 372 | 370 | Bỏ box vì gương, cửa và bản lề chồng với cảnh qua kính chưa đủ để nhận ra lớp van. R-16. |

Ba box xe máy được giữ nguyên hình học và thuộc tính che/cắt mép, đồng thời đã tắt cờ. Hai box bị loại có vùng bbox được ghi trong `calibration_decisions.csv`. Việc ghi vùng ở giai đoạn calibration giữ được bảng ca và đầu vào IAA của lần gán độc lập.

### Đối chiếu 580 dòng calibration

- 572 dòng có dấu vết khớp giữa phiếu sửa và COCO. Kết quả này dùng để hỗ trợ duyệt ảnh.
- C007, C171, C292, C427 và C431 liên quan đến 4 box có `truncated=yes` nhưng hở mép ảnh 2,28–3,21 px. Đề xuất giữ thuộc tính và rà biên theo R-05.
- C124 và C264 đã được thu cạnh phải vùng `ignore_region` theo R-10. Tọa độ trước và sau có trong phiếu sửa.
- C588 cần xem box người lái tuk-tuk `img_040.jpg #442` theo R-14.

### Hậu kiểm hai vùng ignore

| Ca | Ảnh và box | Kết quả kiểm hình học |
|---|---|---|
| C124 | `img_008.jpg`, vùng `#116`, người `#94` | Cạnh phải từ x=951 xuống x=928,76 px. Phần giao với box người từ 94,7% xuống 0%. |
| C264 | `img_019.jpg`, vùng `#224`, người `#222` | Cạnh phải từ x=957 xuống x=925,50 px. Phần giao với box người từ 61,5% xuống 0%. |

Các ca không vẽ box được trích từ guideline và nhật ký gốc vào [bảng hậu kiểm](reviews/round-02/image_issues_post_calibration.csv). Bảng này ghi thời điểm hậu kiểm, nguồn và quy tắc cho từng ca.

## 4. Kết quả duyệt

- Vòng QC: 2
- Người review và ngày: `TEAM.md` phân công mã 286. Kết luận vòng 1 ghi ngày 17/11/2026. Vòng 2 chuyển kết luận cho 38 ảnh sau khi kiểm ảnh và nhãn khớp bản chụp vòng 1. Hai ảnh đã chỉnh chờ xác nhận.
- Đã duyệt / cả lô: 38/40 ảnh. `img_008.jpg` và `img_019.jpg` đang chờ duyệt lại.
- Ảnh lỗi / ảnh đã duyệt: 0/38 (0,0%) theo phiếu.
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/380 (0,0%) theo phiếu.
- Vật bỏ sót, ảnh và vùng: Phiếu đã ghi cho 38 ảnh không báo vật bỏ sót. Sàng lọc ảnh chồng nhãn có gợi ý xem thêm mép phải `img_014.jpg` theo R-02/R-12.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Phiếu ghi 32/34 ảnh ca khó và 6/6 ảnh còn lại. Bộ nhãn có 0 box bật `can_review`. Hai ảnh chờ xem lại ghi ở bảng hậu kiểm.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Script kiểm bản chụp nhãn và thiếu lượt duyệt. Người duyệt dùng ảnh để đánh giá độ sát biên của box, chẳng hạn hai xe mờ trong `img_014.jpg` theo R-01/R-12.

### Sàng lọc toàn bộ lô trước khi duyệt

- Sàng lọc 40 ảnh với 432 box đối tượng và 14 vùng `ignore_region` theo bộ nhãn trong manifest.
- Có 7 box `truncated=yes` còn cách mép gần nhất trên 2 px trong sàng lọc. Hai vùng ignore được thu như bảng hậu kiểm. Kiểm lại ranh giới cụm trên hai ảnh vòng 2.
- Ảnh chồng nhãn vòng 2 nằm trong `work/review/round-02/` trên máy, tái tạo bằng công cụ QC từ bộ nhãn và ảnh gốc.
- Lệnh QC tính kết quả chính thức từ các dòng trong `qc_review.csv`.

## 5. Quyết định bàn giao

- Quyết định: Hold
- Lý do, người nhận và phạm vi sử dụng: `submission/qc/reviews/round-02/qc_result.json` ghi Hold vì thiếu kết luận cho hai ảnh đã sửa. Nhóm và coach dùng hồ sơ trong phạm vi bài thực hành QC nhãn 2D.
- Nếu còn việc: Reviewer 286 xem `img_008.jpg` và `img_019.jpg`, ghi phiếu vòng 2 rồi chạy lại báo cáo. Thành viên bổ sung bản guideline độc lập của 286 và phần nhận xét cá nhân. Nhờ người ngoài nhóm mở gói và kiểm thử một ca.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Đã truy C124 từ bảng trên về `data/images/img_008.jpg`, guideline v1.1 R-10, dòng C124 trong `calibration_decisions.csv` và vùng `#116` của COCO mới. Kiểm lần này là đối chiếu kỹ thuật. Người ngoài nhóm cần xác nhận mở thử gói.
