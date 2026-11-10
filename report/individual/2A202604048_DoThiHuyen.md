# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Đỗ Thị Huyền
- MSSV: 2A202604048
- Email: hdo110194@gmail.com
- Lớp: L2B
- Link repo bài làm: https://github.com/NhatHaDo/K5-L2-DAY02--Th-Huy-n-2A202604048-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1OTe2ahqPQU4GkcEnehpmPqYHKhE_7AXZ?usp=sharing

## 2. Schema và mini guideline

Xem [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Ba ca mơ hồ chính đã quyết định:

- `drive_014`: xe cứu hộ có cần cẩu → `truck` (cabin tách rời thiết bị phía sau). Xe đỏ thân hộp cỡ lớn ở góc phải → `van` (thân ngắn, ca-pô ngắn, nhỏ hơn hẳn bus).
- `drive_024`: xe ba bánh chở khách → không gán (ngoài schema); minivan trắng/bạc cỡ nhỏ → `van`.
- `drive_042`: xe chở hàng chất bao tải cao → `truck`; xe buýt bị cắt ở rìa ảnh → vẫn gán `bus`, box chạm rìa.

Hai điểm khác biệt giữa tài liệu đề bài và gold, phát hiện khi chấm:

1. **Thứ tự class ID:** gold train và `dataset/labels/eval` dùng thứ tự car=0, van=1, truck=2, bus=3. `data.yaml`/README ghi car=0, truck=1, bus=2, van=3. Bằng chứng: trong 10 ảnh eval, mọi xe buýt lớn đều mang ID 3.
2. **Xe đỗ và xe ở xa:** README ghi "xe đang đỗ vẫn là car", nhưng gold không gán dãy xe đỗ ven đường và dòng xe nhỏ ở xa (ví dụ `drive_052`, `drive_056`, `drive_060` trong eval).

## 3. Annotation và self-QC

- Số file nhãn train: 50/50 (gán trên CVAT, export Ultralytics YOLO Detection 1.0).
- Kết quả checker và lỗi đã xử lý: `check_yolo_labels.py` đạt 50/50. Các lỗi đã xử lý:
  - Lần copy đầu bị sai đường dẫn nên vòng lặp tạo file rỗng sinh ra 50 file trống, nhưng checker vẫn báo đạt. Phát hiện bằng cách đếm tổng số dòng nhãn, sau đó copy lại.
  - CVAT export theo thứ tự label lúc tạo project (car, truck, van, bus), lệch với schema. Đã map ID theo tên class trong `data.yaml` của file export.
- Ảnh đã rà bằng mắt (trên CVAT, trong lúc self-QC và rework): `drive_001`, `002`, `003`, `011`, `012`, `014`, `024`, `030`, `031`, `038`, `042`, `043`, `044`, `045`, `049`, `050`.
- Lỗi phát hiện khi rà và cách sửa:
  - Box lỏng ở cạnh trên (`drive_001`): kéo lại cạnh.
  - Sót xe bị che một phần (`drive_001`): vẽ thêm box.
  - Minivan trắng gán nhầm `car`: đổi sang `van`.
- Commit khóa nhãn ban đầu: `adeb8e8`.

## 4. Gold QC và rework

| Chỉ số | Trước sửa (`adeb8e8`) | Sau sửa (`21008c4`) |
|---|---:|---:|
| TP | 470 | 503 |
| FP | 718 | 172 |
| FN | 183 | 150 |
| Precision | 0.3956 | 0.7452 |
| Recall | 0.7198 | 0.7703 |
| F1 | 0.5106 | 0.7575 |
| mean_iou_of_matches | 0.8323 | 0.8501 |
| passed | false | false |

Kết quả chấm cuối: [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json).

Các vòng rework:

| Vòng | Thay đổi | Tổng box | F1 |
|---|---|---:|---:|
| 0 | Bản khóa ban đầu | 1188 | 0.5106 |
| Kiểm tra | Chỉ đổi thứ tự ID sang thứ tự của gold, không sửa box | 1188 | 0.6095 |
| 1 | Đổi ID + xóa xe đỗ ven đường/xe nhỏ ở xa ở khoảng 25 ảnh thừa nhiều nhất (`030`, `002`, `003`, `044`, `049`, `045`, `031`, `043`, `050`, `011`, `012`, `038`, `028`…) | 735 | 0.7565 |
| 2 | Đổi thêm minivan `car` → `van`, xóa thêm xe đỗ ở `041`, `009`, `040`, `027`, `024`, `016`, `019`, `034` | 675 | 0.7575 |

Phân tích:

- **Lệch thứ tự class.** Bảng sai lớp ban đầu xoay vòng đều (bus→van 54, van→truck 21, truck→bus 17), không lẻ tẻ như lỗi nhìn nhầm. Chỉ đổi ID, F1 đã tăng từ 0.51 lên 0.61 và FN giảm từ 183 xuống 92.
- **Box thừa.** Nguyên nhân lớn nhất là gán xe đỗ và xe ở xa theo README, trong khi gold không gán. Mean IoU luôn ≥ 0.83, nên độ bám sát của box không phải vấn đề.
- **Vòng 2 không cải thiện.** FP giảm nhưng FN tăng từ 128 lên 150, tức là đã xóa nhầm một số xe đang lưu thông ở cự ly trung bình. Ranh giới "xa/không gán" chưa nhất quán giữa các ảnh.
- **Chưa đạt ngưỡng F1 0.80.** Đã dừng rework để kịp chạy pipeline. Còn khoảng 17 minivan bị gán `car` thay vì `van`, và FP/FN rải ở cỡ box 20–70px.
- **Nguyên tắc rework:** chỉ sửa nhãn của mình, không sửa gold, không copy box từ gold.

## 5. Kết quả YOLO

- Runtime: Google Colab, GPU T4. Ultralytics bản mới nhất cài bằng `pip install -U ultralytics` ở cell đầu notebook, Torch có sẵn của Colab.
- Tham số: giữ nguyên notebook (`yolo26n.pt`, 50 epoch, imgsz 640, batch -1, seed 20260909). Không sửa `data.yaml`.
- Best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (không commit file `.pt`).

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.5894 |
| Recall | 0.5160 |
| mAP50 | 0.5610 |
| mAP50–95 | 0.3116 |

Kết quả:

- Ảnh dự đoán: [`assets/eval_predictions/visuals/`](assets/eval_predictions/visuals/)
- Confusion matrix: [`assets/yolo26_runs/vehicle_eval/confusion_matrix.png`](assets/yolo26_runs/vehicle_eval/confusion_matrix.png)
- Log train: [`assets/yolo26_runs/vehicle_annotation/results.csv`](assets/yolo26_runs/vehicle_annotation/results.csv)

> Lưu ý đọc kết quả: nhãn train và eval cùng dùng thứ tự của gold (0 car, 1 van, 2 truck, 3 bus), còn `data.yaml` vẫn đặt tên theo README. Vì vậy trên ảnh và biểu đồ, tên **"truck" = van**, **"bus" = truck**, **"van" = bus**. Các chỉ số tổng không bị ảnh hưởng.

Số đối tượng ground truth trong eval: car 94, van 5, truck 2, bus 12. Trong train: car 547, van 38, truck 18, bus 72.

1. **Lớp nào recall thấp nhất?** `van` (ID 1, hiện tên "truck" trên biểu đồ). Theo confusion matrix, chỉ bắt đúng 1/5 xe van; 2 xe bị đoán là `car`, 2 xe bị đoán thành ID 3. Ví dụ ở [`drive_056`](assets/eval_predictions/visuals/drive_056.jpg), chiếc van trắng nhỏ ở giữa phía trên ảnh (gold ID 1) bị dự đoán thành ID 3, hiện nhãn "van 0.47" nhưng nghĩa là bus. Ngược lại, van ở [`drive_059`](assets/eval_predictions/visuals/drive_059.jpg) được đoán đúng ("truck 0.45" = van). Các lớp khác: car khoảng 86/94, bus khoảng 10/12. Truck chỉ có 2 mẫu nên recall 2/2 không đáng tin. Một lỗi đáng chú ý nữa ở [`drive_052`](assets/eval_predictions/visuals/drive_052.jpg): xe buýt hai tầng cỡ lớn ở giữa ảnh không được phát hiện.
2. **mAP50 0.56 nhưng mAP50–95 chỉ 0.31.** Model tìm đúng vị trí xe ở ngưỡng IoU 0.5, nhưng box chưa khớp chặt ở ngưỡng IoU cao. Cần kiểm tra:
   - cạnh box của xe bị nhòe chuyển động (ví dụ [`drive_059`](assets/eval_predictions/visuals/drive_059.jpg), xe đen đang chạy),
   - xe bị che hoặc bị cắt khung,
   - xe nhỏ ở xa, nơi lệch vài pixel đã làm IoU giảm mạnh.

   Nhãn train có mean IoU 0.85 so với gold, nên lệch box chủ yếu do model nano chỉ học trên 50 ảnh, không hẳn do nhãn lỏng.
3. **Lỗi class liên quan guideline.**
   - (a) **van ↔ car:** khi chấm gold có 17 box `car` mà gold là `van`. Trên eval, model cũng nhầm van: 2/5 xe van bị đoán là `car`, 2/5 bị đoán thành bus (`drive_056`). Van ít mẫu (38 box train), nên lớp này dễ bị nuốt vào car hoặc bus. Định nghĩa `van` trong guideline chưa phân biệt rõ minivan/MPV với SUV/hatchback cao.
   - (b) **Xe đỗ ven đường:** README bảo gán, gold không gán. Model dự đoán cả dãy xe đỗ bên trái [`drive_052`](assets/eval_predictions/visuals/drive_052.jpg) (car 0.27–0.85), làm FP tăng và precision chỉ còn 0.59.
   - (c) **Lệch thứ tự class giữa `data.yaml` và gold:** khiến tên lớp trên ảnh bị đảo. Ví dụ xe buýt ở [`drive_053`](assets/eval_predictions/visuals/drive_053.jpg) hiện nhãn "van 0.54" nhưng thực ra là ID 3 = bus, tức là đoán đúng.
   - Dữ kiện hỗ trợ: bảng sai lớp xoay vòng ở mục 4 và confusion matrix.
4. **Vì sao không dùng 10 ảnh eval để train?** Notebook dùng chính 10 ảnh này làm validation, để chọn `best.pt` theo epoch và để tính chỉ số cuối. Nếu đưa vào train thì model đã "thấy đáp án", chỉ số sẽ bị thổi phồng và không còn đo được khả năng tổng quát. Ngay cả hiện tại, eval vẫn đã tham gia vào việc chọn checkpoint, nên đây không phải test set độc lập. Kết quả trên 10 ảnh cũng chỉ có tính tham khảo vì số mẫu rất ít, nhất là truck (2) và van (5).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Export CVAT | Thứ tự label trong export (car, truck, van, bus) khác schema | Map ID theo tên class trong `data.yaml` của export | Đã xử lý |
| Copy nhãn vào repo | Sai đường dẫn, vòng lặp tạo 50 file rỗng, checker vẫn đạt | Đếm tổng số dòng nhãn, copy lại | Đã xử lý; checker không phát hiện được lỗi này |
| Gold QC lần 1 | F1 0.51, bảng sai lớp xoay vòng đều | So với gold eval có sẵn trong repo → phát hiện lệch thứ tự ID | Remap theo thứ tự của gold |
| Gold QC | Vẽ gần gấp đôi số box của gold (xe đỗ, xe xa) | Xóa xe đỗ/xe xa trên CVAT theo quy ước của gold eval, 2 vòng | F1 0.76, chưa đạt 0.80 |
| Colab | Popup chờ GitHub cấp phép | Bỏ qua (repo public), chạy rồi lưu bản sao vào Drive | Đã chạy xong |

## 7. Tự đánh giá theo rubric

| Tiêu chí | Tự đánh giá |
|---|---|
| Hoàn thành và định dạng YOLO | Đạt: 50 file, checker 50/50. ID theo thứ tự của gold, đã ghi rõ lý do. |
| Đầy đủ vật thể | Recall cuối 0.77. Có sót xe ở cự ly trung bình do xóa nhầm trong vòng 2. |
| Độ bám sát box | Tốt: mean IoU 0.85. |
| Phân lớp nhất quán | Còn lỗi van↔car (khoảng 17 box). |
| Mini guideline | Có định nghĩa lớp, quy tắc box, 3 ca mơ hồ và mục cập nhật sau rework. |
| Gold-set QC | Chưa đạt F1 0.80 (0.7575). Có 2 vòng rework, kèm số liệu và nguyên nhân. |
| Pipeline YOLO | Colab chạy được, không dùng eval để train, có metrics, ảnh eval và phân tích. |

## 8. Công cụ AI và tài liệu tham khảo

- **Claude (Anthropic)** được dùng để:
  - đọc tài liệu đề bài và lên các bước làm,
  - viết script kiểm tra/chuyển ID nhãn và script thống kê lỗi FP/FN theo lớp và kích thước,
- **Phần tự làm và tự kiểm tra:**
  - toàn bộ việc gán và sửa nhãn trên CVAT (vẽ box, chọn lớp, xóa box),
  - quyết định cho từng ảnh khi rework,
  - chạy checker, evaluator và Colab,
  - kiểm tra lại các ca mơ hồ trên ảnh gốc,
  - viết báo cáo
- AI không gán hay xóa box thay cho người làm, và không dùng gold train để chọn box.
- Nguồn ngoài repo: không có.
