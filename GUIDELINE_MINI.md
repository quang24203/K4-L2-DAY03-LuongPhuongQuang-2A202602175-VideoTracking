# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Lương Phương Quang`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): Không gán người, xe đạp, xe máy/mô tô, biển báo hoặc xe trong quảng cáo/gương.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (2 giây @ 12.5 fps) | Vẫn là cùng một xe và cần bảo toàn identity. |
| Xe bị che lâu hơn ngưỡng trên | mở track mới | Khoảng mất dấu dài làm identity không còn đủ chắc chắn. |
| Xe rời khung hình rồi quay lại | **track mới** | Xe đã ra khỏi khung được xem là kết thúc track cũ. |
| Hai xe cắt nhau / chồng lên nhau | giữ ID riêng; đặt keyframe dày trước, trong và sau lúc giao nhau | Không đổi ID chỉ vì hai bbox chồng lấn. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; không gán khi chưa phân biệt được với vật thể ngoài schema |
| Xe đang đỗ, không di chuyển | vẫn gán và giữ track trong toàn bộ thời gian xe còn trong khung; kiểm tra cảnh báo bbox đứng im trước khi kết luận là lỗi |
| Keyframe đặt dày ở đâu | đặt dày khi xe rẽ, phanh, bị che, cắt nhau, gần rìa ảnh hoặc bbox interpolation bị lệch; đặt thêm quanh frame 54, 101-108 và 136 khi kiểm tra clip_01 |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01`, frame 51-53, ID 4
- Tình huống: bbox xuất hiện trước thời điểm track tham chiếu bắt đầu.
- Quyết định: kiểm tra lại frame bắt đầu và bấm outside trước khi xe thực sự xuất hiện.
- Lý do: chấm gold xác định đây là bbox thừa, không phải một đoạn nhìn thấy hợp lệ của xe.

### Ca 2
- Clip / frame / ID: `clip_01`, frame 81-100, ID 6
- Tình huống: track có bbox trước khi track tham chiếu 6 xuất hiện.
- Quyết định: xác định lại frame đầu tiên nhìn thấy xe; không kéo track ngược về các frame chưa có xe.
- Lý do: đoạn 20 frame bị chấm là ghost/bbox thừa; quy tắc bắt đầu track phải dựa trên frame đầu tiên xác định được xe.

### Ca 3
- Clip / frame / ID: `clip_01`, frame 54-55, ID 4
- Tình huống: bbox vẫn cùng ID nhưng bị lệch so với xe; IoU lần lượt khoảng 0.501 và 0.584.
- Quyết định: thêm keyframe quanh đoạn chuyển động và kiểm tra lại interpolation ở giữa hai keyframe.
- Lý do: lỗi này không phải ID switch; cần sửa hình học bbox mà không đổi track_id.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Phải kiểm tra riêng frame bắt đầu và kết thúc của từng track; không để bbox xuất hiện trước khi xe vào khung hoặc còn treo sau khi xe ra khỏi khung.
- Khi bbox đứng im nhiều frame, phải xem video để phân biệt xe thật sự đứng yên với quên bấm outside; không tự động xóa track chỉ vì cảnh báo validator.
- Khi xe rẽ, bị che, cắt nhau hoặc interpolation lệch, phải thêm keyframe cục bộ; không đổi ID nếu lỗi chỉ nằm ở vị trí bbox.
