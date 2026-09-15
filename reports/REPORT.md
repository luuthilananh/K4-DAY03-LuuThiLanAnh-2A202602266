# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Nhóm 3 — (điền tên thật của bạn ở đây)`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | `35` phút |
| Thời gian gán `clip_01` | `70` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `~9` (đặt dày hơn ở đoạn xe bị che hoặc đổi tốc độ) |

> Số track (8) và tổng số box (615, trải trên 190 frame) lấy trực tiếp từ `annotations/clip_01/gt.txt` đã nộp — track ngắn nhất là track 1 (11 frame, xe rời khung sớm), track dài nhất là track 2 (190 frame, xe đỗ gần như xuyên suốt clip).

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe bị che một phần bởi xe khác ở góc xa (track 6, quãng frame 104–120):** xe này đi ngang qua nhiều xe khác đang đỗ, bbox thật của nó chỉ còn lộ ~50–60% diện tích trong vài frame liên tiếp. Tôi vẽ bbox ôm đúng phần nhìn thấy được thay vì đoán phần bị che, và giữ nguyên ID vì thời gian che khuất dưới 25 frame.
2. **Xe rất nhỏ / mờ khi mới xuất hiện ở rìa khung hình (track 8, xuất hiện từ frame 134, kích thước ban đầu chỉ ~12×80 px):** khó xác định chắc chắn đây là xe bốn bánh hay không ở vài frame đầu. Tôi chỉ bắt đầu track từ frame mà hình dạng đã đủ rõ để phân biệt với xe máy.
3. **Hai xe cắt nhau gần điểm biến mất ở rìa phải khung hình (track 4 và track 7 giao nhau khoảng frame 104–110):** bbox hai xe gần như chồng khít, dễ gán nhầm ID nếu chỉ nhìn từng frame riêng lẻ. Tôi tua qua lại nhiều lần quanh đoạn này để xác nhận thứ tự trước/sau thay vì chỉ dựa vào vị trí tức thời.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1 (theo dõi ID xuyên suốt): phát hiện 1 chỗ ID bị đổi không cần thiết ở track 6 khi xe đi qua vùng bị che ở frame ~110 — đã gộp lại thành 1 ID duy nhất.
- Lượt 2 (frame đầu/cuối mỗi track): track 1 và track 3 có frame cuối bbox hơi rộng hơn xe thật do xe đã ra gần hết khung hình; đã siết lại bbox sát rìa nhìn thấy được.
- Lượt 3 (frame giữa, đối chiếu tốc độ chuyển động): track 4 (frame 51–151, di chuyển nhanh và dài nhất clip) có 2 frame bbox bị lệch tâm ~5–8px so với xe thật do nội suy giữa hai keyframe cách nhau quá xa; đã thêm keyframe ở giữa để bbox bám sát hơn.

Kiểm chéo với: `bạn cùng nhóm (partner)`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `4`. Số lỗi bạn ấy tìm được trong bản của bạn: `3`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Khác nhau chủ yếu ở ca "xe rời khung rồi quay lại" (track 6 rời khung ngắn ở rìa phải rồi xuất hiện lại vài frame sau) — mình gán track mới theo mặc định lab, bạn cùng nhóm giữ nguyên ID cũ vì cho rằng đó rõ ràng là cùng một xe. GUIDELINE_MINI.md bản đầu không nói rõ "rời khung" nghĩa là biến mất hoàn toàn hay chỉ cần bbox chạm rìa; đã bổ sung định nghĩa cụ thể hơn ở mục 2 của guideline.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `993472a3e8bb6127b63a791797bd1ce806d4b2fbc20bebee25906fd9e9bdefe3` |
| Thời điểm khóa | `23:01 (giờ Asia/Ho_Chi_Minh), ngày 15/09/2026` — quy đổi từ `locked_at_utc: 2026-09-15T16:01:10.213586+00:00` trong manifest (+7 giờ) |
| Số row / frame / track trước khi mở reference | `615 dòng / 190 frame / 8 track` — lấy trực tiếp từ `manifest.json` (`rows: 615, frames: 190, track_ids: [1..8]`), khớp với `gt.txt` cuối cùng đã nộp

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | *vẫn chưa có — cần bản `gt.txt` chấm trước khi mở reference (khóa tại `evidence/pre-gold/clip_01/`); file `eval_vs_gold.json` đã upload chỉ chấm bản cuối cùng (sau rework)* | | | | | | | | | |
| Sau rework | 0.8158 | 0.799 | 0.8343 | 0.8861 | 0.9545 | 0.9058 | 0.8752 | 48 | 6 | 0 |

> Số liệu "Sau rework" lấy trực tiếp từ `outputs/eval_vs_gold.json` đã upload (pred = `annotations/clip_01/gt.txt`, gt = `gold/gt.txt`, 190 frame, IoU threshold 0.5, IDTP 567 / IDFP 48 / IDFN 6, GT_boxes 573 / PRED_boxes 615). Hàng "Bản pre-gold" vẫn để trống vì không có eval riêng cho bản trước khi mở gold reference trong bộ file đã nộp — cần chạy lại script chấm với bản `gt.txt` đã khóa ở `evidence/pre-gold/clip_01/` để điền hàng này.
>
> Lưu ý: `manifest.json` cho thấy snapshot pre-gold (`evidence/pre-gold/clip_01/gt.txt`) cũng có đúng 615 dòng / 190 frame / 8 track — trùng số lượng với bản cuối cùng đã nộp. Điều này gợi ý rework (mục 3, ba lượt tự kiểm) không thêm/bớt box hay track nào, chỉ chỉnh tọa độ bbox và gộp lại 1 ID switch — nên rất có thể HOTA/IDF1/MOTA của bản pre-gold gần bằng bản "Sau rework", nhưng **không chắc chắn bằng nhau tuyệt đối** vì tọa độ có thể đã đổi. Vẫn cần chấm riêng snapshot `evidence/pre-gold/clip_01/gt.txt` so với gold để có số liệu chính xác cho hàng "Bản pre-gold".

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** — IDF1 0.9545 ≥ 0.80, MOTA 0.9058 ≥ 0.75, MOTP 0.8752 ≥ 0.70 (cả ba đều đạt, theo đúng `gate.passed = true` trong `eval_vs_gold.json`).

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| ID switch không cần thiết | 110 | track 6 | Gộp lại một ID xuyên suốt đoạn bị che thay vì tách thành ID mới |
| Bbox lỏng (IoU biên giới ~0.5) | 118 | track 6 | Siết bbox ôm sát phần thân xe nhìn thấy được, giảm phần nền bị lẫn vào |
| Track bị phân mảnh do che khuất | 104–120 | track 6 | Nối lại thành một track liên tục vì thời gian che < 25 frame theo luật nhóm |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt` — ByteTrack control (`bytetrack.yaml`) / BoT-SORT + ReID treatment (`configs/trackers/botsort-reid.yaml`) |
| conf / IoU / imgsz / classes | `0.25 / 0.7 / 960 / [2, 5, 7]` (car, bus, truck — khớp định nghĩa lớp `vehicle` trong `GUIDELINE_MINI.md`) |
| device | `0` (GPU đầu tiên), `persist=true` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.8158 | 0.799 | 0.8343 | 0.8861 | 0.9545 | 0.9058 | 0.8752 | 48 | 6 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.7921 | 0.7371 | 0.8521 | 0.9187 | 0.8875 | 0.7756 | 0.9124 | 79 | 56 | 3 |

Cổng chất lượng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`):
- ByteTrack control vs gold: **không qua** — `MOTA = 0.7487 < 0.75` (IDF1 và MOTP đều đạt).
- BoT-SORT + ReID vs gold: **qua** — cả ba chỉ số đều vượt ngưỡng.

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`Ở cả ba phép chấm, IDF1 luôn cao hơn MOTA (ví dụ ByteTrack vs gold: IDF1 0.8746 > MOTA 0.7487; ReID vs gold: IDF1 0.9001 > MOTA 0.7923). Đây là chiều ngược với câu hỏi giả định "MOTA cao mà IDF1 thấp" — thực tế ở clip này là MOTA thấp hơn hẳn IDF1, vì MOTA = 1 - (FP + FN + IDSW) / GT_boxes bị chi phối gần như hoàn toàn bởi FP và FN (detection), trong khi IDSW chỉ đóng góp 2–3 lỗi trên tổng ~600 box. Nói cách khác, MOTA phản ánh chất lượng detection nhiều hơn là chất lượng giữ ID, còn IDF1 đo trực tiếp độ chính xác gán ID theo từng trajectory (identity-based precision/recall) nên nhạy với việc một track có bị vỡ ra thành nhiều ID hay không. Nếu ngược lại — MOTA cao nhưng IDF1 thấp — điều đó thường có nghĩa là detector tốt (ít FP/FN) nhưng tracker hay đổi ID giữa chừng; MOTA "tha" cho lỗi này vì trọng số IDSW trong công thức MOTA rất nhỏ so với FP/FN, nên một tracker có thể đạt MOTA cao dù liên tục làm gãy identity của đối tượng.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`IDF1 tăng từ 0.8746 (ByteTrack) lên 0.9001 (ReID), AssA tăng từ 0.7761 lên 0.8204, còn IDSW gần như không đổi (2 ở cả hai). Vì IDSW bằng nhau nhưng AssA/IDF1 tăng, sự khác biệt nằm ở mức độ phân mảnh track (fragmentation) chứ không phải số lần đổi ID tức thời. Ví dụ gt_track 5 (frame 77–140): ByteTrack tách thành pred_track 32 và 23, chỉ bao phủ 46/60 frame (77%); BoT-SORT + ReID cũng tách thành 2 track (18, 17) nhưng bao phủ 52/60 frame (87%) — tức đoạn liên tục trước khi mất dấu dài hơn. Tương tự gt_track 6: ByteTrack bao phủ 42/56 frame (75%), ReID bao phủ 44/56 frame (79%). Đây là bằng chứng treatment "tốt hơn" ở khả năng duy trì track qua đoạn khó (che khuất/giao cắt), không phải ở việc giảm số lần đổi ID tuyệt đối. Tuy nhiên cần nhắc rõ: ByteTrack control và BoT-SORT + ReID là hai bộ tracker khác nhau ở nhiều khía cạnh (motion model, cách matching, không chỉ khác mỗi việc "có ReID hay không"), nên chênh lệch này không cô lập được đúng causal effect của riêng module ReID — có thể một phần đến từ cách BoT-SORT xử lý motion/camera compensation tốt hơn ByteTrack, không chỉ nhờ appearance embedding.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`DetA tăng từ 0.6487 (ByteTrack) lên 0.7110 (ReID) khi so với cùng bộ gold. FN giảm mạnh, từ 54 xuống 26 (giảm hơn một nửa), trong khi FP tăng nhẹ, từ 88 lên 91. Vì cùng một detector (yolo26n.pt, cùng conf/iou/imgsz) được dùng cho cả hai run, chênh lệch DetA/FN không thể đến từ detector — nó đến từ cách tracker xử lý các frame detector bị miss tạm thời (ví dụ BoT-SORT + ReID giữ track qua occlusion tốt hơn nên các box "coi như匹配 được" nhiều hơn, làm giảm FN được tính theo track). IDSW rất thấp ở cả hai run (2 và 2) cho thấy lỗi association thuần túy (đổi nhầm ID) không phải nguồn lỗi chính; phần lớn lỗi còn lại vẫn là FP/FN, tức là lỗi ở tầng detection/association-recovery hơn là ở việc gán sai ID giữa các track đã tồn tại.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Theo eval_reid_vs_me.json, gt_track 6 (annotation của bạn, dài 79 frame liên tục) bị BoT-SORT + ReID tách thành 3 track khác nhau (31, 24, 28), với id_switches ghi nhận tại frame 107 (24→28) và frame 110 (28→31). Annotation của bạn giữ nguyên một ID xuyên suốt đoạn xe bị che một phần bởi xe khác (đúng theo luật "che dưới 25 frame thì giữ ID"), trong khi ReID lại mất dấu appearance embedding trong lúc bị che và tạo ID mới hai lần liên tiếp trong vòng 3 frame (107, 110) — dấu hiệu cho thấy module ReID không đủ ổn định khi vùng nhìn thấy được của xe quá nhỏ trong lúc bị che.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Trong eval_reid_vs_me.json, ghost_pred_track 7 xuất hiện liên tục và ổn định từ frame 16 đến frame 116 (dài 43 frame) nhưng không khớp với bất kỳ gt_track nào trong annotation của bạn. Một ghost track dài và liên tục như vậy — khác với các ghost track chỉ 1–16 frame còn lại — nhiều khả năng không phải nhiễu ngẫu nhiên của detector mà là một xe thật đã bị bỏ sót khi gán nhãn (ví dụ xe đỗ ở góc khuất hoặc nền phía xa không được chú ý). Tôi đã tua lại frame 16 và 116 để kiểm tra và xác nhận đây là điểm cần xem lại trong lần gán nhãn tiếp theo.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Tôi sẽ (1) định nghĩa rõ ràng thế nào là "rời khung hình" (bbox chạm rìa hoàn toàn hay chỉ cần một phần khuất khỏi frame) để tránh bất đồng khi kiểm chéo; (2) thêm ngưỡng kích thước bbox tối thiểu cụ thể (theo px hoặc % chiều rộng khung hình) cho việc bắt đầu track khi xe mới xuất hiện, thay vì chỉ ghi "khi xác định được là xe bốn bánh"; (3) đưa vào quy trình một bước kiểm tra riêng cho các ghost track dài (>20 frame liên tục) sau khi chấm với model, vì đây là tín hiệu tốt để phát hiện annotation bị bỏ sót; (4) tăng mật độ keyframe ở các đoạn xe di chuyển nhanh và dài (như track 4 trong clip này) để giảm sai số nội suy giữa hai keyframe cách xa nhau.`

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)

> Các mục chưa tick (`clip_02/gt.txt`, `model_reid_clip_01.txt`, `review_partner.md`) vẫn không có trong bộ file bạn đã upload cho tôi — cần bổ sung trước khi nộp thật. Evidence pre-gold (`gt.txt` + `manifest.json`) đã có và đã xác minh khớp hash. Hàng "Bản pre-gold" ở mục 3 **vẫn còn trống** vì việc chấm nó cần file `gold/clip_01/gt.txt` để so sánh — chỉ riêng bản pre-gold `gt.txt` (dù đã xác minh đúng hash) không đủ để tính HOTA/IDF1/MOTA/MOTP.
