# Benchmark Report – Lab 16: Cloud AI Environment Setup

**Họ và tên:** Phạm Trung Hiệu  
**Mã sinh viên:** 2A202600766  
**Ngày thực hiện:** 18/06/2026

---

## 1. Môi trường thực thi

| Thông số | Giá trị |
|---|---|
| Cloud Provider | AWS |
| Instance Type | `t3.small` |
| CPU | 2 vCPU |
| RAM | 2 GB |
| Framework | LightGBM (CPU fallback) |
| Dataset | Credit Card Fraud Detection |

---

## 2. Kết quả Benchmark

### 2.1 Hiệu năng Training

| Chỉ số | Giá trị |
|---|---|
| Tổng số dòng dữ liệu | 284,807 |
| Train set | 199,364 rows (70%) |
| Test set | 85,443 rows (30%) |
| Thời gian load dữ liệu | 2.69 giây |
| Thời gian training | **3.03 giây** |
| Số iterations (boosting rounds) | 100 |

### 2.2 Chất lượng Mô hình

| Chỉ số | Giá trị | Đánh giá |
|---|---|---|
| Accuracy | 0.9974 | Cao – nhưng misleading do imbalanced data |
| Precision | 0.3563 | Thấp – nhiều false positive |
| Recall | 0.5946 | Trung bình – phát hiện được ~60% fraud |
| **F1-score** | **0.4456** | Trung bình |
| **AUC-ROC** | **0.7871** | Trung bình – chấp nhận được |

### 2.3 Hiệu năng Inference

| Chỉ số | Giá trị |
|---|---|
| Inference latency | **1.66 ms** |
| Throughput | **121,195 rows/giây** |

---

## 3. Phân tích kết quả

### Training Time
LightGBM hoàn thành training trên ~200K rows chỉ trong **3.03 giây** trên CPU `t3.small`. Đây là tốc độ rất tốt nhờ thuật toán gradient boosting được tối ưu hóa cho CPU, sử dụng histogram-based learning giúp giảm độ phức tạp tính toán.

### AUC-ROC
AUC = **0.7871** cho thấy mô hình có khả năng phân biệt giao dịch gian lận ở mức trung bình. Nguyên nhân chính là dataset cực kỳ mất cân bằng (fraud chỉ chiếm ~0.17% tổng giao dịch), khiến mô hình thiên về dự đoán class majority. Với GPU và deep learning (AutoEncoder, LSTM), AUC có thể đạt ~0.95+.

### Inference Speed
Latency **1.66ms** và throughput **121,195 rows/s** cho thấy LightGBM hoàn toàn đáp ứng được yêu cầu real-time fraud detection trong production, ngay cả trên instance nhỏ.

---

## 4. Lý do sử dụng CPU thay GPU

Tài khoản AWS Free Tier không được cấp quota mặc định cho các GPU instance (`g4dn.xlarge`, `p3.2xlarge`). Việc sử dụng GPU instance yêu cầu:

1. Gửi yêu cầu tăng Service Quota lên AWS Support (thường mất 1–3 ngày duyệt).
2. Chi phí ~$0.526–$3.06/giờ tùy instance.
3. Tài khoản hiện tại đang ở Free Tier với `$0.00` chi phí phát sinh.

Do đó, lab được thực hiện với **CPU fallback plan** sử dụng LightGBM trên `t3.small` để:
- Hoàn thành đúng deadline mà không cần chờ quota approval.
- Không phát sinh chi phí ngoài mức Free Tier.
- Vẫn minh họa được quy trình cloud ML deployment đầy đủ.

---

## 5. So sánh CPU vs GPU (ước tính)

| Chỉ số | CPU (t3.small) – Thực tế | GPU (g4dn.xlarge) – Ước tính |
|---|---|---|
| Instance cost | ~$0.023/giờ | ~$0.526/giờ |
| Train time | 3.03s | ~0.5–1s |
| AUC-ROC | 0.7871 | ~0.95+ (deep learning) |
| Framework | LightGBM | XGBoost GPU / PyTorch |
| Phù hợp | Tabular data nhỏ–vừa | Large-scale, unstructured data |

---

## 6. Kết luận

Lab 16 đã triển khai thành công môi trường Cloud AI trên AWS với CPU fallback. Mặc dù không có GPU, LightGBM vẫn cho kết quả training nhanh và inference latency thấp, phù hợp cho bài toán fraud detection trên tabular data. Để cải thiện AUC, hướng tiếp theo là xin GPU quota và thử nghiệm với deep learning models hoặc ensemble methods.

---

## 7. Artifacts

| File | Mô tả |
|---|---|
| `benchmark_result.json` | Kết quả benchmark chi tiết dạng JSON |
| `benchmark_result.png` | Biểu đồ trực quan kết quả |
| `bills.png` | Screenshot AWS Billing ($0.00) |
| `terraform/` | IaC code cho AWS infrastructure |
| `terraform-gcp/` | IaC code cho GCP infrastructure |
