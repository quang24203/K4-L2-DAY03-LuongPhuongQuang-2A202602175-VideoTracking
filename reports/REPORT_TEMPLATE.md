# Báo cáo Ngày 3 — Tracking Annotation

**Họ tên / nhóm:** Lương Phương Quang (K4-L2-DAY03)
**Ngày:** 2026-09-15

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán clip_02 (warm-up) | 30 phút |
| Thời gian gán clip_01 | 60 phút |
| Số track đã vẽ trong clip_01 | 8 |
| Số keyframe trung bình mỗi track | chưa cung cấp |

**Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:**
1. Một số bbox bị cảnh báo đứng im trong nhiều frame; cần xem lại bằng mắt để phân biệt xe đứng yên với quên bấm outside.
2. Một số bbox bị lệch ở frame 54, 101-108 và 136; cần thêm keyframe quanh các đoạn này.
3. Có bbox thừa trước hoặc sau thời gian sống tham chiếu ở các đoạn frame 51-53, 68-78, 81-100, 149-151 và 169-171.

## 2. Tự kiểm

**Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):**
- Lượt 1: Validator đạt 0 lỗi; có 3 cảnh báo bbox đứng im ở track 1, cần kiểm tra lại bằng mắt.
- Lượt 2: Kiểm tra frame đầu/cuối và đối chiếu với gold/gt.txt, phát hiện bbox thừa ở các đoạn frame 51-53, 68-78, 81-100, 149-151 và 169-171.
- Lượt 3: Các bbox lệch đáng chú ý gồm frame 54, 101-108 và 136; cần xem lại các keyframe lân cận.

Các finding ở phần dưới là kết quả tự kiểm và đối chiếu tự động với gold reference; không ghi thông tin kiểm chéo.

## 3. Pre-gold lock và chấm kết quả annotation

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ evidence/pre-gold/clip_01/manifest.json | b4ae14717876d1efc5256fa36d89ca300caec586e3c25edfd3960b64409565df |
| Thời điểm khóa | 2026-09-15T10:40:40.906417+00:00 |
| Số row / frame / track trước khi mở reference | 622 / 190 / 8 |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| **Bản pre-gold** | chưa có `eval_pre_gold.json` | | | | | | | | | |
| **Annotation của bạn vs gold** (`outputs/eval_vs_gold.json`) | 0.768 | 0.754 | 0.783 | 0.863 | 0.949 | 0.893 | 0.844 | 55 | 6 | 0 |

**Qua cổng (IDF1 >= 0.80, MOTA >= 0.75, MOTP >= 0.70):** CÓ

**Finding từ kết quả của bạn khi đối chiếu với gold, không phải nhãn gold:**

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox thừa trong annotation của bạn | 51-53, 68-78, 81-100, 149-151, 169-171 | 4, 5, 6, 8 | Kiểm tra lại outside ở điểm bắt đầu/kết thúc track. |
| Bbox lệch trong annotation của bạn | 54, 101-108, 136 | 4, 6, 7, 8 | Thêm keyframe quanh các frame có IoU thấp. |
| Cảnh báo bbox đứng im | 12-26, 46-60, 143-157 | 1 | Xem lại video; chỉ sửa nếu thực sự quên outside. |

## 4. Kết quả model: ByteTrack control vs ReID treatment

**Cấu hình từ outputs/model_run_config.json:**

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | Python 3.13.15 / Ultralytics 8.4.145 / Torch 2.11.0+cu128 / lap 0.5.13 |
| weights / hai tracker | yolo26n.pt / ByteTrack (bytetrack.yaml) / BoT-SORT + ReID (botsort-reid.yaml) |
| conf / IoU / imgsz / classes | 0.25 / 0.7 / 960 / 2, 5, 7 |
| device | GPU 0; persist=true; 190 frame |

**So sánh các metrics:**

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| **bạn vs gold** | 0.768 | 0.754 | 0.783 | 0.863 | 0.949 | 0.893 | 0.844 | 55 | 6 | 0 |
| **ByteTrack control vs gold** | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| **BoT-SORT + ReID vs gold** | 0.764 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.859 | 91 | 26 | 2 |
| **ReID vs bạn** | 0.724 | 0.669 | 0.787 | 0.867 | 0.876 | 0.751 | 0.853 | 85 | 69 | 1 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**
MOTA (0.893) thấp hơn IDF1 (0.949). Annotation giữ identity khá tốt và không có IDSW, nhưng vẫn có FP/FN và bbox chưa khớp hoàn toàn. MOTA cộng FP, FN và ID switch theo từng detection/frame; vì vậy lỗi identity có thể không bị phạt nặng như IDF1 khi một ID bị đổi hoặc bị tách trong thời gian dài.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**
BoT-SORT + ReID cao hơn ByteTrack ở HOTA (0.764 so với 0.709), DetA (0.711 so với 0.649), AssA (0.820 so với 0.776), IDF1 (0.900 so với 0.875) và MOTA (0.792 so với 0.749). ReID vẫn có 2 IDSW, bằng ByteTrack, nhưng các lỗi xảy ra ở frame 87 và 113; ByteTrack đổi ID ở frame 59 và 94. Vì hai tracker implementation khác nhau, đây không phải phép đo causal effect riêng của ReID.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**
So với gold, ByteTrack có DetA 0.649, 88 FP và 54 FN; BoT-SORT + ReID có DetA 0.711, 91 FP và 26 FN. Như vậy ReID giảm FN mạnh nhưng tăng FP nhẹ, cho thấy coverage/detection cải thiện nhưng vẫn có bbox thừa. AssA của ReID cũng cao hơn ByteTrack (0.820 so với 0.776), nên lỗi còn lại là cả detector/coverage lẫn association, không chỉ một phía.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**
Ở frame 59, ByteTrack đổi ID của gold track 4 (ID 14 sang 15), trong khi ReID không báo IDSW tại frame này. Đây là một ví dụ ReID xử lý identity tốt hơn ByteTrack; tuy nhiên cần xem trực tiếp frame để kết luận bbox nào đúng.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**
ReID vẫn đổi ID ở frame 87 với gold track 5 và frame 113 với gold track 6; ngoài ra có các track bị phân mảnh. Vì vậy nên xem lại các đoạn quanh frame 87 và 113, nhưng không dùng output model làm nhãn thay thế. So với nhãn của bạn, ReID đạt IDF1 0.876 và MOTA 0.751, cho thấy model còn nhiều khác biệt về coverage (69 FN) dù có 1 IDSW.

## 6. Nếu phải gán thêm 10 clip nữa

**Bạn sẽ sửa gì trong GUIDELINE_MINI.md, và đổi gì trong quy trình làm việc của mình?**
Bổ sung quy tắc rõ ràng về outside khi xe rời khung, cách giữ ID khi xe bị che, và khoảng cách tối đa giữa các keyframe ở đoạn chuyển động nhanh. Quy trình nên luôn gồm validator, visual review ba lượt, kiểm tra các cảnh báo, rồi mới khóa pre-gold.

## 7. Tệp đã nộp

- [x] annotations/clip_01/gt.txt
- [x] annotations/clip_02/gt.txt
- [x] evidence/pre-gold/clip_01/gt.txt và manifest.json
- [x] GUIDELINE_MINI.md đã điền
- [x] outputs/eval_vs_gold.json
- [x] outputs/model_bytetrack_clip_01.txt
- [x] outputs/model_reid_clip_01.txt
- [x] outputs/model_run_config.json
- [x] outputs/eval_bytetrack_vs_gold.json, outputs/eval_reid_vs_gold.json, outputs/eval_reid_vs_me.json
- [x] reports/review_partner.md (gold QC tự động)
- [x] reports/REPORT.md (file này)