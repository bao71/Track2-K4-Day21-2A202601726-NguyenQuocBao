# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Nguyễn Quốc Bảo |
| MSSV | 2A202601726 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/bao71/Track2-K4-Day21-2A202601726-NguyenQuocBao |
| Ngày nộp | 21/8/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 200 | 0.1 | 5 | 0.7149 | 0.874 |
| 2 | 100 | 0.1 | 3 | 0.7109 | 0.878 |
| 3 | 50 | 0.05 | 2 | 0.6051 | 0.846 |

**Bộ siêu tham số đã chọn:** `n_estimators=200`, `learning_rate=0.1`, `max_depth=5`.

**Lý do:** Tôi chọn bộ siêu tham số này vì nó đạt `f1_score` cao nhất trong các lần thử nghiệm trên MLflow. Bộ `n_estimators=100`, `learning_rate=0.1`, `max_depth=3` có accuracy cao hơn một chút, nhưng F1 thấp hơn, nên không phù hợp bằng với tiêu chí chính của bài. Điều này cho thấy accuracy không phải lúc nào cũng phản ánh tốt khả năng nhận diện lớp thu nhập cao trong dữ liệu mất cân bằng. Cấu hình `n_estimators=50`, `learning_rate=0.05`, `max_depth=2` cho F1 thấp hơn rõ rệt, có thể do mô hình còn yếu và chưa học đủ quan hệ trong dữ liệu. Việc tăng số cây và độ sâu giúp mô hình cải thiện F1, trong khi learning rate vẫn giữ ở mức ổn định.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Tập Adult/Census Income là bài toán phân loại mất cân bằng, trong đó chỉ khoảng 24,8% mẫu thuộc lớp thu nhập trên 50K. Nếu một mô hình luôn dự đoán tất cả là thu nhập thấp, accuracy vẫn có thể đạt khoảng 0,752 nhưng hoàn toàn không phát hiện được lớp thu nhập cao. Vì vậy accuracy dễ gây hiểu nhầm trong bài này. F1-score của lớp dương đo cân bằng giữa precision và recall cho nhóm thu nhập cao, nên phản ánh tốt hơn khả năng nhận diện đúng nhóm quan trọng. Tôi không dùng `average="weighted"` hoặc `average="macro"` vì các cách tính đó có thể làm chỉ số bị ảnh hưởng bởi lớp đa số hoặc không đúng với quality gate của lab, vốn yêu cầu F1 của lớp dương.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Cài thư viện Python ban đầu bị lỗi encoding và thiếu `pkg_resources`. | Project đặt trong đường dẫn có dấu và phiên bản `setuptools` quá mới so với MLflow. | Chuyển project sang đường dẫn không dấu và pin `setuptools<81`. |
| GitHub Actions không pull được dữ liệu từ S3. | DVC remote thiếu region và GitHub Secrets ban đầu chưa đặt đúng tên. | Thêm `region = ap-southeast-2` vào `.dvc/config` và tạo đúng các secrets AWS. |
| API trên EC2 restart nhưng chưa phục vụ được model. | Phiên bản scikit-learn trên EC2 khác với môi trường train. | Cài lại `scikit-learn==1.4.2` và `joblib==1.4.2` trong virtual environment của EC2. |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7149 | 0.874 |
| Bước 3 (thêm `train_batch2`) | 0.7354 | 0.882 |

**Nhận xét:** Sau khi thêm `train_batch2`, F1 tăng từ 0.7149 lên 0.7354 và accuracy tăng từ 0.874 lên 0.882. Kết quả này cho thấy dữ liệu bổ sung giúp mô hình nhận diện lớp thu nhập cao tốt hơn một chút, nhưng phần quan trọng nhất của Bước 3 là pipeline đã tự động chạy lại từ commit dữ liệu, kiểm tra quality gate và triển khai lại model mà không cần thao tác thủ công.
