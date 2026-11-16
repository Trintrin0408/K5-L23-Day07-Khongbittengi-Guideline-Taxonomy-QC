# Báo cáo QC — Khongbittengi

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Thực hành đặc tả và kiểm chất lượng nhãn bbox 2D trên 40 ảnh tuyển chọn. Độ đại diện cho dữ liệu vận hành chưa được đánh giá.
- Nguồn và phép sử dụng dữ liệu: Xem tác giả, nguồn và giấy phép của từng ảnh trong `data/ATTRIBUTION.csv`.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json`. SHA-256 của COCO nhãn cuối là `acf97c01caa827c6ed4d07676c49598d442d131e52423bebc499da3db8699893`, khớp bản chụp dùng cho vòng 3.
- Guideline và taxonomy khi gán / khi bàn giao: Gán độc lập theo guideline v1.0 và taxonomy v1.0. Bản nhãn sau calibration dùng guideline v1.1 (editorial) và giữ taxonomy v1.0.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt trên 40 ảnh, 4 thành viên, mỗi người 20 ảnh.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 246 cặp ghép 1–1, IoU trung bình 0,780, đồng thuận lớp 245/246 (99,6%). Nguồn: `submission/qc/iaa_summary.md`.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 102/206 box chưa ghép ở hai phía khi cộng các cặp; 0 box được bỏ qua do ignore; 82 ca bất đồng vùng ignore.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: Guideline v1.1 mục 7 ghi trường hợp van trong `img_005.jpg` được cả hai gán `truncated=no` dù nhóm xác định là `yes` theo R-05. Chưa có xác nhận đối chiếu mẫu của coach.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: Nhật ký có 590 dòng trên 34/40 ảnh. Reviewer 286 đã xác nhận 5 box qua 10 dòng liên quan. 580 dòng còn lại đã được điền kết quả đối chiếu dữ liệu có hỗ trợ AI. Trong đó 572 dòng khớp dấu vết và 8 dòng cần ưu tiên xem ảnh. Các dòng này đang chờ xác nhận duyệt ảnh.
- Tác động v1.1 lên nhãn và ảnh: Các quyết định ghi phạm vi theo ảnh. Bản v1.1 làm rõ cách áp dụng các quy tắc hiện có và giữ nguyên lớp, thuộc tính và ngưỡng.
- Tham chiếu bằng chứng khi Rework: Có bản chụp các vòng trong `submission/qc/reviews/`. Vòng 3 chụp lại cùng bộ nhãn và nhật ký đã điền 580 kết quả đối chiếu.

### Kết quả xác nhận 5 box có cờ ở vòng 1

Reviewer `286` xác nhận 5 ca ngày 17/11/2026. Các ID dưới đây tham chiếu nhãn vòng 1 và COCO cũ `labels_calibrated_901d7068c987807e.json`. Bản COCO mới là `labels_calibrated_d5734a9c824e5a44.json`.

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
- C124 và C264 có vùng `ignore_region` phủ phần lớn box người gán riêng. Rà biên vùng theo R-10.
- C588 cần xem box người lái tuk-tuk `img_040.jpg #442` theo R-14.

## 4. Kết quả duyệt

- Vòng QC: 3
- Người review và ngày: `TEAM.md` phân công mã 286. Reviewer xác nhận 5 ca calibration ngày 17/11/2026. Nhật ký ghi 580 kết quả đối chiếu dữ liệu ngày 17/11/2026. Phiếu vòng 3 đang chờ ghi người và ngày duyệt ảnh.
- Đã duyệt / cả lô: 0/40 ảnh.
- Ảnh lỗi / ảnh đã duyệt: 0/0 (chưa tính được tỉ lệ).
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/0 (chưa tính được tỉ lệ).
- Vật bỏ sót, ảnh và vùng: Phiếu đang trống, chưa có số liệu duyệt về vật bỏ sót.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Vòng 3 ghi nhận 0/34 ảnh ca khó và 0/6 ảnh còn lại đã duyệt. Bộ nhãn có 0 box bật `can_review`. Mục 4 liệt kê tín hiệu để ưu tiên khi duyệt.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Script kiểm bản chụp nhãn và thiếu lượt duyệt. Người duyệt dùng ảnh để đánh giá độ sát biên của box, chẳng hạn hai xe mờ trong `img_014.jpg` theo R-01/R-12.

### Sàng lọc toàn bộ lô trước khi duyệt

- Kiểm 40 ảnh với 432 box đối tượng và 14 vùng `ignore_region`. Bản chụp QC khớp nhãn, phiếu và nhật ký hiện tại.
- Phát hiện 7 box `truncated=yes` còn cách mép gần nhất trên 2 px và 2 cặp vùng ignore phủ từ 50% box riêng. Đây là dấu hiệu cần xem ảnh.
- Danh sách 8 ảnh ưu tiên gồm các ảnh từ kiểm tra trên và `img_040` (ca C588). Hướng dẫn và link đến cả 40 ảnh nằm trong `work/review/round-03/REVIEW_CHECKLIST.md` trên máy.
- Kết quả chính thức đọc từ phiếu `qc_review.csv`: 0/40 ảnh đã được reviewer ghi kết quả.

## 5. Quyết định bàn giao

- Quyết định: Hold
- Lý do, người nhận và phạm vi sử dụng: Khớp `submission/qc/reviews/round-03/qc_result.json`. Vòng 3 còn 40 ảnh chưa duyệt nên chưa có tỉ lệ lỗi. Nhóm và coach có thể dùng báo cáo để theo dõi phần còn thiếu của bài thực hành.
- Nếu còn việc: Reviewer dùng danh sách ảnh ở mục 4 để kiểm tra các ảnh ưu tiên và duyệt đủ 40/40 ảnh. Sau khi ghi phiếu, chạy `qc_report.py --round 3` và cập nhật báo cáo.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Tra mã ca trong `calibration_decisions.csv`. Mỗi ghi chú đối chiếu dẫn đến ảnh, box COCO hoặc vùng cần xem. Dùng `batch_manifest.json` để kiểm phiên bản nhãn.
