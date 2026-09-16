# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: Hoàng Công Chứ  - 2A202602187 (làm cá nhân)
Ngày: 2026-09-15

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: CVAT Community 2.74.1 (Docker local) |
| Thời gian gán `clip_02` (warm-up) | 30 phút |
| Thời gian gán `clip_01` | 90 phút |
| Số track đã vẽ trong `clip_01` | 8 track (ID 1–8) |
| Số keyframe trung bình mỗi track | ~5 keyframe/track |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe bị che khuất khi đi cắt nhau (frame 60–80):** Hai xe (track 4 và 5) đi chéo cắt nhau, bounding box chồng lấn lớn. Xử lý: bật thuộc tính `Occluded` cho xe bị che khuất ở phía sau, giữ nguyên ID của từng xe qua đoạn giao cắt, và cắm keyframe ngay trước/sau điểm giao cắt để tránh trôi hộp.
2. **Xe nhỏ xuất hiện ở xa ở rìa trên khung hình (track 2 những frame đầu):** Kích thước rất bé, mờ và khó phân biệt giữa xe con hay phương tiện khác. Xử lý: chỉ bắt đầu tạo track và vẽ bounding box từ frame nhận diện rõ ít nhất 30% thân xe bốn bánh theo đúng guideline.
3. **Xe đứng yên trong thời gian dài (track 3 từ frame 1–15):** Bounding box bất động nhiều frame liên tiếp, dễ nhầm với việc quên bấm `Outside`. Xử lý: kiểm tra kỹ từng frame bằng mắt để xác nhận xe thực sự đang đỗ/dừng trong cảnh trước khi di chuyển; giữ nguyên track ID và tọa độ hộp.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Rà soát tính nhất quán của ID qua toàn bộ video. Không phát hiện hiện tượng ID switch (đổi nhầm ID giữa các xe), đặc biệt ở đoạn giao nhau phức tạp frame 60–80.
- Lượt 2: Kiểm tra frame xuất hiện (entry) và biến mất (exit) của từng track. Phát hiện track 3 đứng yên frame 1–15, đã kiểm tra lại và xác nhận xe đỗ yên thật, không phải box treo hay quên Outside.
- Lượt 3: Kiểm tra nội suy hình học (interpolation drift) ở các frame nằm giữa keyframe. Thêm keyframe tại frame 93 và 97 cho track 4 khi xe rẽ phải để bounding box ôm sát thân xe, không bị lệch midpoint.

Kiểm chéo với: Tự kiểm chéo độc lập (bài làm cá nhân). Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: 0 (làm cá nhân). Số lỗi bạn ấy tìm được trong bản của bạn: 1 (cảnh báo track 3 bbox đứng im frame 1–15; đã xác minh là xe đỗ thật).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Do làm cá nhân nên không có xung đột giữa hai người. Luật còn thiếu ban đầu là: chưa quy định rõ ngưỡng frame đứng yên tối đa cần kiểm tra lại để phân biệt giữa xe dừng đỗ thật và lỗi quên đánh dấu Outside, cũng như tỷ lệ diện tích nhìn thấy tối thiểu (30%) khi xe bắt đầu tiến vào khung hình từ rìa ảnh.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `[f568e07d2f819d2a11126982a2681a294cc11b9909e7a3b849434f0c4fda32be]` |
| Thời điểm khóa | `2026-09-15` |
| Số row / frame / track trước khi mở reference | `614 rows / 190 frames / 8 tracks` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | — | — | — | — | — | — | — | — | — | — |
| Sau rework | 0.742 | 0.726 | 0.760 | 0.825 | 0.959 | 0.914 | 0.798 | 45 | 4 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox treo / thừa | 81–100 | ID 6 | Bấm outside đúng frame xe rời khung (xóa box thừa trước khi track chính thức xuất hiện) |
| Bbox treo / thừa | 139–144 | ID 5 | Bấm outside đúng frame xe rời khung (xóa box tồn dư sau khi xe đã rời khung) |
| Bbox treo / thừa | 149–152 | ID 4 | Bấm outside đúng frame xe rời khung (xóa box tồn dư sau khi xe đã rời khung) |
| Bbox treo / thừa | 51–53 | ID 4 | Bấm outside đúng frame xe rời khung (xóa box thừa trước khi track xuất hiện) |
| Bbox treo / thừa | 169–171 | ID 8 | Bấm outside đúng frame xe rời khung (xóa box tồn dư sau khi xe đã rời khung) |
| Bbox trôi | 155 | ID 6 | Thêm keyframe quanh đây vì IoU giảm xuống 0.51 |
| Bbox trôi | 81–83 | ID 5 | Thêm keyframe quanh đây vì IoU giảm xuống 0.52–0.53 |
| Bbox trôi | 167–168 | ID 8 | Thêm keyframe quanh đây vì IoU giảm xuống 0.53–0.54 |
| Bbox trôi | 101, 104 | ID 6 | Thêm keyframe quanh đây vì IoU giảm xuống 0.53–0.55 |
## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `Python / ultralytics / torch / lap` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml & botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.70 / 960 / [2, 5, 7]` |
| device | `0 (GPU) hoặc cpu` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.742 | 0.726 | 0.760 | 0.825 | 0.959 | 0.914 | 0.798 | 45 | 4 | 0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 | 91 | 26 | 2 |
| ReID vs bạn | 0.669 | 0.616 | 0.731 | 0.822 | 0.856 | 0.709 | 0.801 | 101 | 77 | 1 |
## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

- **So sánh số liệu:** Trong nhãn thủ công của bạn (bạn vs gold), MOTA đạt `0.914` còn IDF1 đạt `0.959` (MOTA thấp hơn IDF1). Tuy nhiên, khi xét các model tự động (ví dụ ByteTrack: MOTA `0.749` so với IDF1 `0.875`), MOTA cũng thấp hơn đáng kể so với IDF1.
- **Ý nghĩa khi MOTA cao mà IDF1 thấp:** Nếu một hệ thống có MOTA cao nhưng IDF1 thấp, điều đó chứng tỏ model **phát hiện và định vị đối tượng rất tốt** (ít bỏ sót FN và ít sinh nhiễu FP), nhưng **khả năng duy trì nhất quán ID rất kém** (thường xuyên bị đổi ID giữa chừng hoặc cắt nhỏ track).
- **Vì sao MOTA không phạt nặng lỗi ID:** MOTA (Multiple Object Tracking Accuracy) tính tổng các lỗi False Positives (FP), False Negatives (FN) và ID Switches (IDSW) chia cho tổng số đối tượng ground truth. Trong công thức này, mỗi lần đổi ID chỉ được tính là 1 lỗi IDSW đơn lẻ, trong khi lỗi thiếu khung (FN) hoặc thừa khung (FP) diễn ra liên tục trên từng frame và chiếm trọng số lớn hơn nhiều trong tổng số phạt. Do đó, MOTA ít nhạy cảm với lỗi sai lệch định danh (identity) hơn so với IDF1 (vốn tập trung đánh giá độ khớp tuyệt đối của ID).

---

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- **Sự khác biệt về chỉ số:** 
  - **IDF1:** Tăng từ `0.875` (ByteTrack) lên `0.900` (ReID).
  - **AssA (Association Accuracy):** Tăng rõ rệt từ `0.776` lên `0.820`.
  - **IDSW (ID Switch):** Cả hai đều giữ nguyên mức `2` ID switch.
- **Phân tích qua chuỗi frame:** Ở đoạn xe bị che khuất hoặc giao cắt (ví dụ quanh frame 87 với track gold 5), ByteTrack dễ bị nhầm lẫn tính liên tục do chỉ dựa vào kinematic (chuyển động học), dẫn đến việc FN cao (54 xuống còn 26 ở ReID). BoT-SORT tích hợp thêm nhánh trích xuất đặc trưng ngoại hình (ReID appearance embedding) giúp nhận diện lại chính xác chiếc xe sau khi bị che khuất, kéo AssA và IDF1 lên cao hơn. 
- *Lưu ý quan trọng:* Kết quả này **không cô lập hoàn toàn hiệu ứng nhân quả (causal effect) của riêng ReID**, vì ByteTrack và BoT-SORT có sự khác biệt về toàn bộ cấu trúc cài đặt tracker bên dưới (tracker implementation), chứ không chỉ khác biệt mỗi môđun ReID.

---

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- **Sự thay đổi của các chỉ số:** Khi chuyển từ ByteTrack sang BoT-SORT + ReID, **DetA** tăng từ `0.649` lên `0.711`; **FN** giảm mạnh từ `54` xuống `26` (giảm hơn một nửa); tuy nhiên **FP** tăng nhẹ từ `88` lên `91`.
- **Đánh giá bản chất lỗi còn lại:** Lỗi còn lại của hệ thống thiên nhiều về **detector (khả năng phát hiện và khoanh bounding box)** hơn là association. Mặc dù AssA đã đạt mức rất tốt (`0.820`) và IDSW thấp (`2`), chỉ số DetA (`0.711`) và số lượng FP còn khá cao (`91`) cho thấy model vẫn gặp khó khăn trong việc lọc nhiễu nền hoặc nhận diện chính xác biên cạnh của vật thể ở xa.

---

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- **Frame & ID:** Khoảng **frame 87 – frame 113**, đặc biệt với **track gold 5 (bị model tách thành ID [18, 17])**.
- **Nguyên nhân:** Khi hai xe đi cắt nhau gây che khuất cục bộ, mô hình ReID bị nhẫm lẫn đặc trưng ngoại hình (do góc quay thay đổi hoặc ánh sáng mờ) nên đã ngắt track cũ và gán sang ID mới. Trong khi đó, **bạn (nhãn thủ công)** nắm được ngữ cảnh toàn cục (global context) của video, nhận biết rõ đây vẫn là một chiếc xe đang di chuyển liên tục nên giữ nguyên một ID xuyên suốt từ đầu đến cuối một cách chính xác.

---

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- **Phân tích qua Evidence:** Khi đối chiếu kết quả `eval_reid_vs_me.json`, ta thấy độ phủ thời lượng (coverage) của nhãn thủ công ở một số track bị ngắn hơn thực tế (ví dụ track bản A 6 mới phủ 38/76 frame tương ứng 50%, track 5 phủ 51/68 frame tương ứng 75%). 
- **Lý do xem lại annotation:** Model ReID đã phát hiện ra các phần đuôi/đầu của xe ở các frame rìa (khi xe mớiớm xuất hiện hoặc sắp rời khung hình với tỉ lệ hiển thị rất nhỏ < 30%). Điều này chỉ ra rằng nhãn thủ công của bạn đôi khi **quên bấm Outside quá sớm hoặc bắt đầu gán muộn**, trong khi model nhạy bén hơn ở các frame biên, giúp bạn rút kinh nghiệm bổ sung quy định chặt chẽ hơn về điểm đầu/cuối của track trong các lần gán nhãn tiếp theo.
## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- **Sửa trong `GUIDELINE_MINI.md`:**
  1. *Quy định rõ ràng về Entry/Exit:* Bổ sung điều khoản cụ thể: "Chỉ bắt đầu tạo track khi nhìn thấy rõ $\ge 30\%$ thân xe; kết thúc track (Outside) ngay khi phần thân xe còn lại trong khung $< 15\%$".
  2. *Quy định ngưỡng kiểm tra xe dừng đỗ:* Thêm quy tắc: nếu bounding box đứng im $\ge 20$ frame liên tiếp, bắt buộc phải tua tới lui 5 frame để xác nhận xe dừng thật hay người gán quên đặt Outside.
  3. *Quy định kích thước tối thiểu:* Các phương tiện ở quá xa có kích thước $< 20 \times 15$ pixel sẽ không gán để đảm bảo tính khả thi cho detector.

- **Đổi trong quy trình làm việc:**
  1. Luôn xem lướt (preview) toàn bộ video 1–2 lần trước khi đặt bounding box đầu tiên để nắm được số lượng đối tượng, hướng di chuyển và các điểm giao cắt phức tạp.
  2. Áp dụng nghiêm ngặt quy trình tự kiểm 3 lượt tua (Lượt 1: ID timeline $\rightarrow$ Lượt 2: Điểm vào/ra khung hình $\rightarrow$ Lượt 3: Độ ôm sát và drift ở các frame giữa).

## 7. Tệp đã nộp

- [v] `annotations/clip_01/gt.txt`
- [v] `annotations/clip_02/gt.txt`
- [v] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [v] `GUIDELINE_MINI.md` đã điền
- [v] `outputs/eval_vs_gold.json`
- [v] `outputs/model_bytetrack_clip_01.txt`
- [v] `outputs/model_reid_clip_01.txt`
- [v] `outputs/model_run_config.json`
- [v] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [v] `reports/review_partner.md`
- [v] `reports/REPORT.md` (file này)
