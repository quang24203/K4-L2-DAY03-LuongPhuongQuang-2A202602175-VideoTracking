# Gold QC — Day 3

Tổng hợp QC tự động từ validator và các file kết quả so với `gold/gt.txt`.

| Trường | Giá trị |
| --- | --- |
| Author | Lương Phương Quang |
| Reviewer | Gold QC tự động |
| Pair ID | N/A |
| CVAT version | chưa cung cấp |
| Thời điểm review | 2026-09-15 |

## Danh sách finding

Mỗi finding ghi `frame + ID + lỗi + cách sửa`, dựa trên kết quả đối chiếu gold.

| # | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát + rule áp dụng | Cách sửa đề xuất | Closure: fixed / not-a-defect / needs-review |
| ---: | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | 51-53 | 51-53 | 4 | Bbox thừa ở đầu track | Gold không có track 4 ở đoạn này; rule: bắt đầu track tại frame đầu tiên xác định được xe. | Kiểm tra frame đầu, bấm outside trước khi xe xuất hiện. | needs-review |
| 2 | 81-100 | 81-100 | 6 | Bbox thừa ở đầu track | Gold xác định ID 6 chưa xuất hiện trong đoạn này; có 20 frame ghost. | Xác định lại frame bắt đầu của ID 6. | needs-review |
| 3 | 54-55 | 54-55 | 4 | Bbox lệch | IoU khoảng 0.501 và 0.584; ID vẫn khớp nên đây là lỗi hình học. | Thêm keyframe quanh đoạn 54-55, giữ nguyên ID. | needs-review |
| 4 | 101-108 | 101-108 | 6 | Bbox lệch | Các frame 101, 102, 106, 107, 108 có IoU thấp. | Thêm keyframe cục bộ và kiểm tra interpolation. | needs-review |
| 5 | 149-151 | 149-151 | 4 | Bbox thừa ở cuối track | Gold cho thấy track 4 đã rời khung nhưng bbox vẫn còn. | Bấm outside đúng frame xe ra khỏi khung. | needs-review |
| 6 | 169-171 | 169-171 | 8 | Bbox thừa ở cuối track | Gold cho thấy track 8 đã rời khung nhưng bbox vẫn còn. | Kiểm tra frame cuối và bấm outside. | needs-review |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | 8 track; validator không báo lỗi |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | So với gold: IDSW 0 |
| Occlusion ngắn giữ ID; crossing không đổi ID | N/A | Không có dữ liệu trực tiếp |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING | ID 4, 6, 8; các đoạn trong finding |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | FINDING | Frame 54-55 và 101-108 |
| Frame giữa hai keyframe không bị interpolation drift | FINDING | IoU thấp ở frame 54-55, 101-108 |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | 622 rows, frame 1..190, 8 track |
| Mọi finding có cách sửa và closure do tác giả điền | N/A | Closure cần xác nhận sau khi sửa trong CVAT |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | Annotation vs gold: IDSW 0 |
| 2 — endpoint/scope | FINDING | Bbox thừa ở frame 51-53, 81-100, 149-151, 169-171 |
| 3 — geometry/interpolation | FINDING | IoU thấp ở frame 54-55 và 101-108 |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: bbox thừa ở frame 81-100, ID 6; track phải bắt đầu khi xe đầu tiên được xác định rõ.
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do: cảnh báo bbox đứng im có thể là xe thật sự đứng yên; cần xem video trước khi kết luận.
3. Một rule cần Lab Coach làm rõ: thời điểm bấm outside khi xe sát rìa khung và cách xử lý xe đứng yên lâu.