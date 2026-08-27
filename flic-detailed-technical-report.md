# Báo cáo kỹ thuật chi tiết: FLIC (Personalised Federated Learning on Heterogeneous Feature Spaces)

Bản báo cáo này cung cấp cái nhìn chi tiết và đầy đủ nhất về ba phần cốt lõi của nghiên cứu FLIC: **Tóm tắt (Abstract)**, **Đóng góp khoa học (Contributions)**, và **Phương pháp nghiên cứu (Methodology)**, đi kèm với toàn bộ hệ thống công thức toán học và quy trình thuật toán hoàn chỉnh.

---

## I. TÓM TẮT (ABSTRACT)

Hầu hết các phương pháp học liên bang (Federated Learning - FL) cá nhân hóa hiện nay đều giả định dữ liệu thô của tất cả các thiết bị biên (clients) được định nghĩa trong cùng một không gian con chung (tức là tất cả các client lưu trữ dữ liệu của họ theo cùng một cấu trúc/schema giống nhau) [2]. Đối với các ứng dụng thực tế, giả định này cực kỳ hạn chế vì các client thường sử dụng các hệ thống riêng biệt để thu thập và lưu trữ dữ liệu, dẫn đến việc sử dụng các biểu diễn dữ liệu không đồng nhất hay dị dạng [2].

Để giải quyết khoảng trống công nghệ này, nghiên cứu đề xuất một khung làm việc tổng quát mang tên **FLIC**, cho phép ánh xạ dữ liệu dị dạng của từng client vào một **không gian ẩn chung (common latent space)** thông qua các **hàm nhúng cục bộ (local embedding functions)** [3].
*   Không gian ẩn chung này được học theo phương thức liên bang bằng cách sử dụng **trọng tâm Wasserstein (Wasserstein barycenters)** [3].
*   Trong khi đó, các hàm nhúng cục bộ được huấn luyện song song tại mỗi client thông qua cơ chế **căn chỉnh phân phối (distribution alignment)** [3].

Cơ chế căn chỉnh này được tích hợp mượt mà vào mô hình học liên bang cá nhân hóa giúp hệ thống hoạt động ổn định, đạt hiệu năng phân loại vượt trội so với các mô hình đối sánh trong môi trường dữ liệu dị dạng, đồng thời có bảo đảm lý thuyết vững chắc hỗ trợ cho tính thực tiễn của phương pháp luận [4].

![Figure 1 - Quy trình FLIC](input_file_3.png)  
*Hình 1: Minh họa quy trình FLIC cho $b = 3$ client sở hữu các tập dữ liệu chữ số dị dạng (MNIST ảnh $28\times 28$, USPS ảnh $16\times 16$, SVHN ảnh $32\times 32$) được ánh xạ thông qua các hàm nhúng cục bộ $\phi_i$ về không gian ẩn chung $\Phi$ và căn chỉnh phân phối có điều kiện lớp $\nu_{\phi_i}$ hướng tới các phân phối neo ẩn tương ứng $\mu_c$ [4].*

---

## II. ĐÓNG GÓP KHOA HỌC (CONTRIBUTIONS)

Nghiên cứu đóng góp 4 giá trị khoa học cốt lõi sau [5]:

1.  **Chính thức hóa bài toán học liên bang ngang cá nhân hóa trên không gian đặc trưng dị dạng (Personalised Horizontal FL on Heterogeneous Feature Spaces):** Đây là nghiên cứu đầu tiên định nghĩa toán học chính xác kịch bản học liên bang cá nhân hóa khi các client sở hữu dữ liệu thô sống ở các không gian có số chiều khác nhau hoặc ý nghĩa ngữ nghĩa của các tọa độ vector khác nhau. Khung làm việc tổng quát **FLIC** cho phép mỗi client tận dụng tri thức từ dữ liệu của các client khác ngay cả khi họ không chia sẻ chung định dạng biểu diễn thô [5].
2.  **Khung căn chỉnh phân phối và cấu trúc thuật toán liên bang đồng thời:** Đề xuất cơ chế học các hàm nhúng đặc trưng cục bộ song song với việc học các **phân phối neo ẩn (latent anchor distributions)** toàn cục theo phương thức liên bang (cập nhật cục bộ tại client và tổng hợp trung tâm tại server). Đồng thời, chỉ ra cách tích hợp mượt mà cơ chế này vào bất kỳ thuật toán FL cá nhân hóa nào (như FedRep) để đơn giản hóa việc ứng dụng thực tế [5].
3.  **Cung cấp bảo đảm lý thuyết hội tụ toán học vững chắc:** Chứng minh rằng trong một kịch bản học hồi quy tuyến tính đơn giản hóa, thuật toán FLIC có khả năng khôi phục chính xác không gian con ẩn thực tế (chỉ sai lệch tối đa một phép biến đổi dấu của ma trận trực giao) với tốc độ hội tụ tuyến tính phi tiệm cận (non-asymptotic linear convergence) [5].
4.  **Đánh giá thực nghiệm toàn diện:** Minh chứng hiệu năng ưu việt của FLIC thông qua thực nghiệm trên cả tập dữ liệu mô phỏng (Toy datasets) và các bài toán thực tế (MNIST-USPS, TextCaps đa phương thức ảnh/văn bản) [5].

---

## III. PHƯƠNG PHÁP NGHIÊN CỨU (METHODOLOGY)

### 1. Định nghĩa bài toán (Problem Formulation)
Hệ thống xem xét mô hình học liên bang ngang tập trung gồm một server trung tâm điều phối và $b$ client hoạt động cộng tác [6].
*   Mỗi client $i \in [b]$ sở hữu tập dữ liệu cục bộ riêng tư $D_i = \{(x_i^{(j)}, y_i^{(j)})\}_{j\in[n_i]}$ [6].
*   Dữ liệu thô sống ở các không gian dị dạng: với mỗi client $i$, $x_i^{(j)} \in \mathcal{X}_i \subseteq \mathbb{R}^{k_i}$ (trong đó số chiều đầu vào $k_i$ có thể khác nhau giữa các client, và ý nghĩa ngữ nghĩa của các tọa độ là không tương đồng) [6].
*   Tính không đồng nhất về mặt thống kê (statistical heterogeneity) thể hiện ở chỗ dữ liệu $(x_i^{(j)}, y_i^{(j)}) \sim \nu_i$ tuân theo một độ đo xác suất cục bộ riêng biệt [6].
*   Có sự xuất hiện của dịch chuyển xác suất tiên nghiệm (prior probability shift) khi mỗi client chỉ sở hữu dữ liệu của một số lớp cụ thể $y_i^{(j)} \in \mathcal{Y}_i \subseteq [C]$ với $C$ là tổng số lớp [6].

### 2. Hàm mục tiêu tối ưu hóa (Objective Function)
Để giải quyết sự dị dạng của không gian đặc trưng thô, FLIC sử dụng các hàm nhúng đặc trưng cục bộ học được $\{\phi_i: \mathcal{X}_i \to \Phi\}_{i \in [b]}$ nhằm ánh xạ dữ liệu của client về không gian ẩn chung $\Phi \subseteq \mathbb{R}^k$ có số chiều $k$ cố định [7].

#### Hàm khớp dữ liệu cục bộ (Empirical Risk):
Mục tiêu là tìm kiếm các hàm nhúng $\phi_i$ và các mô hình cục bộ cá nhân hóa được tham số hóa bởi $\theta_i \in \mathbb{R}^{d_i}$ để tối ưu hóa hàm mất mát phân loại [7]:
$$f(\theta_{1:b}, \phi_{1:b}) = \sum_{i=1}^b \omega_i f_i(\theta_i, \phi_i) \quad (1)$$
Trong đó, $\sum_{i=1}^b \omega_i = 1$ là trọng số đóng góp của các client. Với mỗi client, hàm loss cục bộ được tính bằng [7]:
$$f_i(\theta_i, \phi_i) = \frac{1}{n_i} \sum_{j=1}^{n_i} \ell \left( y_i^{(j)}, g_{\theta_i}^{(i)} \left[ \phi_i \left( x_i^{(j)} \right) \right] \right) \quad (2)$$
*   **Giải thích:** $\ell(\cdot, \cdot)$ đại diện cho hàm loss phân loại (ví dụ: Cross-Entropy hoặc $\ell_2$ norm) [8]. $g_{\theta_i}^{(i)}$ là mô hình cục bộ cá nhân hóa nhận đầu vào từ không gian ẩn $\phi_i(x_i^{(j)}) \in \Phi$ [8].

#### Hàm mục tiêu toàn cục tích hợp căn chỉnh phân phối (Global Objective):
Để tránh dữ liệu cùng nhãn lớp từ các client khác nhau bị phân tán rải rác mất đi cấu trúc ngữ nghĩa trong không gian ẩn $\Phi$, FLIC ràng buộc phân phối đặc trưng nhúng có điều kiện lớp của các client $\nu_{\phi_i}^{(c)}$ tiến sát tới một tập hợp $C$ phân phối neo ẩn tham chiếu được chia sẻ toàn cầu $\{\mu_c\}_{c\in[C]}$ [8].

Bài toán tối ưu hóa toàn cục được định nghĩa như sau [9]:
$$\theta_{1:b}^*, \phi_{1:b}^*, \mu_{1:C}^* = \arg\min_{\theta_{1:b}, \phi_{1:b}, \mu_{1:C}} \sum_{i=1}^b F_i(\theta_i, \phi_i, \mu_{1:C})$$
Trong đó, hàm mục tiêu cục bộ điều hòa của mỗi client $i$ là [9]:
$$F_i(\theta_i, \phi_i, \mu_{1:C}) = \omega_i f_i(\theta_i, \phi_i) + \lambda_1 \omega_i \sum_{c\in\mathcal{Y}_i} W_2^2 \left( \mu_c, \nu_{\phi_i}^{(c)} \right) + \lambda_2 \omega_i \sum_{c\in\mathcal{Y}_i} \frac{1}{J} \sum_{j=1}^J \ell \left( c, g_{\theta_i}^{(i)} \left[ Z_c^{(j)} \right] \right) \quad (3)$$

*   **Giải thích chi tiết từng thành phần trong công thức (3) [10]:**
    1.  **Thành phần thứ nhất $\omega_i f_i(\theta_i, \phi_i)$:** Đo mức độ khớp dữ liệu phân loại trên các mẫu thực tế cục bộ sau khi nhúng.
    2.  **Thành phần thứ hai $\lambda_1 \omega_i \sum_{c\in\mathcal{Y}_i} W_2^2 \left( \mu_c, \nu_{\phi_i}^{(c)} \right)$:** Số hạng phạt điều hòa dựa trên **khoảng cách Wasserstein bậc 2 ($W_2^2$)**. Số hạng này ép phân phối đặc trưng nhúng có điều kiện lớp của client ($c \in \mathcal{Y}_i$) tiệm cận với phân phối neo ẩn chung tương ứng $\mu_c$ để đồng nhất ngữ nghĩa của "bộ dịch" giữa các client. $\lambda_1 > 0$ là siêu tham số kiểm soát mức độ căn chỉnh.
    3.  **Thành phần thứ ba $\lambda_2 \omega_i \sum_{c\in\mathcal{Y}_i} \frac{1}{J} \sum_{j=1}^J \ell \left( c, g_{\theta_i}^{(i)} \left[ Z_c^{(j)} \right] \right)$:** Số hạng tùy chọn giúp hiệu chuẩn trực tiếp bộ phân loại cá nhân hóa với phân phối neo thông qua $J$ mẫu $\{Z_c^{(j)}\}$ được rút trực tiếp từ $\mu_c$. Thành phần này giúp giải quyết hiện tượng trôi hiệp biến (covariate shift) và giảm sự mơ hồ giữa các lớp trong không gian ẩn. $\lambda_2 > 0$ là tham số điều hòa.

### 3. Giả định Gauss và Công thức đóng Wasserstein bậc 2
Để việc tính toán và tối ưu hóa gradient diễn ra nhanh chóng, nghiên cứu đưa ra hai thiết kế giả định Gauss quan trọng [11]:
1.  **Giả định phân phối neo Gauss:** $\mu_c = \mathcal{N}(v_c, \Sigma_c)$ với $v_c \in \mathbb{R}^k$. Ma trận hiệp phương sai được viết dưới dạng Cholesky $\Sigma_c = L_c L_c^\top$ (với $L_c$ khả nghịch bằng cách cộng một lượng nhỏ $\varepsilon I_k$) để đảm bảo điều kiện nửa xác định dương. Mẫu được sinh hiệu quả thông qua: $Z_c^{(j)} = v_c + L_c \xi_c^{(j)}$ với $\xi_c^{(j)} \sim \mathcal{N}(0_k, I_k)$ [11].
2.  **Xấp xỉ phân phối nhúng cục bộ bằng Gauss:** Phân phối nhúng thực nghiệm của client cũng được xấp xỉ bằng Gauss: $\hat{\nu}_{\phi_i}^{(c)} = \mathcal{N}(\hat{m}_i^{(c)}, \hat{\Sigma}_i^{(c)})$ [11].

Nhờ hai giả định Gauss này, **khoảng cách Wasserstein bậc 2** có công thức đóng cực kỳ tiện lợi giúp tránh các phép tính tích phân phức tạp [12]:
$$W_2^2 \left( \mu_c, \nu_{\phi_i}^{(c)} \right) = \left\| v_c - m_i^{(c)} \right\|^2 + \mathfrak{B}^2 \left( \Sigma_c, \Sigma_i^{(c)} \right) \quad (4)$$
*   **Giải thích:** $\|\cdot\|^2$ là chuẩn Euclidean bình phương của sai lệch giá trị trung bình. $\mathfrak{B}(\cdot, \cdot)$ biểu thị khoảng cách Bures giữa hai ma trận hiệp phương sai xác định dương [12].

### 4. Khung xương học liên bang cá nhân hóa (PFL) tích hợp
FLIC có khả năng tích hợp mượt mà với nhiều mô hình PFL khác nhau thông qua việc cấu trúc lại các tham số cục bộ và toàn cục [13]. Nghiên cứu minh họa cụ thể với thuật toán **FedRep** (chia tách mô hình thành biểu diễn chia sẻ toàn cục $\alpha$ và phân loại cá nhân hóa cục bộ $\beta_i$) [13]:
*   **Mô hình cục bộ:** $g_{\theta_i}^{(i)} = g_{\beta_i}^{(i)} \circ g_{\alpha}$ [13].
*   **Tham số mô hình:** $\theta_i = [\alpha, \beta_i]$ [13].

---

## IV. THUẬT TOÁN CHI TIẾT CỦA FLIC (ALGORITHM S2)

Cơ chế cập nhật tham số và quy trình giao tiếp client-server của FLIC được trình bày chi tiết dưới đây (ứng với biến thể tích hợp FedRep) [14]:

### 1. Mã giả chi tiết của Thuật toán FLIC (Detailed Algorithm S2)

```text
Yêu cầu khởi tạo: α^{(0)}, μ^{(0)}_{1:C} = [Σ^{(0)}_{1:C}, v^{(0)}_{1:C}] với Σ^{(0)}_c = L^{(0)}_c [L^{(0)}_c]^T, φ^{(0,0)}_{1:b}, β^{(0,0)}_{1:b} và tốc độ học η ≤ η_max.

1: For t = 0 to T − 1 do (Vòng lặp truyền thông toàn cục)
2:   Lấy mẫu ngẫu nhiên tập hợp At+1 gồm các client đang hoạt động với tỷ lệ r.
3:   For i ∈ At+1 do (Mỗi client được chọn chạy song song)
4:     Server trung tâm gửi biểu diễn chia sẻ α^{(t)} và phân phối neo μ^{(t)}_{1:C} xuống client.
5:     // CẬP NHẬT CÁC THAM SỐ CỤC BỘ (Hàm nhúng φ_i và classifier đầu β_i)
6:     For m = 0 to M − 1 do (M bước huấn luyện cục bộ)
7:       Lấy mẫu một batch dữ liệu fresh I^{(i,m)}_{t+1} gồm n'_i mẫu.
8:       Sinh các mẫu neo ẩn Z^{(j,t,m)}_c ~ μ^{(t)}_c cho j ∈ I^{(i,m)}_{t+1} và c ∈ Y_i 
         thông qua công thức: Z^{(j,t,m)}_c = v^{(t)}_c + L^{(t)}_c ξ^{(t,m)}_i với ξ^{(t,m)}_i ~ N(0_k, I_k).
9:       Cập nhật hàm nhúng cục bộ φ_i bằng gradient descent:
         φ^{(t, m+1)}_i = φ^{(t,m)}_i - η * (n_i / |I^{(i,m)}_{t+1}|) * \sum_{j ∈ I^{(i,m)}_{t+1}} \nabla_{φ_i} \ell(y_i^{(j)}, g^{(i)}_{[α^{(t)}, β^{(t,m)}_i]} [φ^{(t,m)}_i(x_i^{(j)})]) 
                         - η * λ_1 * \sum_{c ∈ Y_i} \nabla_{φ_i} W_2^2(μ^{(t)}_c, ν^{(c)}_{φ^{(t,m)}_i}).
10:      Cập nhật classifier đầu cục bộ β_i bằng gradient descent:
         β^{(t, m+1)}_i = β^{(t,m)}_i - η * (n_i / |I^{(i,m)}_{t+1}|) * \sum_{j ∈ I^{(i,m)}_{t+1}} \nabla_{β_i} \ell(y_i^{(j)}, g^{(i)}_{[α^{(t)}, β^{(t,m)}_i]} [φ^{(t,m)}_i(x_i^{(j)})])
                         - η * λ_2 * \sum_{c ∈ Y_i} \nabla_{β_i} \ell(y_i^{(j)}, g^{(i)}_{[α^{(t)}, β^{(t,m)}_i]} [Z^{(j,t,m)}_c]).
11:    End for
12:    Lưu lại tham số cục bộ cuối cùng: φ^{(t+1,0)}_i = φ^{(t,M)}_i và β^{(t+1,0)}_i = β^{(t,M)}_i.
13:    
14:    // CẬP NHẬT CÁC THAM SỐ TOÀN CỤC Ở MỨC CỤC BỘ (Tránh hiện tượng client drift)
15:    Cập nhật biểu diễn chia sẻ cục bộ α_i:
         α^{(t+1)}_i = α^{(t)} - η * (n_i / |I^{(i,M)}_{t+1}|) * \sum_{j ∈ I^{(i,M)}_{t+1}} \nabla_{α} \ell(y_i^{(j)}, g^{(i)}_{[α^{(t)}, β^{(t,M)}_i]} [φ^{(t,M)}_i(x_i^{(j)})])
                       - η * λ_2 * \sum_{c ∈ Y_i} \nabla_{α} \ell(y_i^{(j)}, g^{(i)}_{[α^{(t)}, β^{(t,M)}_i]} [Z^{(j,t,m)}_c]).
16:    For c = 1 to C do
17:      Ước lượng trung bình thực nghiệm mˆ^{(c,t)}_i và hiệp phương sai Σˆ^{(c,t)}_i của đặc trưng nhúng bằng φ^{(t,M)}_i.
18:      Cập nhật vector trung bình neo cục bộ v_{i,c}:
         v^{(t+1)}_{i,c} = v^{(t)}_c - η * λ_1 * \nabla_{v_c} ||v^{(t)}_c - mˆ^{(c,t)}_i||^2
                           - η * λ_2 * (n_i / |I^{(i,m)}_{t+1}|) * \sum_{j ∈ I^{(i,m)}_{t+1}} \nabla_{v_c} \ell(y_i^{(j)}, g^{(i)}_{[α^{(t)}, β^{(t,M)}_i]} [Z^{(j,t,m)}_c]).
19:      Cập nhật thừa số Cholesky hiệp phương sai neo cục bộ L_{i,c}:
         L^{(t+1)}_{i,c} = L^{(t)}_c - η * λ_1 * \nabla_{L_c} B^2(L^{(t)}_c [L^{(t)}_c]^T, Σˆ^{(c,t)}_i)
                           - η * λ_2 * (n_i / |I^{(i,m)}_{t+1}|) * \sum_{j ∈ I^{(i,m)}_{t+1}} \nabla_{L_c} \ell(y_i^{(j)}, g^{(i)}_{[α^{(t)}, β^{(t,M)}_i]} [Z^{(j,t,m)}_c]).
20:    End for
21:    // TRUYỀN THÔNG VỀ SERVER
22:    Client gửi α^{(t+1)}_i, v^{(t+1)}_{i,1:C} và L^{(t+1)}_{i,1:C} về server trung tâm.
23:  End for
24:  
25:  // TỔNG HỢP TOÀN CỤC TRÊN SERVER
26:  Cập nhật biểu diễn chia sẻ toàn cục bằng trung bình trọng số:
     α^{(t+1)} = (b / |At+1|) * \sum_{i ∈ At+1} ω_i * α^{(t+1)}_i.
27:  Cập nhật phân phối neo toàn cục mới thông qua Trọng tâm Wasserstein (Wasserstein Barycenter):
     For c = 1 to C do
28:    v^{(t+1)}_c = (b / |At+1|) * \sum_{i ∈ At+1} ω_i * v^{(t+1)}_{i,c}.
29:    L^{(t+1)}_c = (b / |At+1|) * \sum_{i ∈ At+1} ω_i * L^{(t+1)}_{i,c} và đặt Σ^{(t+1)}_c = L^{(t+1)}_c [L^{(t+1)}_c]^T.
30:  End for
31: End for
Đầu ra đảm bảo: Các bộ tham số tối ưu α^{(T)}, μ^{(T)}_{1:C}, φ^{(T,0)}_{1:b}, β^{(T,0)}_{1:b}.
```

### 2. Thuyết minh luồng truyền thông và Cơ chế hoạt động của thuật toán
*   **Phân phối thông tin (Server -> Client):** Ở mỗi vòng truyền thông $t$, các client được chọn sẽ nhận mô hình chia sẻ toàn cầu $\alpha^{(t)}$ và tập hợp phân phối neo ẩn chung $\mu_{1:C}^{(t)}$ làm gốc hiệu chuẩn [14].
*   **Cập nhật song song tại client:** Để giảm chi phí truyền thông và nghẽn băng thông, mỗi client thực hiện $M$ bước huấn luyện cục bộ (SGD) để cập nhật "máy dịch" cục bộ (hàm nhúng $\phi_i$) và "đầu phân loại cá nhân hóa" $\beta_i$ [14].
*   **Căn chỉnh phân phối cục bộ (Distribution Alignment):** Trong quá trình huấn luyện cục bộ, gradient của khoảng cách Wasserstein ($W_2^2$) (Equation 4) hoạt động như một động lực ép các đặc trưng nhúng cục bộ cùng một lớp hội tụ về vùng không gian được đánh dấu bởi phân phối neo [14].

![Sự hội tụ của các lớp qua t-SNE](input_file_7.png)  
*Hình 2: Trực quan hóa t-SNE quá trình căn chỉnh phân phối trên tập dữ liệu Toy LM qua 10, 50, và 100 epoch huấn luyện. Các hình tròn/tam giác đại diện cho dữ liệu nhúng của các client khác nhau, ngôi sao đại diện cho tâm phân phối neo $\mu_c$. Ta thấy dữ liệu từ các không gian dị dạng ban đầu của các client đã dần hội tụ chuẩn xác về đúng tâm phân phối neo tương ứng [15].*

*   **Tính toán cập nhật toàn cầu một bước:** Sau khi huấn luyện cục bộ xong, client thực hiện duy nhất 1 bước cập nhật cho các biến toàn cục ($\alpha_i, \mu_{i}$) trước khi gửi về server [15]. Việc thực hiện cập nhật toàn cục chỉ 1 bước này đóng vai trò quan trọng trong việc triệt tiêu hiện tượng lệch client (client drift) [15].
*   **Tổng hợp trọng tâm Wasserstein trên Server (Aggregation):** Nhận được các tham số neo ẩn cục bộ gửi lên, server tiến hành giải bài toán trọng tâm Wasserstein thông qua việc tính trung bình các vector trung bình $v_{i,c}$ và ma trận hiệp phương sai dạng Cholesky $L_{i,c}$ của các client [15]. Cơ chế này giúp sinh ra một bộ phân phối neo toàn cầu mới tối ưu và cân bằng nhất cho vòng truyền thông tiếp theo [15].

![Độ hội tụ lý thuyết và Thực nghiệm](input_file_4.png)  
*Hình 3: Kết quả phân tích lý thuyết và thực tế của FLIC. Bên trái biểu thị các điểm đặc trưng nhúng thực tế hội tụ khớp với Oracle lý thuyết. Bên phải biểu thị đồ thị độ chính xác hội tụ của khoảng cách góc chính (principal angle distance) tiệm cận về mức tối ưu tuyến tính $10^{-8}$ chỉ sau khoảng 30 epoch [16].*
