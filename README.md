# Bài thực hành Ngày 3 - Gán nhãn tracking cho video giao thông

Ngày 2 bạn vẽ bbox trên ảnh tĩnh. Hôm nay dữ liệu có thêm một chiều nữa: **thời gian**.
Trong bốn giờ, bạn sẽ gán nhãn tracking cho một clip 15 giây, kiểm tra chất lượng
nhãn bằng gold set, rồi cho một hệ thống tracking tự động chạy trên đúng clip đó và
so kết quả của model với nhãn của mình:

```text
clip video -> CVAT Track Mode -> export MOT 1.1 -> QC với gold set
           -> YOLO + ByteTrack chạy cùng clip -> so ba chiều -> báo cáo
```

Nhãn tracking = nhãn detection + một cột: `track_id`. Cột đó máy **không suy ra
được từ một frame riêng lẻ** — nó là quyết định của bạn, và nó là thứ được chấm
hôm nay.

## Mục tiêu học tập

Sau lab, bạn có thể:

1. Gán nhãn tracking trong CVAT **Track Mode**, dùng keyframe + interpolation đúng chỗ.
2. Xử lý ba tình huống khó của video: vật thể **bị che tạm thời**, **vào/ra khung hình**,
   và **hai vật thể cắt nhau**.
3. Export đúng định dạng **MOT 1.1** — định dạng giữ được `track_id`.
4. Đọc và dùng **HOTA, DetA, AssA, IDF1, MOTA** để tìm lỗi trong nhãn của chính mình,
   chứ không chỉ để biết điểm cao hay thấp.
5. Chạy pipeline **tracking-by-detection** (YOLO + ByteTrack) và giải thích được
   chỗ model và người không đồng ý với nhau.

## Bài nộp

| Tệp | Nội dung |
| --- | --- |
| `annotations/clip_01/gt.txt` | nhãn tracking của bạn cho clip chính (export MOT 1.1) |
| `annotations/clip_02/gt.txt` | nhãn clip warm-up |
| `GUIDELINE_MINI.md` | luật ID của nhóm bạn + ít nhất ba ca mơ hồ đã gặp và cách quyết |
| `outputs/eval_vs_gold.json` | kết quả chấm nhãn với gold (sau khi giảng viên phát) |
| `outputs/model_clip_01.txt` | kết quả tracking của model, định dạng MOT |
| `outputs/eval_model_vs_gold.json`, `outputs/eval_model_vs_me.json` | hai phép so còn lại |
| `reports/REPORT.md` | báo cáo, điền từ `reports/REPORT_TEMPLATE.md` |
| `reports/review_partner.md` | danh sách lỗi tìm được trong clip của bạn cùng nhóm + reviewer checklist |

Đọc [GUIDE.md](GUIDE.md) theo thứ tự thao tác và đối chiếu [RUBRIC.md](RUBRIC.md)
trước khi nộp.

## Cấu trúc thư mục

```text
Day3-Lab/
  data/clips/clip_01/     190 frame, 8 track — BÀI CHÍNH, không có gold khi pull
  data/clips/clip_02/      60 frame, 6 track — warm-up, gold có sẵn ở gt/gt.txt
  annotations/            nhãn của bạn đặt ở đây
  gold/                   trống; giảng viên phát gold clip_01 vào đây ở mốc 2:35
  tools/                  check / evaluate / visualize / run_tracker
  notebooks/              notebook Colab chạy YOLO + ByteTrack
  reports/                mẫu báo cáo
  outputs/                kết quả chấm và kết quả model
```

Không đổi tên/đổi thứ tự frame, không sửa `data/clips/clip_02/gt/gt.txt`, và không
sửa gold sau khi nhận.

## Công cụ chấm

Tất cả script trong `tools/` **chỉ dùng thư viện chuẩn của Python** — chạy được ngay,
không cần cài gì (trừ `visualize_tracks.py` cần Pillow và `run_tracker.py` cần
ultralytics). Các chỉ số HOTA / DetA / AssA / LocA / MOTA / MOTP / IDF1 đã được
đối chiếu khớp tuyệt đối với [TrackEval](https://github.com/JonathonLuiten/TrackEval),
bộ đánh giá chính thức của MOTChallenge — xem `instructor/tools/validate_metrics.py`.

```bash
# 1. kiểm định dạng, trước khi nộp  (không cần gold)
python3 tools/check_mot_labels.py --clip data/clips/clip_01 --tracks annotations/clip_01/gt.txt

# 2. chấm nhãn với gold  (sau khi giảng viên phát)
python3 tools/evaluate_tracking.py --pred annotations/clip_01/gt.txt \
    --gt gold/clip_01/gt.txt --seqinfo data/clips/clip_01/seqinfo.ini \
    --output outputs/eval_vs_gold.json
```

## Điều kiện đi tiếp

Sau khi chấm với gold, bạn qua cổng annotation khi cả ba điều kiện cùng đúng:

| Chỉ số | Ngưỡng | Bắt lỗi gì |
| --- | ---: | --- |
| `IDF1` | >= 0.80 | ID có bị nhảy, bị tách, bị gán nhầm không |
| `MOTA` | >= 0.75 | có bỏ sót xe hoặc vẽ thừa bbox không |
| `MOTP` | >= 0.70 | bbox có khít không |

Chưa đạt thì **sửa nhãn rồi chạy lại** — `evaluate_tracking.py` in ra sẵn danh sách
frame và ID cần sửa, phân theo đúng năm loại lỗi trong slide. Đây là chỉ số chất
lượng *annotation*, không phải chỉ số của model.

## Ghi chú dữ liệu

Nguồn gốc, giấy phép và định dạng nhãn: xem [data/README.md](data/README.md).
Clip chỉ gán **xe bốn bánh**; người đi bộ, xe đạp và **xe máy** nằm ngoài schema,
gán thêm sẽ bị tính là bbox thừa. Không thêm video/ảnh cá nhân, biển số, hay dữ
liệu nhạy cảm vào repository hoặc Colab công khai.
