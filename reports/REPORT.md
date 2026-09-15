# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: **Nguyễn Đức Anh — solo**  
Ngày: **15/09/2026**

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT |
| Thời gian gán `clip_02` (warm-up) | khoảng 10 phút |
| Thời gian gán `clip_01` | khoảng 45 phút |
| Số track đã vẽ trong `clip_01` | 8 |
| Số keyframe trung bình mỗi track | khoảng 6 keyframe/track |

Ba tình huống khó nhất khi gán clip này, và cách tôi xử lý:

1. Một số xe mới đi vào khung hình còn nhỏ và hơi mờ nên khó xác định frame bắt đầu. Tôi tua chậm và chỉ bắt đầu track từ frame đầu tiên mà tôi nhận ra chắc chắn đó là xe bốn bánh.
2. Có những đoạn xe bị che hoặc các xe đi gần nhau nên khá dễ nhầm ID. Tôi kiểm tra các frame trước và sau đoạn bị che, dựa vào hướng di chuyển và vị trí để giữ ID cũ nếu vẫn là cùng một xe.
3. Bbox nội suy bị trôi ở các đoạn xe đổi tốc độ hoặc hướng chuyển động. Tôi thêm keyframe ở giữa đoạn và chỉnh bbox ôm sát phần xe nhìn thấy được.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Tôi kiểm tra ID của 8 track, đặc biệt ở các đoạn xe đi gần nhau. Không thấy trường hợp đổi ID giữa chừng trong bản cuối.
- Lượt 2: Tôi phát hiện một số track bắt đầu quá sớm hoặc kết thúc quá muộn, rõ nhất là ID 5, 6 và 8.
- Lượt 3: Tôi thấy bbox bị trôi ở một số frame giữa, nhất là quanh frame 79, 96 và 102–106.

Kiểm chéo với: **không có — tôi làm solo**. Vì vậy không có file nhận xét từ thành viên cùng nhóm.  
Số lỗi tôi tìm được trong bản của bạn cùng nhóm: **không áp dụng**. Số lỗi bạn cùng nhóm tìm được trong bản của tôi: **không áp dụng**.

Vì làm solo nên không có trường hợp hai người quyết khác nhau. Tuy nhiên, sau khi tự kiểm tôi thấy `GUIDELINE_MINI.md` cần nói rõ hơn về frame bắt đầu/kết thúc track, cách xử lý xe bị che và vị trí cần đặt thêm keyframe khi chuyển động thay đổi.

## 3. Chấm với gold — trước và sau rework

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Lần chấm đầu | 0.773 | 0.753 | 0.798 | 0.858 | 0.948 | 0.890 | 0.842 | 62 | 1 | 0 |
| Sau rework | — | — | — | — | — | — | — | — | — | — |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**

Notebook hiện mới lưu kết quả của một lần chấm nên tôi chưa điền số liệu sau rework. Sau khi đọc danh sách lỗi, các vị trí tôi xác định cần sửa là:

| Loại lỗi | Frame | ID | Cách sửa |
| --- | --- | --- | --- |
| Bbox thừa trước khi xe xuất hiện | 58–78 | 5 | Đặt lại frame bắt đầu và đánh dấu outside trước khi xe thực sự xuất hiện |
| Bbox thừa trước khi xe xuất hiện | 79–100 | 6 | Cắt phần bbox nội suy bị treo và bắt đầu track đúng frame xe xuất hiện |
| Bbox trôi | 102–104 | 6 | Thêm keyframe quanh đoạn này và chỉnh bbox ôm sát xe |
| Bbox trôi | 106 | 7 | Thêm keyframe để bbox không bị lệch khi nội suy |
| Bbox thừa sau khi xe rời khung | 169–171 | 8 | Kết thúc track đúng frame xe rời khỏi khung hình |

## 4. Kết quả model và so sánh ba chiều

Cấu hình: model `yolo26n.pt`, tracker `bytetrack.yaml`, conf `0.25`, imgsz `960`

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.773 | 0.753 | 0.798 | 0.858 | 0.948 | 0.890 | 0.842 | 62 | 1 | 0 |
| model vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 | 88 | 54 | 2 |
| model vs bạn | 0.630 | 0.575 | 0.693 | 0.819 | 0.830 | 0.675 | 0.791 | 88 | 115 | 3 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi là 0.890, thấp hơn IDF1 là 0.948. Kết quả này cho thấy phần giữ ID của tôi khá ổn vì không có ID switch, nhưng vẫn còn nhiều bbox thừa làm MOTA giảm. Nếu một bài có MOTA cao nhưng IDF1 thấp thì thường detector vẫn tìm được phần lớn vật thể nhưng liên kết ID qua các frame chưa tốt. MOTA cộng lỗi ID switch cùng FP và FN rồi chia theo tổng số bbox gold, nên một vài lần đổi ID có thể bị “loãng” khi clip có nhiều bbox. Trong khi đó IDF1 đánh giá trực tiếp tính nhất quán của ID trên toàn bộ quãng đời track nên nhạy hơn với lỗi này.

**2. DetA và AssA của model lệch nhau bao nhiêu? Cái nào kéo HOTA xuống — model không tìm ra xe, hay tìm ra rồi nhưng đánh mất ID?**

DetA của model là 0.649 và AssA là 0.776, lệch nhau 0.127. Vì DetA thấp hơn rõ rệt nên yếu tố kéo HOTA xuống chủ yếu là khâu phát hiện: model còn bỏ sót xe và tạo bbox thừa. Khả năng giữ ID tốt hơn phần phát hiện, dù vẫn có 2 lần ID switch ở frame 59 và 94.

**3. Một chỗ bạn đúng và model sai (frame, ID, vì sao):**

Ở frame 94, xe mang ID 5 trong bản của tôi vẫn được giữ liên tục, còn ByteTrack chuyển từ ID 23 sang ID 32. Khi xem các frame liền trước và sau, vị trí và hướng di chuyển cho thấy đây vẫn là cùng một xe, vì vậy việc tôi giữ nguyên ID là hợp lý hơn.

**4. Một chỗ model đúng và bạn sai (frame, ID, vì sao):**

Ở frame 102, bbox của ID 6 trong bản tôi bị trôi và chỉ đạt IoU khoảng 0.56 so với gold. Bbox của model ở đoạn này bám theo vị trí xe tốt hơn. Nguyên nhân là tôi đặt keyframe hơi thưa nên nội suy không theo kịp chuyển động của xe.

**5. Trong ba loại bất đồng giữa bạn và model, loại nào nhiều nhất? Nó nói gì về clip này?**

Loại xuất hiện nhiều nhất là bbox chỉ có ở một bên. Ví dụ, ở frame 111 có 3 bbox chỉ model có và 2 bbox chỉ tôi có. Ngoài ra còn nhiều frame liên tiếp từ 106 đến 110 có cùng dạng bất đồng. Điều này cho thấy phần khó nhất của clip không hẳn là xe cắt nhau, mà là quyết định xe nào thực sự cần gán khi xe nhỏ, bị che hoặc nằm gần rìa ảnh. Model cũng có xu hướng nhận nhầm vật thể tĩnh thành xe, thể hiện qua các track thừa như ID 10 và 41.

## 6. Nếu phải gán thêm 10 clip nữa

Tôi sẽ bổ sung vào `GUIDELINE_MINI.md` ba luật rõ hơn: chỉ bắt đầu track khi xác định chắc chắn là xe bốn bánh; kết thúc track ngay tại frame xe rời khung hoặc không còn nhìn thấy; và thêm keyframe tại những đoạn xe đổi hướng, đổi tốc độ hoặc sắp bị che. Với xe bị che dưới 25 frame, tôi sẽ giữ nguyên ID nếu có thể xác nhận bằng vị trí và hướng di chuyển; nếu xe đã rời khung rồi xuất hiện lại thì tạo ID mới.

Về quy trình, tôi sẽ gán theo từng track từ đầu đến cuối, sau đó kiểm tra ba lượt riêng cho ID, điểm bắt đầu/kết thúc và bbox ở các frame giữa. Tôi cũng sẽ chạy kiểm tra sớm sau vài track thay vì đợi gán xong toàn bộ clip, vì như vậy các lỗi bbox treo và keyframe thưa sẽ được phát hiện sớm hơn.

## 7. Tệp đã nộp

- [X] `annotations/clip_01/gt.txt`
- [X] `annotations/clip_02/gt.txt`
- [X] `GUIDELINE_MINI.md` đã điền
- [X] `outputs/eval_vs_gold.json`
- [X] `outputs/model_clip_01.txt`
- [X] `outputs/eval_model_vs_gold.json`, `outputs/eval_model_vs_me.json`
- [ ] `reports/review_partner.md` — không áp dụng vì làm solo
- [X] `reports/REPORT.md` (file này)
