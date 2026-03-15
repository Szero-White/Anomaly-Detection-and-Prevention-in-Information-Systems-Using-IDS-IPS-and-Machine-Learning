================================================================================
                           HƯỚNG DẪN CÀI ĐẶT VÀ SỬ DỤNG
================================================================================

## 1. YÊU CẦU HỆ THỐNG

- Python 3.10+
- Ubuntu/Debian Linux
- pfSense (cho chế độ giám sát thời gian thực)
- GPU (khuyến nghị cho VAE)

## 2. CÀI ĐẶT

# Tạo môi trường ảo
python3 -m venv mlenv
source mlenv/bin/activate

# Cài đặt dependencies
pip install -r requirements.txt

# Cài đặt CICFlowMeter (trích xuất flow từ PCAP)
pip install cicflowmeter

## 3. CẤU HÌNH

# Tạo file .env từ template
cp .env.example .env

# Chỉnh sửa file .env với thông tin pfSense:
PFSENSE_HOST=172.16.158.100
PFSENSE_USER=admin
PFSENSE_INTERFACE=em1

## 4. CHẠY DỰ ÁN

### 4.1 Phân tích file PCAP
python scripts/detection_pipeline.py --pcap data/sample.pcap

### 4.2 Giám sát liên tục từ pfSense
python scripts/detection_pipeline.py --continuous --interval 30

### 4.3 Giám sát với chế độ streaming (capture song song với analysis)
python scripts/detection_pipeline.py --continuous --streaming --interval 30

### 4.4 Kiểm tra với dữ liệu test
python tests/test_flows_detection.py --flows data/test/test_flows_20260106_153916.csv

## 5. CÁC TÙY CHỌN ACTION

--action alert      # Hiển thị cảnh báo trên console
--action log        # Ghi log vào file
--action block      # Chặn IP qua pfSense
--action webhook    # Gửi webhook
--action email      # Gửi email cảnh báo
--action wazuh      # Tích hợp Wazuh HIDS

Ví dụ:
python scripts/detection_pipeline.py --pcap sample.pcap --action alert --action log

## 6. CẤU TRÚC THƯ MỤC

source/
├── scripts/
│   ├── detection_pipeline.py   # Script chính
│   └── nids/                   # Package NIDS
│       ├── layers.py           # XGBoost, VAE classifiers
│       ├── pipeline.py         # Detection pipeline
│       ├── actions.py          # Response actions
│       ├── capture.py          # pfSense capture
│       └── wazuh.py            # Wazuh integration
├── notebooks/
│   ├── train_cicids_final.ipynb      # Train XGBoost
│   └── train_anomaly_detector.ipynb  # Train VAE
├── data/
│   ├── models/                 # Trained models
│   ├── cicids/                 # CIC-IDS-2017 dataset
│   └── test/                   # Test data
├── tests/
│   └── test_flows_detection.py # Test script
└── logs/                       # Detection logs

## 7. HUẤN LUYỆN MÔ HÌNH

### 7.1 Train XGBoost (phân loại tấn công)
jupyter notebook notebooks/train_cicids_final.ipynb

### 7.2 Train VAE (phát hiện bất thường)
jupyter notebook notebooks/train_anomaly_detector.ipynb

## 8. THÔNG SỐ QUAN TRỌNG

- WINDOW_SIZE = 5 (sliding window cho temporal features)
- Detection threshold = 0.5 (mặc định)
- Block duration = 300s (thời gian chặn IP)

## 9. LIÊN HỆ

GitHub: https://github.com/minhTheGuy/AnomalyDetection

================================================================================
