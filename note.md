MOSSE (Minimum Output Sum of Squared Error) filter-based tracker là một thuật toán theo dõi đối tượng hiệu quả, có tốc độ cao và hoạt động tốt ngay cả trong điều kiện thay đổi ánh sáng, xoay và biến đổi hình dạng. Dưới đây là từng bước thực hiện thuật toán MOSSE tracker:

### **1. Khởi tạo bộ lọc MOSSE**
- **Chọn đối tượng**: Người dùng chọn một vật thể trong khung hình đầu tiên bằng cách vẽ một hộp giới hạn (bounding box).
- **Cắt và xử lý trước ảnh**:
  - Chuyển đổi sang ảnh grayscale để giảm nhiễu màu.
  - Áp dụng log transformation để giảm tác động của thay đổi ánh sáng.
  - Chuẩn hóa ảnh sao cho có trung bình bằng 0 và phương sai bằng 1.
  - Chuyển ảnh sang miền Fourier bằng **Fast Fourier Transform (FFT)**.
- **Tạo đầu ra tổng hợp (Synthetic Target)**:
  - Một mặt nạ Gaussian được tạo ra với đỉnh tập trung vào vị trí đối tượng cần theo dõi.
  - Mặt nạ này cũng được chuyển sang miền Fourier.
- **Tính toán bộ lọc MOSSE**:
  - Bộ lọc được tính theo công thức:
    \[
    H^* = \frac{\sum G_i \cdot F_i^*}{\sum F_i \cdot F_i^* + \epsilon}
    \]
    Trong đó:
    - \( H^* \) là bộ lọc cần tìm.
    - \( G_i \) là đầu ra mong muốn (Gaussian mask).
    - \( F_i \) là ảnh đầu vào đã được xử lý.
    - \( * \) là liên hợp phức.
    - \( \epsilon \) là một số nhỏ để tránh chia cho 0.

### **2. Theo dõi đối tượng**
- **Cập nhật khung theo dõi**:
  - Lấy khung hình mới từ video.
  - Cắt ảnh theo vị trí của đối tượng trong khung trước đó.
  - Chuyển đổi ảnh sang miền Fourier và thực hiện phép tương quan với bộ lọc \( H^* \):
    \[
    R = H^* \cdot F
    \]
  - Chuyển kết quả ngược về miền không gian bằng **IFFT** và xác định vị trí có giá trị cao nhất để xác định vị trí mới của đối tượng.
- **Cập nhật bộ lọc theo thời gian**:
  - Sau mỗi lần theo dõi, bộ lọc được cập nhật để thích ứng với thay đổi của đối tượng theo công thức:
    \[
    N_i = \eta (G_i \cdot F_i^*) + (1 - \eta) N_{i-1}
    \]
    \[
    D_i = \eta (F_i \cdot F_i^* + \epsilon) + (1 - \eta) D_{i-1}
    \]
    \[
    H^*_i = \frac{N_i}{D_i}
    \]
  - Trong đó, \( \eta \) là hệ số học (learning rate), giúp bộ lọc thích nghi theo thời gian.

### **3. Các cải tiến của MOSSE Tracker**
- **Cập nhật cửa sổ theo dõi (Tracking Window Update)**:
  - Nếu đối tượng thay đổi vị trí quá nhanh hoặc di chuyển ngoài khung, thuật toán có thể cập nhật lại vị trí cửa sổ theo dõi để duy trì theo dõi.
- **Affine Transformations**:
  - Dùng biến đổi affine như co giãn, quay để tăng dữ liệu huấn luyện cho bộ lọc, giúp theo dõi tốt hơn trong các điều kiện thay đổi.

MOSSE filter-based tracker có ưu điểm về tốc độ, độ chính xác cao, phù hợp với các hệ thống theo dõi thời gian thực, nhưng vẫn có hạn chế khi đối tượng bị che khuất hoàn toàn hoặc thay đổi hình dạng quá mạnh.