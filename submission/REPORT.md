# Báo cáo nhóm 30

## 1. Số liệu luyện tập
- F1 theo seed (rule / cross_check / overall): seed 30007: lỗi luật ≈ 1.0 / lỗi gán nhãn ≈ 0.6 / F1 tổng ≈ 0.8; Truy vết F1: tier_a 1.0 / tier_b 1.0 / notify 1.0.
  - Riêng phần lỗi luật file + detection (`rules_file_det.py` — Tùng):
    - Lô sạch `runs/clean` (`--seed 5 --no-faults`, 50 ảnh): **0 lỗi** (FP = 0).
    - Seed luyện tập `runs/r30007` (250 ảnh): **6 lỗi** (`missing_image`: 2, `wrong_hash`: 1, `bbox_out_of_bounds`: 1, `duplicate_box`: 2, `invalid_class`: 0).
    - Seed thử thêm `runs/r30008` (250 ảnh): **7 lỗi** (`wrong_hash`: 1, `invalid_class`: 2, `bbox_out_of_bounds`: 2, `duplicate_box`: 2, `missing_image`: 0).
  - Riêng phần lỗi luật metadata + panoptic (`rules_meta_pan.py` — Hương):
    - Lô sạch `runs/clean`: **0 lỗi** (FP = 0).
    - Seed luyện tập `runs/r30007`: **5 lỗi** (`scene_leak`: 1, `guideline_mismatch`: 1, `invalid_category_id`: 1, `thing_overlap`: 2).
    - Lô kiểu đề ẩn `runs/opaque77` (`inject_release.py --seed 77 --opaque`: tên ảnh mã hoá, id đánh lại, byte ảnh/mask đổi): **4 lỗi**, mỗi loại 1.
  - Validator đã ghép đủ 9 loại lỗi luật (`validate.py` gọi `rules_file_det` + `rules_meta_pan`): lô sạch 0 lỗi; F1 lỗi luật ≈ 1.0 trên seed 30007, 11, 12, 13 và `opaque77` (`score_findings.py`, F1 làm tròn).
- Số ứng viên trên lô sạch (`--no-faults`): **1** ứng viên / 50 ảnh (`instance_merge`, lệch tự nhiên của COCO), 0 lỗi luật; hàng đợi trên seed luyện tập: **24 ảnh** (`instance_merge` 10, `class_swap` 9, `missing_box` 4, `bbox_drift` 1).
  - Xem ảnh (Huỳnh làm công cụ `tools/viewer.py`, chia đều 5 người, gộp bằng `tools/gop_review.py`): **18 confirmed, 6 rejected**. Cả 6 ảnh bị bác đều là `instance_merge`: hai khung detection chồng lên nhau nên khung sau "dính" màu của segment bên cạnh, nhưng vật thứ hai thật ra có segment riêng (vd. `000000007108` đàn voi, `000000010707` hai cốc, `000000017627` hai xe ô tô).
  - 4 ảnh validator đoán `class_swap` nhưng xem ảnh thì detection đúng, panoptic sai, nên đổi thành `category_swap`: `000000001532` (car/truck), `000000006723` (truck/bus), `000000015079` (knife/fork), `000000018380` (cốc nhựa: cup/wine glass).
  - `errors.json` cuối: **29 lỗi** = 11 lỗi luật + 18 lỗi gán nhãn đã xác nhận (`class_swap` 5, `category_swap` 4, `instance_merge` 4, `missing_box` 4, `bbox_drift` 1).
- Ngưỡng đã chọn và cách chọn (không dùng `_private`):
  - **Luật file + detection (`rules_file_det.py`):** Chọn theo định nghĩa ở `docs/DELIVERY.md` và kiểm bằng lô sạch `runs/clean` (50 ảnh calib):
    - `BOUND_EPS = 1e-4` (toạ độ chuẩn hoá, tương đương ~0,064 px trên ảnh 640 px): bỏ qua sai số làm tròn 6 chữ số thập phân của nhãn YOLO gốc (trên lô sạch và `r30007`, box sát mép tự nhiên chỉ lệch tối đa ~0,0003 px, không gây FP), nhưng bắt được mọi box tràn mép thật (ở `r30007` tràn ~27,2 px).
    - `DUP_SIZE_TOL_PX = 0.5` và `DUP_SHIFT_MAX_PX = 2.5` (pixel): theo `docs/DELIVERY.md`, `duplicate_box` là box lặp cùng lớp, cùng kích thước, tâm chỉ lệch 1–2 px. Không dùng ngưỡng IoU thấp vì COCO có các cặp vật thật chồng nhau tự nhiên tới IoU ~0,66. Kiểm đồng thời chênh lệch kích thước (`|Δw|·W <= 0.5`, `|Δh|·H <= 0.5`) và độ lệch tâm (`|Δx|·W <= 2.5`, `|Δy|·H <= 2.5`) giúp lô sạch ra 0 FP (cặp box cùng lớp gần nhất trên lô sạch lệch tâm >= 6 px hoặc chênh kích thước >= 2,34 px) mà vẫn bắt đúng 2 cặp box lặp ở `r30007` (cùng kích thước `size_diff = 0`, lệch tâm đúng 1 px và 2 px).
  - **Luật metadata + panoptic (`rules_meta_pan.py`):** ba luật đầu là so khớp chính xác, không có ngưỡng; riêng `thing_overlap` chọn so sánh **không dung sai**.
    - `scene_leak`: gom ảnh theo `scene_id` trong `metadata.json`; cảnh nào có hơn một giá trị `split` thì báo, target là `scene_id`.
    - `guideline_mismatch`: so `guideline_version` của ảnh với cột `guideline` của **job** trong `jobs.csv` (không so với đa số ảnh trong job, vì guideline gốc ghi ở job).
    - `invalid_category_id`: `category_id` của segment không có trong `categories` của `panoptic.json`.
    - `thing_overlap`: đếm pixel từng segment trong mask (id = R + G·256 + B·256², bỏ id 0 là nền), báo khi `area` khai **lớn hơn** số pixel thật. Không đặt dung sai vì đo trực tiếp trên dữ liệu: ở lô sạch cả 643/643 segment có `area` **bằng đúng** số pixel; ở `r30007` 2688/2690 segment bằng đúng, 2 segment còn lại thừa 671 px (23,7% area) và 31 383 px (17,8%); ở `opaque77` 2689/2690 bằng đúng, segment còn lại thừa 17 556 px (26,6%). Không có segment nào `area` nhỏ hơn số pixel. Khoảng cách giữa "bằng đúng" và "thừa hàng trăm pixel" rất xa nên ngưỡng 0 không gây FP; mask là PNG (nén không mất dữ liệu) nên việc đổi byte ở đề ẩn không làm lệch số pixel.
   - **Đối chiếu detection ↔ panoptic (`cross_check.py` — Việt):** Đổi box YOLO sang pixel, ghép mỗi box với segment `thing` có IoU bbox lớn nhất và chỉ nhận `IoU > 0,3`. Khác tên lớp và `IoU > 0,4` thì báo `class_swap` (máy không tự quyết `category_swap`, để người xem ảnh đổi); cùng lớp thì dùng pixel mask: `bbox_drift` khi `coverage < 0,70` hoặc `IoU < 0,42` hoặc (`IoU < 0,74` và box `> 2%` ảnh), với box `> 0,5%` ảnh; `instance_merge` khi segment `>= 1024 px`, hai box cùng lớp có `IoU < 0,25` và (`in1 + in2 >= 70%` số pixel segment hoặc mỗi box phủ `>= 60%` diện tích và `>= 800 px`); `missing_box` khi segment chưa ghép có `>= 1024 px` và `> 1%` ảnh. Bỏ ảnh đã có lỗi luật, mỗi ảnh chỉ giữ một ứng viên theo ưu tiên `class_swap > missing_box > instance_merge > bbox_drift`. Kết quả: lô sạch 1 ứng viên/50 ảnh, seed 30007 có 24 ứng viên; sau khi 5 người xem ảnh: **18 confirmed, 6 rejected**.

## 2. Vì sao báo hết ứng viên làm F1 giảm?
Ứng viên chỉ là chỗ detection và panoptic **bất đồng**, không phải lỗi chắc chắn. Bất đồng có thể do lệch tự nhiên (vật chồng nhau, bị che), và khi lớp khác nhau thì máy không biết bên nào sai. Số của nhóm trên seed 30007 (phần lỗi gán nhãn, coi kết quả xem ảnh là đúng):

| | Báo hết 24 ứng viên theo type máy đoán | Báo sau khi xem ảnh |
|---|---|---|
| Đúng (TP) | 14 | 18 |
| Báo nhầm (FP) | 10 = 6 ảnh bị bác + 4 ảnh sai type | 0 |
| Bỏ sót (FN) | 4 (4 `category_swap` không được khai) | 0 |
| Precision / Recall / F1 | 0,58 / 0,78 / **0,67** | 1 / 1 / 1 |

- **6 ảnh bị bác** (25% hàng đợi) thành báo nhầm nếu khai hết, làm precision giảm.
- **4 ảnh sai type** bị phạt **hai lần**: khai `class_swap` không có trong đáp án (FP), còn `category_swap` thật thì không được khai (FN). F1 chấm theo cặp (type, ảnh) nên đúng ảnh mà sai type vẫn tính là sai.
- Sai type còn ảnh hưởng bàn giao: `class_swap` là lỗi detection nên ảnh bị **bỏ khỏi v2**, còn `category_swap` chỉ lỗi panoptic nên ảnh **ở lại v2**. Khai theo máy sẽ loại oan 4 ảnh.
- Lô sạch ra 1 ứng viên / 50 ảnh, cho thấy hàng đợi luôn có lệch tự nhiên, nên bước xem ảnh là bắt buộc chứ không chỉ để cho đẹp.

## 3. Quyết định và lineage
- **QC v1:** Phát hiện 29 lỗi trên tổng số 250 ảnh (tỉ lệ 29/250 = **11,6%**). Theo quy tắc nghiệm thu (ngưỡng tối đa cho phép là 2%), kết luận bắt buộc là **HOLD** lô v1 (không Release).
- **Lineage:** Nhóm đã ghi nhận quyết định `HOLD` cho `lab250_70a563f4cf6f` vào `lineage.jsonl`. Sau khi xử lý bàn giao bản sửa đổi v2 (`lab250_70a563f4cf6f_v2`), nhóm tiếp tục ghi nhận sự kiện phát hành release v2 với lô cha (parent) là v1.
- **Vì sao training run R-021 phải xem lại hoặc train lại:** Khi truy vết tác động bằng lệnh `lineage.py impact --release-id lab250_70a563f4cf6f`, hệ thống báo `R-021: affected=True action=review_or_retrain`. Lý do kỹ thuật là mô hình máy học học trực tiếp trọng số từ các phân phối và nhãn dữ liệu của lô huấn luyện. Việc phát hành lô dữ liệu mới (v2) không tự động làm thay đổi hay sửa chữa trọng số của model đã được huấn luyện trên lô v1 (vốn chứa tới 11,6% dữ liệu sai lệch). Do đó, mô hình `R-021` đã bị nhiễm sai số và bắt buộc phải được đánh giá lại hiệu năng trên tập kiểm thử độc lập hoặc phải được train lại hoàn toàn trên tập dữ liệu v2 sạch.

## 4. Hash chứng minh gì, không chứng minh gì?
- **Hash chứng minh điều gì:** Mã băm mật mã (SHA-256) chứng minh **tính toàn vẹn kỹ thuật (integrity)** và **tính bất biến** của tệp dữ liệu. Nó chứng thực rằng file ảnh/nhãn mà người nhận tải về hoàn toàn trùng khớp từng byte với tệp lúc người bàn giao lập `manifest.csv`, đảm bảo không có sự sai lệch do đường truyền, lưu trữ hỏng hay bị can thiệp trái phép.
- **Hash KHÔNG chứng minh điều gì:** Mã băm **không chứng minh được tính đúng đắn về mặt ngữ nghĩa (semantic correctness)** của dữ liệu. Một tệp nhãn bị gán sai lớp đối tượng (ví dụ dán nhãn xe con thành xe tải), bounding box bị trôi lệch, hoặc một bức ảnh chụp nội dung rác/sai quy cách vẫn sinh ra một mã SHA-256 hoàn toàn hợp lệ. Hash bảo vệ tính nguyên vẹn của nội dung, nhưng chỉ có quy trình kiểm định chất lượng (QC), đối chiếu đa nguồn và review bằng mắt người mới xác minh được nhãn có đúng hay không.

## 5. Truy vết
- **Lớp bị thay đổi quy tắc (`rule_class`):** Qua đối chiếu `guideline/changelog.md` và `incoming/ticket.md`, lớp bị thay đổi quy chuẩn gán nhãn giữa v1.2 và v1.3 là `surfboard` (ở v1.3 quy định vật bị che trên 50% hoặc cắt ở mép ảnh vẫn phải gán box, còn v1.2 cho phép bỏ qua).
- **Tiêu chí phân tầng truy vết:**
  - **Tầng A (tier_a - 2 ảnh):** Ảnh thuộc các cảnh mà team AI đã nêu đích danh trong ticket (`scene_497`, `scene_734`), thuộc các job áp dụng guideline cũ v1.2 (J01-J04), và trong file nhãn detection thực tế có chứa ít nhất một bounding box thuộc lớp `surfboard`.
  - **Tầng B (tier_b - 4 ảnh):** Các ảnh khác cũng thuộc job áp dụng guideline cũ v1.2 và có nhãn `surfboard`, nhưng nằm ở các cảnh khác chưa bị team AI phát hiện.
  - **Danh sách cảnh báo (`notify`):** Toàn bộ người gán (`annotator`) và người kiểm (`reviewer`) của các job chứa ảnh thuộc tầng a và b. Nhóm xác định gồm 3 nhân sự: `A03`, `A05`, `Q03`.
- **Vì sao lấy guideline theo job (`jobs.csv`) thay vì lấy theo `metadata.json`:** Vì trong lô dữ liệu có thể tồn tại lỗi `guideline_mismatch` (metadata của ảnh bị ghi sai phiên bản guideline so với phiên bản ban hành của job). Guideline là chính sách quản lý được phân bổ và thống nhất theo từng đợt công việc (Job), do đó thông tin tại `jobs.csv` là căn cứ xác thực gốc; việc lấy theo job giúp đảm bảo không bị bỏ sót hoặc khoanh vùng sai các ảnh cần truy vết.

## 6. Bug hoặc điểm chưa rõ trong lab (nếu có)
<<>>
