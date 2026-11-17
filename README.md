# Phát hiện ô tô với YOLOv8

Dự án thực nghiệm phát hiện và khoanh vùng ô tô trong ảnh giao thông.
Mô hình được huấn luyện bằng Ultralytics YOLOv8, với một lớp `car`.
Notebook chính: `archive/data/train_model.ipynb`.

## Mục lục
- [Giới thiệu](#giới-thiệu)
- [Cấu trúc dự án](#cấu-trúc-dự-án)
- [Dữ liệu](#dữ-liệu)
- [Cài đặt](#cài-đặt)
- [Huấn luyện](#huấn-luyện)
- [Dự đoán](#dự-đoán)
- [Kết quả](#kết-quả)
- [Khắc phục sự cố](#khắc-phục-sự-cố)
- [Giới hạn](#giới-hạn)

## Giới thiệu
Đây là bài toán phát hiện đối tượng, không phải phân loại toàn ảnh.
Mỗi dự đoán gồm hộp giới hạn, nhãn lớp và độ tin cậy.
Notebook đọc nhãn CSV, chuẩn bị dữ liệu định dạng YOLO và huấn luyện mô hình.
Trọng số YOLOv8n được dùng làm điểm khởi đầu cho quá trình fine-tune.

## Cấu trúc dự án
```text
.
├── README.md
├── archive/
│   └── data/
│       ├── data.yaml
│       ├── train_model.ipynb
│       ├── train_solution_bounding_boxes (1).csv
│       ├── sample_submission.csv
│       ├── yolov8n.pt
│       ├── training_images/
│       ├── testing_images/
│       ├── yolo_dataset/
│       │   ├── images/{train,val}/
│       │   └── labels/{train,val}/
│       └── runs/detect/car_detector_run-2/
│           ├── args.yaml
│           ├── results.csv
│           └── weights/{best,last}.pt
└── archive (2)/
    └── train_model.ipynb
```
`archive/data` chứa dữ liệu và kết quả của quy trình YOLO được mô tả ở đây.
`archive (2)` có notebook riêng; không cần dùng để chạy hướng dẫn này.
Ultralytics lưu log, biểu đồ và trọng số trong thư mục `runs`.

## Dữ liệu
### Nhãn huấn luyện
`train_solution_bounding_boxes (1).csv` chứa nhãn gốc, với các cột:

| Cột | Ý nghĩa |
| --- | --- |
| `image` | Tên ảnh |
| `xmin`, `ymin` | Góc trên bên trái của hộp theo pixel |
| `xmax`, `ymax` | Góc dưới bên phải của hộp theo pixel |

Nhãn YOLO có định dạng một đối tượng trên mỗi dòng:
```text
class_id x_center y_center width height
```
Tọa độ tâm, chiều rộng và chiều cao được chuẩn hóa về khoảng `[0, 1]`.
Lớp `car` có chỉ số `0`; tệp `.txt` trùng tên gốc với ảnh.
Tập dữ liệu hiện có 284 ảnh train và 71 ảnh validation.

### Ảnh kiểm tra
`archive/data/testing_images` hiện chứa 175 ảnh `vid_5_*.jpg`.
Các ảnh này không thuộc hai tập train/validation đã cấu hình.
Có thể chạy dự đoán trên toàn thư mục bằng mô hình đã huấn luyện.

### Cấu hình dữ liệu
`archive/data/data.yaml` trỏ đến thư mục `yolo_dataset`, khai báo ảnh train,
ảnh validation và một lớp tên `car`. Cấu hình tương ứng:
```yaml
path: C:/path/to/archive/data/yolo_dataset
train: images/train
val: images/val
nc: 1
names:
  - car
```
Đường dẫn `path` trong tệp hiện tại có thể phụ thuộc máy đã tạo nó.
Nếu chuyển dự án, hãy sửa `path` cho đúng vị trí dữ liệu trên máy mới.

## Yêu cầu
- Python 3.10 trở lên (notebook hiện dùng Python 3.12).
- pip và Jupyter Notebook hoặc Jupyter trong VS Code.
- PyTorch và torchvision tương thích với hệ điều hành.
- Ultralytics, OpenCV, pandas, matplotlib và scikit-learn.
- RAM đủ để tải dữ liệu; GPU NVIDIA được khuyến nghị khi huấn luyện.

Notebook có thể chạy trên CPU, nhưng quá trình huấn luyện sẽ chậm hơn.
Lần chạy đã lưu sử dụng CPU, ảnh kích thước 640 và batch size 8.

## Cài đặt
Mở terminal tại thư mục gốc của dự án, sau đó tạo môi trường ảo:
```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
```
Nếu dùng Command Prompt thay vì PowerShell:
```bat
.venv\Scripts\activate.bat
```
Cài các thư viện dùng trong notebook:
```powershell
python -m pip install ultralytics opencv-python pandas matplotlib scikit-learn
python -m pip install jupyter
```
PyTorch có thể cần lệnh cài khác tùy phiên bản CUDA và GPU.
Tham khảo hướng dẫn cài PyTorch chính thức để chọn đúng gói.

## Huấn luyện
1. Mở `archive/data/train_model.ipynb` trong VS Code hoặc Jupyter.
2. Chọn kernel Python thuộc môi trường ảo vừa tạo.
3. Kiểm tra CSV, ảnh và `data.yaml` có ở đúng vị trí.
4. Cập nhật các đường dẫn tuyệt đối nếu chúng không khớp máy hiện tại.
5. Chạy lần lượt các ô từ trên xuống và theo dõi log.
6. Sau khi hoàn tất, kiểm tra thư mục `runs/detect`.

Các tham số của lần chạy hiện có:
| Tham số | Giá trị |
| --- | --- |
| Trọng số khởi tạo | `yolov8n.pt` |
| Epoch | 25 |
| Kích thước ảnh | 640 |
| Batch size | 8 |
| Thiết bị | CPU |
| Tên lần chạy | `car_detector_run-2` |

Trọng số tốt nhất: `archive/data/runs/detect/car_detector_run-2/weights/best.pt`.
Trọng số epoch cuối: `archive/data/runs/detect/car_detector_run-2/weights/last.pt`.
Tùy chọn huấn luyện có thể thay đổi trong notebook hoặc tham số Ultralytics.
Không ghi đè lần chạy cần giữ để đối chiếu; Ultralytics có thể tạo tên mới.

## Dự đoán
Có thể dự đoán toàn bộ ảnh kiểm tra bằng Python từ thư mục gốc:
```python
from ultralytics import YOLO

model = YOLO(
    "archive/data/runs/detect/car_detector_run-2/weights/best.pt"
)
results = model.predict(
    source="archive/data/testing_images",
    imgsz=640,
    conf=0.25,
    save=True,
)
```
Ảnh có hộp dự đoán thường được lưu trong `runs/detect/predict`.
Nếu chạy nhiều lần, thư mục có thể mang tên `predict2`, `predict3`, v.v.
`conf` là ngưỡng độ tin cậy; tăng ngưỡng sẽ lọc bớt dự đoán yếu.

Dự đoán một ảnh riêng bằng cách đổi `source`:
```python
results = model.predict(
    source="archive/data/testing_images/vid_5_25100.jpg",
    imgsz=640,
    conf=0.25,
    save=True,
)
```

## Kết quả
Thư mục `car_detector_run-2` lưu cấu hình `args.yaml`, log `results.csv`,
trọng số và các biểu đồ đánh giá do Ultralytics tạo ra.
Metric ở epoch 25 trong log hiện có:

| Chỉ số | Giá trị |
| --- | ---: |
| Precision | 0.99542 |
| Recall | 0.96581 |
| mAP@0.50 | 0.99409 |
| mAP@0.50:0.95 | 0.66571 |

Đây là kết quả trên tập validation của lần chạy đã lưu, không phải cam kết
hiệu năng trên dữ liệu mới. mAP@0.50:0.95 dùng nhiều ngưỡng IoU nghiêm ngặt hơn.

## Khắc phục sự cố
### Không tìm thấy CSV hoặc ảnh
- Kiểm tra thư mục làm việc và đường dẫn tương đối trong notebook.
- Đảm bảo tên CSV có hậu tố `(1)` như tên tệp trong dự án.
- Kiểm tra ảnh gốc và các thư mục train/validation đã được giải nén.

### Không tìm thấy dữ liệu YOLO
- Kiểm tra `path` trong `data.yaml` trỏ đúng đến `yolo_dataset`.
- Không di chuyển riêng thư mục `images` hoặc `labels`.

### CUDA không khả dụng hoặc bộ nhớ thiếu
- CPU vẫn có thể dùng; cấu hình lần chạy được lưu sử dụng CPU.
- Với GPU, kiểm tra driver NVIDIA và phiên bản PyTorch.
- Giảm batch size hoặc `imgsz` nếu thiếu RAM/VRAM.

### Không thấy ảnh kết quả
- Đọc `save_dir` trong log; thư mục đầu ra có thể tăng hậu tố.
- Kiểm tra quyền ghi của tài khoản tại thư mục dự án.

## Giới hạn
- Dữ liệu hiện tại chỉ có một lớp ô tô và số ảnh còn hạn chế.
- Validation có thể không đại diện cho mọi điều kiện giao thông.
- Nên kiểm tra thêm ảnh ban đêm, thời tiết xấu và xe bị che khuất.
- Đánh giá trên tập test có nhãn riêng trước khi triển khai thực tế.
- Ghi lại phiên bản thư viện, seed, cấu hình và cách chia dữ liệu
  để so sánh các lần huấn luyện một cách công bằng.
- Kiểm tra giấy phép của dữ liệu và trọng số trước khi phân phối mô hình.

## Đóng góp
Dự án phục vụ mục đích học tập và thực nghiệm thị giác máy tính.
Khi sửa notebook, hãy ghi rõ thay đổi dữ liệu và siêu tham số.
