# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: **Nguyễn Đức Anh — solo**  
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

Bổ sung của cá nhân: chỉ gán phương tiện thật đang xuất hiện trong cảnh. Không gán hình xe trên biển quảng cáo, vật thể tĩnh có hình dạng giống xe hoặc xe chỉ xuất hiện trong ảnh phản chiếu. Nếu vật thể quá nhỏ hoặc mờ đến mức chưa thể chắc chắn là xe bốn bánh thì chờ đến frame nhìn rõ hơn mới bắt đầu track.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của cá nhân | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che **dưới 25 frame**, đồng thời hướng di chuyển và vị trí sau khi hiện lại vẫn hợp lý | 25 frame tương đương khoảng 2 giây; trong khoảng này vẫn có thể theo dõi được quỹ đạo của xe |
| Xe bị che lâu hơn ngưỡng trên | Tạo ID mới, trừ khi có dấu hiệu rất rõ để xác nhận chắc chắn đó vẫn là cùng một xe | Khi mất dấu quá lâu, khả năng nhầm với xe khác tăng lên |
| Xe rời khung hình rồi quay lại | Tạo **track mới** | Sau khi xe đã rời hẳn khung hình thì không còn đủ bằng chứng để nối với track cũ |
| Hai xe cắt nhau / chồng lên nhau | Xem các frame trước và sau đoạn giao nhau, bám theo hướng đi, vị trí và đặc điểm của từng xe; không đổi ID chỉ vì hai bbox chồng lên nhau | Cách này giúp tránh đổi nhầm ID khi một xe che xe còn lại trong thời gian ngắn |

## 3. Luật bbox

| Tình huống | Luật của cá nhân |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định chắc chắn được đó là xe bốn bánh; ngưỡng tôi chọn là xe có chiều rộng và chiều cao ít nhất khoảng 12 pixel và nhìn ra được thân xe |
| Xe đang đỗ, không di chuyển | Vẫn gán nếu đó là xe thật nằm trong phạm vi cảnh; giữ nguyên ID trong suốt thời gian xe còn nhìn thấy |
| Keyframe đặt dày ở đâu | Đặt dày hơn ở đoạn xe mới xuất hiện, sắp rời khung, đổi hướng, đổi tốc độ, bị che hoặc có bbox nội suy bắt đầu trôi; ở đoạn chuyển động đều có thể đặt thưa hơn |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

### Ca 1
- Clip / frame / ID: `clip_01`, khoảng frame 79, ID 5
- Tình huống: Bản gán có bbox trước khi xe thực sự xuất hiện rõ, do nội suy từ keyframe sau kéo bbox về các frame trước.
- Quyết định: Xóa phần bbox thừa và bắt đầu track tại frame đầu tiên có thể xác định chắc chắn là xe.
- Lý do: Nếu bắt đầu quá sớm thì các bbox không có vật thể thật sẽ bị tính là false positive.

### Ca 2
- Clip / frame / ID: `clip_01`, frame 102–104, ID 6
- Tình huống: Xe thay đổi vị trí nhanh hơn so với đường nội suy nên bbox bị trôi, IoU ở frame 102 chỉ còn khoảng 0.56.
- Quyết định: Thêm keyframe quanh frame 102–104 và chỉnh bbox ôm sát phần xe nhìn thấy.
- Lý do: Giữ keyframe cũ quá thưa làm bbox nằm lệch khỏi xe ở các frame giữa.

### Ca 3
- Clip / frame / ID: `clip_01`, frame 169–171, ID 8
- Tình huống: Xe đã rời khỏi khung nhưng bbox vẫn còn tồn tại thêm vài frame.
- Quyết định: Kết thúc track đúng frame cuối cùng xe còn nhìn thấy và đặt outside ngay sau đó.
- Lý do: Bbox tồn tại sau khi xe rời khung sẽ trở thành bbox treo và làm tăng false positive.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Tôi làm bài **solo** nên không có bước kiểm chéo với thành viên khác. Sau khi tự kiểm và chấm với gold, tôi bổ sung các luật sau:

- Phải kiểm tra riêng frame đầu và frame cuối của từng track; đặt outside ngay sau frame cuối xe còn nhìn thấy, không để nội suy kéo bbox sang đoạn không có xe.
- Khi bbox nội suy có IoU thấp hoặc không còn bám theo xe, phải thêm keyframe ở giữa, đặc biệt tại đoạn xe đổi hướng, đổi tốc độ hoặc bị che.
- Xe bị che dưới 25 frame chỉ được giữ nguyên ID khi vị trí và hướng di chuyển trước/sau đoạn che khớp nhau; nếu xe đã rời hẳn khung thì khi quay lại luôn tạo ID mới.
- Sau khi gán xong phải tua ba lượt riêng: một lượt kiểm tra ID, một lượt kiểm tra đầu/cuối track và một lượt kiểm tra bbox ở các frame giữa.
