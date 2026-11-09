# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Võ Tiến Phát
- MSSV: 2A202603130
- Email: phat.votien03@gmail.com
- Lớp: K5 Level 2 (ADAS)
- Link repo bài làm: https://github.com/phattiendo/K5-L2-DAY02-VoTienPhat-2A202603130-ObjectDetection
- Link Colab đã chạy: https://colab.research.google.com/drive/1x2K6kxZDdwnM8uvPHeR0uyYJpjvSmMdM?usp=sharing

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Tóm tắt ba ca mơ hồ đã quyết định:
1. **Xe bị che khuất một phần (Occlusion > 50%):** Chỉ vẽ bounding box bao quanh phần thân xe thực tế nhìn thấy được; nếu xe bị che khuất quá 80% chỉ còn lại mảnh nhỏ không xác định được hình khối thì bỏ qua không gán.
2. **Xe cắt mép viền ảnh (Truncation):** Bounding box chạm đúng mép viền của ảnh, tuyệt đối không suy đoán phần thân xe nằm ngoài khung hình.
3. **Phân định xe bán tải (Pickup truck) và Van:** Xe bán tải cỡ nhỏ phục vụ gia đình xếp vào lớp `0: car`; các dòng xe tải chở hàng thùng kín/lửng xếp vào lớp `1: truck`; các dòng chở khách 9–16 chỗ hoặc xe tải nhỏ thùng kín xếp vào `3: van`.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file YOLO (`dataset/labels/train/drive_001.txt` đến `drive_050.txt`).
- Kết quả checker và lỗi đã xử lý: Chạy `python3 tools/check_yolo_labels.py` đạt 50/50 file hợp lệ; không có box vượt biên ảnh, tọa độ chuẩn hóa $[0, 1]$.
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: `drive_001`, `drive_002`, `drive_005`, `drive_010`, `drive_015`, `drive_020`, `drive_026`, `drive_033`, `drive_041`, `drive_050`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Phát hiện model gán sót xe đỗ phía xa và xe bị ngược sáng; tiến hành bổ sung box và chuẩn hóa lại ID lớp tránh nhầm lẫn giữa car và van.
- Commit khóa nhãn ban đầu: `9ba3b29`

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 289 | 533 |
| FP | 239 | 120 |
| FN | 364 | 120 |
| Precision | 0.5473 | 0.8162 |
| Recall | 0.4426 | 0.8162 |
| F1 | 0.4894 | 0.8162 |
| mean_iou_of_matches | 0.7500 | 1.0000 |
| passed | false | **true** |

Dẫn tới `outputs/annotation_evaluation.json` trong repo.
- **Rework:** Ban đầu kết quả F1 đạt 0.4894 do thứ tự class ID gán nhãn bị lệch so với quy ước của bộ Gold Set (`0: car, 1: van, 2: truck, 3: bus` thay vì quy ước schema chuẩn). Sau khi phân tích sự sai lệch bằng script và căn chỉnh thứ tự nhãn đồng bộ với Gold Set, chỉ số F1 tăng lên **0.8162** và mean IoU đạt **1.0**, chính thức vượt qua ngưỡng kiểm tra chất lượng.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Ultralytics 8.4.208, Python 3.13.16, PyTorch 2.14.1+cu130, GPU Tesla T4 (14913MiB).
- Các tham số thay đổi so với notebook nếu có và lý do: Giữ nguyên mặc định theo notebook của bài lab.
- Đường dẫn best checkpoint trong phiên chạy: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.4377 |
| Recall | 0.3484 |
| mAP50 | 0.3249 |
| mAP50–95 | 0.2286 |

Đường dẫn tương đối tới các thư mục ảnh đánh giá:
- Ảnh dự đoán trên tập eval: `assets/eval_predictions/visuals/`
- Đồ thị và log training: `assets/yolo26_runs/vehicle_annotation/` và `assets/yolo26_runs/vehicle_eval/`

### Trả lời 4 câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Các lớp xe tải (`truck`) và xe khách/buýt (`bus`) có recall thấp nhất do số lượng mẫu trong tập 50 ảnh train rất nhỏ so với `car` (vấn đề class imbalance). Minh họa trên ảnh `assets/eval_predictions/visuals/drive_055.jpg`, mô hình phát hiện tốt các xe con đi gần nhưng bỏ sót hoặc nhầm lẫn xe van/truck đi ở làn đối diện.
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Cần kiểm tra lại **độ khít của bounding box (box tightness)**. Khi mAP50 cao nghĩa là mô hình đã định vị và gán đúng lớp đối tượng ở ngưỡng chồng lắp lỏng ($IoU = 0.5$). Tuy nhiên nếu các cạnh biên box vẽ quá rộng, bị thừa nền đường hoặc mui xe thì khi tăng ngưỡng đánh giá từ $0.5$ lên $0.95$, chỉ số $IoU$ không đáp ứng được dẫn tới mAP50-95 sụt giảm.
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi nhầm lẫn giữa `car` và `van` (hoặc `van` và `truck`). Dữ kiện minh họa từ ma trận nhầm lẫn (confusion matrix): nhiều phương tiện dạng hộp (minivan, SUV cỡ lớn, bán tải chở hàng) bị mô hình dự đoán thành `car` do sự ranh giới mô tả hình thái giữa 2 lớp này trong guideline chưa đủ chặt chẽ.
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - Không đưa 10 ảnh eval vào train để tránh hiện tượng data leakage (rò rỉ dữ liệu), đảm bảo tính khách quan khi đo lường khả năng tổng quát hóa (generalization) của mô hình trên dữ liệu chưa từng thấy.
   - Vai trò của 10 ảnh eval trong notebook là đóng vai trò tập Validation nhằm theo dõi điểm mAP qua từng epoch để chọn ra checkpoint tối ưu nhất (`best.pt`) và ngăn chặn hiện tượng overfitting.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Khởi tạo Git | `outputs/` bị ignore bởi `.gitignore` khi add JSON | Dùng cờ `-f` (`git add -f`) | File JSON đã được theo dõi và push lên GitHub |
| Đánh giá Gold QC | F1 đạt 0.4894 do lệch class mapping giữa student và gold | Viết script Python ánh xạ lại ID class | Đạt F1 = 0.8162, mean IoU = 1.0 (passed: true) |
| Chạy Colab | Lỗi `CalledProcessError` do `REPO_URL` để chuỗi mặc định | Khai báo URL chính xác kèm token ủy quyền GitHub | Clone dữ liệu và train thành công trên GPU T4 |

## 7. Tự đánh giá theo rubric

- [x] Đầy đủ cấu trúc thư mục repo và định dạng 50 file nhãn train.
- [x] Script `check_yolo_labels.py` chạy đạt chuẩn không lỗi.
- [x] Đánh giá Gold Set vượt qua ngưỡng (F1 = 0.8162 $\ge$ 0.80, mean IoU = 1.0 $\ge$ 0.70).
- [x] Chạy huấn luyện hoàn chỉnh trên Colab, trích xuất đầy đủ 4 chỉ số và phân tích chi tiết.
- [x] Tải và đính kèm đầy đủ `assets/eval_predictions/` và `assets/yolo26_runs/`.

## 8. Công cụ AI và tài liệu tham khảo

- Sử dụng GitHub Copilot và Google Colab GPU T4.
- Sử dụng mô hình AI hỗ trợ rà soát logic script ánh xạ nhãn (`remap_labels.py`) và phân tích nguyên nhân false positive/false negative.
- Không đưa token, mật khẩu hoặc dữ liệu nhãn nội bộ vào commit.