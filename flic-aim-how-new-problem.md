# BÁO CÁO PHÂN TÍCH HỆ THỐNG: FLIC (FEDERATED LEARNING ON HETEROGENEOUS FEATURE SPACES)

---

### 1. PROBLEM (BÀI TOÁN & THÁCH THỨC)
*   **Sự bất tương thích về không gian đặc trưng thô:** Hầu hết các phương pháp học liên bang ngang cá nhân hóa (Personalised Horizontal FL) hiện nay đều bắt buộc phải giả định dữ liệu thô của tất cả các thiết bị biên (clients) cùng chia sẻ chung một cấu trúc/không gian đặc trưng giống hệt nhau
*   **Thực tế dị dạng dữ liệu:** Trong thực tế, các client thu thập và lưu trữ dữ liệu trên các hệ thống, phần cứng và cơ sở dữ liệu riêng biệt. Điều này dẫn đến sự không đồng nhất (dị dạng) đặc trưng (heterogeneous feature spaces) về cả số chiều (dimensionality) lẫn ý nghĩa ngữ nghĩa của các tọa độ vector đặc trưng
    *   *Ví dụ:* Client 1 dùng ảnh MNIST $28 \times 28$ (784 chiều), Client 2 dùng ảnh USPS $16 \times 16$ (256 chiều), Client 3 dùng ảnh SVHN $32 \times 32 \times 3$ (3072 chiều); hoặc các client nắm giữ các loại phương thức đặc trưng hoàn toàn khác nhau như văn bản nhúng BERT (768 chiều) và hình ảnh nhúng ResNet18 (512 chiều)
*   **Hệ quả thất bại của FL truyền thống:** Khi không gian đặc trưng đầu vào dị dạng, các tham số mô hình của các client không tương thích và không thể so sánh được . Do đó, các thuật toán tổng hợp toàn cầu tiêu chuẩn (như FedAvg tính trung bình cộng trọng số tham số) hoàn toàn bị tê liệt vì không thể "lấy trung bình" các ma trận trọng số không cùng số chiều hoặc khác biệt hệ tọa độ

---

### 2. AIM (MỤC TIÊU CHI TIẾT)
*   **Thiết lập Khung học liên bang cá nhân hóa đầu tiên cho không gian đặc trưng dị dạng:** Định nghĩa và giải quyết chính thức về mặt toán học kịch bản học liên bang ngang cá nhân hóa khi không gian dữ liệu của các client không đồng bộ.
*   **Cộng tác học tập bảo mật không chia sẻ dữ liệu thô:** Cho phép mỗi client tự do tận dụng tri thức hữu ích từ dữ liệu của các client khác ngay cả khi họ lưu trữ dưới các cấu trúc và định dạng hoàn toàn khác nhau .
*   **Bảo vệ quyền riêng tư tối đa:** Loại bỏ hoàn toàn sự phụ thuộc vào máy chủ trung tâm trong việc phân phối tập dữ liệu thô tham chiếu dùng chung, ngăn chặn nguy cơ rò rỉ dữ liệu.
*   **Đồng nhất ngữ nghĩa biểu diễn và tối ưu hóa cá nhân hóa:** Giúp dữ liệu sau khi ánh xạ có cấu trúc ngữ nghĩa nhất quán trong một không gian ẩn chung để tiến hành phân loại chính xác, giải quyết trọn vẹn cả sự dị dạng đặc trưng lẫn sự lệch phân phối nhãn thống kê cục bộ (statistical heterogeneity) .

---

### 3. HOW (CƠ CHẾ HOẠT ĐỘNG)
Cơ chế hoạt động của FLIC được vận hành qua pipeline 2 tầng chặt chẽ từ cục bộ đến toàn cục:

#### Bước 1: Ánh xạ không gian con ẩn chung (Local Embedding)
Mỗi client $i$ được trang bị một hàm nhúng cục bộ có thể học được (mạng nơ-ron cục bộ) $\phi_i: \mathcal{X}_i \to \Phi$ để biến đổi dữ liệu thô dị dạng của mình về một **không gian ẩn chung (common latent space) $\Phi$** có số chiều $k$ cố định cho tất cả các client.

#### Bước 2: Hiệu chuẩn ngữ nghĩa qua Phân phối neo ẩn (Latent Anchor Distributions)
Để tránh hiện tượng dữ liệu có cùng ngữ nghĩa (ví dụ: cùng nhãn lớp $c$) từ các client khác nhau bị phân tán hỗn loạn ở các vùng khác nhau trong không gian ẩn $\Phi$, FLIC sử dụng các **phân phối neo ẩn $\{\mu_c\}_{c=1}^C$** (mỗi lớp $c$ có một phân phối neo $\mu_c$) đóng vai trò làm "bộ hiệu chuẩn vạn năng" (universal calibrator) .

#### Bước 3: Căn chỉnh phân phối bằng Khoảng cách Wasserstein ($W_2$)
*   Client thực hiện căn chỉnh phân phối đặc trưng nhúng có điều kiện lớp $\nu_{\phi_i}^{(c)}$ tiến sát tới phân phối neo $\mu_c$ tương ứng bằng cách giảm thiểu **khoảng cách Wasserstein bậc 2 ($W_2^2$)** [2, 9].
*   **## Giả định Gaussian cho công thức đóng siêu tốc

Giả định Gaussian cho công thức đóng siêu tốc: Nhóm tác giả giả định phân phối neo là Gaussian $\mu_c = \mathcal{N}(v_c, \Sigma_c)$ và xấp xỉ phân phối nhúng thực nghiệm của client cũng bằng Gaussian $\hat{\nu}_{\phi_i}^{(c)} = \mathcal{N}(\hat{m}_i^{(c)}, \hat{\Sigma}_i^{(c)})$. Điều này mang lại công thức dạng đóng cho khoảng cách Wasserstein, giúp tránh các phép tính tích phân phức tạp:

<img width="344" height="55" alt="image" src="https://github.com/user-attachments/assets/7d595d50-ac46-46d8-a53f-6c2599b14215" />

Khoảng cách Wasserstein được ưu tiên vì nó vẫn **hữu hạn ngay cả khi hai phân phối hoàn toàn không chồng lấn nhau** (tức là không có phần giao nhau về support).
    Trong đó $\mathfrak{B}$ biểu thị khoảng cách Bures giữa hai ma trận hiệp phương sai. Khoảng cách Wasserstein được ưu tiên vì nó luôn hữu hạn kể cả khi các phân phối ban đầu hoàn toàn lệch nhau (không overlap về mặt support) 

#### Bước 4: Tối ưu hóa hàm mục tiêu cục bộ điều hòa
Mỗi client tối ưu hóa hàm loss cục bộ tích hợp bao gồm loss phân loại chuẩn và loss phạt khoảng cách Wasserstein
$$F_i(\theta_i, \phi_i, \mu_{1:C}) = \omega_i f_i(\theta_i, \phi_i) + \lambda_1 \omega_i \sum_{c\in\mathcal{Y}_i} W_2^2 \left( \mu_c, \nu_{\phi_i}^{(c)} \right) + \lambda_2 \omega_i \sum_{c\in\mathcal{Y}_i} \frac{1}{J} \sum_{j=1}^J \ell \left( c, g_{\theta_i}^{(i)} \left[ Z_c^{(j)} \right] \right)$$
*(Trong đó, thành phần thứ 3 là tùy chọn lấy mẫu trực tiếp từ phân phối neo để hiệu chỉnh classifier, giải quyết hiện tượng trôi hiệp biến covariate shift *

#### Bước 5: Cập nhật song song cục bộ và Tổng hợp toàn cục trên Server
*   **Client update:** Client huấn luyện cục bộ $M$ bước để cập nhật hàm nhúng cục bộ $\phi_i$ và classifier cục bộ $\beta_i$ . Trước khi gửi về server, client thực hiện **chỉ 1 bước** cập nhật cục bộ cho tham số toàn cục (biểu diễn chia sẻ $\alpha_i$ và phân phối neo cục bộ $\mu_{i,c}$) để triệt tiêu hoàn toàn hiện tượng trôi client (client drift) .
*   **Server aggregation:** Server nhận các tham số gửi lên, tiến hành trung bình cộng có trọng số để cập nhật biểu diễn chia sẻ toàn cục $\alpha^{(t+1)}$, và giải bài toán **Wasserstein barycenter** (bằng cách tính trung bình các vector trung bình $v_{i,c}$ và các thừa số Cholesky $L_{i,c}$ để đảm bảo tính nửa xác định dương của ma trận hiệp phương sai) nhằm cập nhật bộ phân phối neo toàn cục mới cho vòng tiếp theo [9, 136].

---

### 4. NEW (ĐIỂM CẢI TIẾN & ĐỘC ĐÁO)
*   **Framework học liên bang ngang đầu tiên phá vỡ giả định đồng nhất không gian:** Trái ngược hoàn toàn với tất cả các mô hình PFL truyền thống (như FedRep, FedAvg-FT, L2GD) bắt buộc các client phải có cấu trúc không gian dữ liệu đầu vào giống hệt nhau, FLIC là framework đầu tiên xử lý thành công dữ liệu thô dị dạng 
*   **Không cần dữ liệu RAD tham chiếu:** Các mô hình đối thủ trước đây như FedHeNN buộc máy chủ phải phân phối một tập dữ liệu thô RAD dùng chung xuống các client, gây bùng nổ số chiều và tiềm ẩn lỗ hổng bảo mật rò rỉ dữ liệu. FLIC tự căn chỉnh mượt mà hoàn toàn trong không gian ẩn thông qua phân phối neo và khoảng cách Wasserstein, tuyệt đối an toàn và tiết kiệm băng thông 
*   **Cơ chế 2 tầng giải quyết đồng thời 2 loại Heterogeneity:** 
    *   *Tầng 1 (Căn chỉnh không gian dị dạng):* Giải quyết triệt để sự dị dạng đặc trưng qua hàm nhúng cục bộ $\phi_i$ và phân phối neo $\mu_c$ 
    *   *Tầng 2 (Học cá nhân hóa):* Giải quyết sự không đồng nhất thống kê còn sót lại (statistical heterogeneity) thông qua học liên bang cá nhân hóa kiểu FedRep (lớp biểu diễn chia sẻ toàn cục $\alpha$ phối hợp lớp phân loại cá nhân cục bộ $\beta_i$)
*   **Định lý hội tụ lý thuyết phi tiệm cận:** Nghiên cứu cung cấp chứng minh toán học cực kỳ vững chắc (Theorem 1). Trong thiết lập kịch bản đơn giản hóa, thuật toán đảm bảo hàm nhúng ước lượng $\hat{\phi}_i$ sẽ hội tụ chính xác về hàm nhúng thực tế up to một phép đảo dấu đường chéo $Q = \text{diag}(\pm 1)$, đồng thời biểu diễn toàn cục học bởi FedRep hội tụ với tốc độ hình học (tuyến tính) về không gian con thực tế
