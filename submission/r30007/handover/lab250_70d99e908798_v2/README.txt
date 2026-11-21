Release: lab250_70d99e908798_v2
Parent: lab250_70d99e908798
Số ảnh: 235
Số nhãn: 1635
Guideline: v1.2 (J01-J04), v1.3 (J05-J10)
QC v1: 29/250 = 11.6%
Kết luận v1: Hold — Tỷ lệ lỗi QC v1 là 29/250 = 11.6% vượt ngưỡng cho phép 2.0%
Kết luận v2: Release — Đã tách 15 ảnh lỗi sang rework_queue, chuẩn hóa split cảnh, cập nhật sha256 và giữ nguyên nhãn detection hợp lệ
Ghi chú: Split: cảnh rò rỉ (scene_leak) được gán theo split đa số trong metadata v1 (hòa chọn train); Hash: các ảnh sai hash đã được mở xem thực tế (ảnh valid giữ lại tính lại sha256 thực tế, ảnh damaged chuyển sang rework_queue); Guideline: lấy phiên bản guideline theo job trong jobs.csv; Rework: 15 ảnh lỗi được phân công lại cho annotator và reviewer theo jobs.csv; Panoptic: phát hiện lỗi panoptic trên các ảnh (000000001353 (instance_merge), 000000001532 (category_swap), 000000001584 (instance_merge), 000000002157 (instance_merge), 000000006723 (category_swap), 000000015079 (category_swap), 000000016598 (thing_overlap), 000000017182 (thing_overlap), 000000018380 (category_swap), 000000018575 (instance_merge), 000000022192 (invalid_category_id)), nhãn detection giữ nguyên trong v2 và đã báo team Panoptic
