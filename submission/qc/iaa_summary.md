# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 4
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 246 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 246 | 0.780 | 99.6% | 59.8% | 95.1% | 102/206 | 0 | CHƯA ĐẠT |
| 154|164 | 6 | 31 | 0.968 | 100.0% | 87.1% | 100.0% | 1/3 | 0 | ĐẠT |
| 154|169 | 9 | 50 | 0.728 | 100.0% | 62.0% | 94.0% | 37/23 | 0 | CHƯA ĐẠT |
| 154|286 | 5 | 42 | 0.789 | 97.6% | 61.9% | 90.5% | 13/65 | 0 | CHƯA ĐẠT |
| 164|169 | 5 | 28 | 0.816 | 100.0% | 60.7% | 92.9% | 25/8 | 0 | CHƯA ĐẠT |
| 164|286 | 9 | 69 | 0.731 | 100.0% | 36.2% | 97.1% | 13/63 | 0 | CHƯA ĐẠT |
| 169|286 | 6 | 26 | 0.737 | 100.0% | 80.8% | 96.2% | 13/44 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 590 ({'bo_sot_thua': 246, 'iou_thap': 144, 'sai_thuoc_tinh': 106, 'bat_dong_ignore': 82, 'sai_lop': 1, 'co_can_review': 11}), trên 34 ảnh cần calibration.
