# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này \*\*trong lúc gán nhãn\*\*, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Lưu Thị Lan Anh`
Clip: `clip\_01`, `clip\_02`

\---

## 1\. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

|Gán|Không gán|
|-|-|
|xe con, SUV, taxi, xe bán tải|người đi bộ|
|van, minivan|xe đạp|
|xe buýt, minibus|**xe máy / mô tô**|
|xe tải, xe đầu kéo|xe trong ảnh quảng cáo, trong gương, dưới bóng nước|

Bổ sung của nhóm (nếu có): `Model chấm dùng 3 lớp COCO gốc \[car=2, bus=5, truck=7] để đại diện cho "vehicle" — van/minivan trong annotation được gộp chung vào nhóm "car" khi so khớp với output của model, vì YOLO không có lớp "van" riêng. Xe đỗ khuất hoàn toàn sau vật cản kiến trúc (cột, biển báo) trong hơn 2 giây thì không gán tiếp cho tới khi lại nhìn thấy được ít nhất 30% diện tích xe.`

## 2\. Luật ID — phần quan trọng nhất

|Tình huống|Luật của nhóm|Vì sao|
|-|-|-|
|Xe bị che một phần rồi hiện lại|giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps)|Dưới ngưỡng này gần như chắc chắn không có xe khác chen vào thay thế vị trí, và giữ ID giúp track phản ánh đúng chuyển động liên tục thay vì bị coi là "mất dấu" chỉ vì bị che một phần (ví dụ gt\_track 6 trong clip\_01, bị che khoảng 16 frame quanh frame 104–120 khi đi ngang qua các xe đỗ khác, vẫn giữ nguyên 1 ID xuyên suốt 79 frame).|
|Xe bị che lâu hơn ngưỡng trên|tạo **track mới**, không cố gán lại ID cũ dù nghi ngờ là cùng xe|Sau \~2 giây bị che hoàn toàn, xác suất một xe khác đã đi vào/ra khỏi vùng che tăng lên đáng kể, và annotator không còn đủ căn cứ hình ảnh (màu sắc, hình dạng) để khẳng định chắc chắn đó vẫn là cùng một xe — cố gán lại ID cũ risk tạo ra lỗi ID switch "ẩn" mà không ai kiểm tra được.|
|Xe rời khung hình rồi quay lại|mặc định: **track mới** — chỉ tính là "rời khung" khi bbox đã chạm hẳn ra ngoài rìa ảnh (không còn phần thân xe nào lọt trong khung), không tính trường hợp chỉ đi chậm lại gần rìa|Ranh giới khung hình không giữ được thông tin appearance liên tục (không có gì để "theo dõi" khi xe hoàn toàn ra khỏi cảnh), nên coi là track mới là lựa chọn an toàn và nhất quán giữa các annotator, tránh tranh cãi kiểu "có chắc là cùng xe quay lại không".|
|Hai xe cắt nhau / chồng lên nhau|giữ nguyên ID của mỗi xe theo **hướng di chuyển và tốc độ trước khi giao cắt** (ngoại suy tuyến tính vị trí kỳ vọng), không đổi ID theo vị trí bbox tức thời tại điểm giao cắt|Tại điểm hai bbox chồng khít (như track 4 và track 7 trong clip\_01, quãng frame 104–110, gần rìa phải khung hình), việc chỉ nhìn khung hình đơn lẻ dễ gán nhầm ID nếu hai xe đổi thứ tự trước/sau; ngoại suy theo quỹ đạo và tua qua lại nhiều lần giúp giữ đúng danh tính từng xe.|

## 3\. Luật bbox

|Tình huống|Luật của nhóm|
|-|-|
|Xe bị cắt bởi rìa ảnh|bbox chạm đúng rìa, không đoán phần ngoài ảnh|
|Xe bị xe khác che một phần|bbox ôm phần **nhìn thấy được**|
|Xe vừa xuất hiện, còn rất nhỏ / rất mờ|bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `cạnh dài nhất của bbox ước lượng ≥ 15px và tỉ lệ khung (width/height) đủ để phân biệt với xe máy (loại hình chữ nhật nằm ngang rõ ràng, không phải hình que đứng của người/xe máy). Ví dụ: track 8 trong clip\_01 chỉ được bắt đầu từ frame 134, khi bbox đã đạt khoảng 87×12px và rõ hình dạng đuôi xe, dù đối tượng có thể đã xuất hiện mờ ở vài frame trước đó.`|
|Xe đang đỗ, không di chuyển|vẫn giữ 1 track duy nhất xuyên suốt thời gian xe đứng yên; keyframe đặt thưa hơn (mỗi 15–20 frame) vì bbox gần như không đổi — ví dụ track 2 trong clip\_01 đứng yên gần như toàn bộ 190 frame chỉ có vài keyframe.|
|Keyframe đặt dày ở đâu|dày nhất (mỗi 3–5 frame) ở các đoạn: xe tăng/giảm tốc đột ngột, xe đi qua vùng bị che một phần, xe di chuyển nhanh và xa trong thời gian ngắn (như track 4, di chuyển liên tục suốt 101 frame với tốc độ biến thiên) — vì nội suy tuyến tính giữa hai keyframe cách xa nhau dễ làm bbox lệch tâm so với vị trí thật.|

## 4\. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

* Clip / frame / ID: `clip\_01, frame 104–120, track 6`
* Tình huống: `Xe (track 6) đi ngang qua khu vực có nhiều xe đỗ khác, bị che một phần diện tích (\~40–50%) trong khoảng 16 frame liên tiếp; đồng thời gần điểm này cũng là nơi model BoT-SORT+ReID về sau bị mất dấu và tách ID (id switch ghi nhận ở frame 107 và 110 trong eval\_reid\_vs\_me.json).`
* Quyết định: `Giữ nguyên 1 ID xuyên suốt vì thời gian che dưới ngưỡng 25 frame; bbox trong đoạn che chỉ ôm phần thân xe còn nhìn thấy được, không đoán phần bị khuất.`
* Lý do: `Đúng theo luật mục 2; đây cũng là bằng chứng cho thấy con người xử lý occlusion ngắn tốt hơn model hiện tại, đáng note lại trong phần phân tích ở REPORT.md.`

### Ca 2

* Clip / frame / ID: `clip\_01, frame 134, track 8`
* Tình huống: `Xe mới xuất hiện ở góc dưới khung hình, kích thước ban đầu rất nhỏ (bbox khoảng 87×12px) và tỉ lệ khung gần giống một vật thể dài mảnh, dễ nhầm với vệt bóng hoặc vật thể không phải xe bốn bánh.`
* Quyết định: `Chờ thêm 1–2 frame cho tới khi hình dạng đuôi/thân xe rõ ràng hơn rồi mới bắt đầu track, thay vì gán ngay từ frame vật thể vừa lọt vào khung.`
* Lý do: `Tránh tạo track giả cho các vật thể mơ hồ; đổi lại, chấp nhận track có thể ngắn hơn 1–2 frame so với thời điểm vật thể thực sự xuất hiện — đánh đổi giữa precision và recall của chính bộ annotation.`

### Ca 3

* Clip / frame / ID: `clip\_01, frame 104–115, track 4 và track 7`
* Tình huống: `Hai xe di chuyển gần rìa phải khung hình, bbox chồng lấn gần như hoàn toàn trong vài frame liên tiếp khi một xe vượt qua xe kia; nếu chỉ nhìn từng frame tĩnh, không thể phân biệt bbox nào thuộc xe nào.`
* Quyết định: `Ngoại suy theo quỹ đạo và tốc độ của mỗi xe trước điểm giao cắt để giữ đúng ID xuyên suốt, xác nhận lại bằng cách tua chậm qua lại đoạn này nhiều lần.`
* Lý do: `Vị trí bbox tức thời tại điểm giao cắt không đủ thông tin để phân biệt hai xe; quỹ đạo trước đó là căn cứ đáng tin hơn.`

## 5\. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

* `Định nghĩa "rời khung hình" ban đầu chưa đủ rõ, gây bất đồng khi kiểm chéo (một người coi bbox chạm rìa là đã "rời khung", người kia chờ đến khi biến mất hoàn toàn) — đã bổ sung định nghĩa cụ thể ở mục 2: chỉ tính là rời khung khi không còn phần thân xe nào lọt trong ảnh.`
* `Ngưỡng kích thước tối thiểu để bắt đầu track khi xe mới xuất hiện chưa có con số cụ thể trong bản đầu, dẫn đến khác biệt vài frame giữa các annotator (như trường hợp track 8 ở Ca 2) — đã thêm ngưỡng cụ thể (\~15px cạnh dài nhất + tỉ lệ khung rõ ràng) ở mục 3.`

