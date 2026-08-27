# TÀI LIỆU TỔNG HỢP: FLIC - BẢN DỊCH GOOGLE DOC VÀ PHẦN GIỚI THIỆU (INTRODUCTION)

## PHẦN 1: DỮ LIỆU TỪ GOOGLE DOC (AGAIN_FLIC_27/8_10p.m)

### 1. Trả lời cho câu hỏi vector là gì?

#### a) Vector là gì?
Vector đơn giản là một dãy các con số có thứ tự.
*   **Ví dụ:** 
    *   $x = [2, 3]$ là một vector có 2 chiều.
    *   $x = [2, 3, 5, 7]$ là một vector có 4 chiều.

#### b) Vector gồm những gì?
*   **Ví dụ mô tả một người:** Tưởng tượng một người cao 173 cm, nặng 65 kg, 20 tuổi $\rightarrow$ Ta có thể viết được vector $x = [173, 65, 20]$.
    *   Trong đó: $173$ = feature 1, $65$ = feature 2, $20$ = feature 3.
    *   Ta gọi: $x \in \mathbb{R}^3$. Nghĩa là vector $x$ nằm trong không gian 3 chiều.

#### c) "Feature" chính là cái gì?
Feature là một đặc trưng dùng để mô tả dữ liệu.
*   **Ví dụ mô tả một ngôi nhà:** $x = [120, 3, 2, 150]$.
    *   Có thể mang ý nghĩa: `[diện tích, phòng ngủ, số phòng tắm, khoảng cách đến trung tâm TP]`.
    *   $\rightarrow$ Vector chỉ là cái hộp chứa các giá trị feature theo một thứ tự nhất định.

#### d) Trông như thế nào trong máy tính?
Trong Python, chúng ta biểu diễn vector bằng mảng:
```python
import numpy as np
x = np.array([2, 3, 5, 7])  # Tương đương với một điểm trong không gian 4 chiều
```

#### e) Biểu diễn nhiều vector dưới dạng ma trận
Giả sử có 3 mẫu dữ liệu (samples):
*   $x_1 = [1, 2, 3]$
*   $x_2 = [2, 4, 1]$
*   $x_3 = [7, 8, 9]$

Chúng ta có thể gộp lại thành một ma trận dữ liệu $X$:
$$X = \begin{bmatrix} 1 & 2 & 3 \\ 2 & 4 & 1 \\ 7 & 8 & 9 \end{bmatrix}$$

Trong Machine Learning, dữ liệu thường xuất hiện dưới dạng ma trận này:
*   **Mỗi hàng** = một sample (mẫu dữ liệu).
*   **Mỗi cột** = một feature (đặc trưng).

#### f) Hình ảnh cũng là vector
Giả sử có một ảnh đen trắng (grayscale / ảnh xám) kích thước $2 \times 2$ pixel:
*   Mỗi pixel là một giá trị độ sáng (ví dụ: $10, 50, 100, 200$).
*   Ta có thể trải phẳng ảnh thành một vector: $x = [10, 50, 100, 200] \in \mathbb{R}^4$ (vector 4 chiều).
*   Như vậy, một ảnh kích thước $28 \times 28$ pixel $\rightarrow 28 \times 28 = 784$ pixel $\rightarrow$ Biến thành vector $x \in \mathbb{R}^{784}$ gồm $784$ con số: $[x_1, x_2, \dots, x_{784}]$.

---

### 2. Vấn đề trong nghiên cứu FLIC

Trong nghiên cứu FLIC sử dụng các tập dữ liệu MNIST, USPS và SVHN, các client sở hữu dữ liệu có kích thước ảnh đầu vào khác nhau:
*   **MNIST:** Ảnh kích thước $28 \times 28 \rightarrow 784$ chiều.
*   **USPS:** Ảnh kích thước $16 \times 16 \rightarrow 256$ chiều.
*   **SVHN:** Ảnh kích thước $32 \times 32 \rightarrow 1024$ chiều.

#### Vấn đề khi khác số chiều đặc trưng:
Giả sử Client A có dữ liệu $x_A \in \mathbb{R}^{784}$ và Client B có dữ liệu $x_B \in \mathbb{R}^{256}$. Nếu muốn tính khoảng cách Euclid trực tiếp $\|x_A - x_B\|$, chúng ta **không thể thực hiện được** vì hai vector không cùng kích thước đầu vào. 

Đây chính là bài toán cốt lõi mà FLIC giải quyết: **các client có không gian đặc trưng dị dạng (heterogeneous feature spaces) khác nhau về số chiều hoặc ý nghĩa của các tọa độ đặc trưng.**

#### Giải pháp đề xuất của FLIC:
FLIC đề xuất đưa dữ liệu của client qua một hàm nhúng (embedding function) $\phi_i(x_i)$ để tạo ra vector đặc trưng ẩn $z_i = \phi_i(x_i)$ nằm trong một **không gian ẩn chung (common latent space) $\Phi$**:
$$\phi_i: \mathcal{X}_i \rightarrow \Phi$$

*   **Ví dụ hoạt động:**
    *   Client A ($x_A \in \mathbb{R}^{784}$) $\xrightarrow{\phi_A}$ $z_A \in \mathbb{R}^{100}$
    *   Client B ($x_B \in \mathbb{R}^{256}$) $\xrightarrow{\phi_B}$ $z_B \in \mathbb{R}^{100}$
    *   $\rightarrow$ Bây giờ, cả $z_A$ và $z_B$ đều nằm trong không gian chung $\mathbb{R}^{100}$, cho phép so sánh, tính toán khoảng cách và tổng hợp mô hình trực tiếp.

#### Chi tiết về cơ chế nhúng (Embedding):
*   **Cách đơn giản nhất:** Nhân vector đầu vào với một ma trận trọng số $W$.
    *   Giả sử đầu vào $x \in \mathbb{R}^5$ (vector kích thước $1 \times 5$). Chúng ta khởi tạo ma trận trọng số $W$ kích thước $2 \times 5$ để ánh xạ từ $\mathbb{R}^5 \rightarrow \mathbb{R}^2$.
    *   Công thức: $z = Wx + b$.
*   **Ví dụ cụ thể:**
    *   Đầu vào $x = [10, 20, 30, 40, 50]^\top$
    *   Ma trận trọng số $W = \begin{bmatrix} 0.1 & 0.2 & 0.1 & 0.3 & 0.3 \\ 0.3 & 0.1 & 0.2 & 0.1 & 0.3 \end{bmatrix}$
    *   Các thành phần vector đầu ra $z \in \mathbb{R}^2$:
        *   $z_1 = 0.1(10) + 0.2(20) + 0.1(30) + 0.3(40) + 0.3(50) = 35$
        *   $z_2 = 0.3(10) + 0.1(20) + 0.2(30) + 0.1(40) + 0.3(50) = 30$
    *   $\rightarrow$ Kết quả thu được một vector $z = [35, 30]^\top$ chỉ có 2 chiều. Mỗi chiều mới là tổ hợp tuyến tính của tất cả các chiều cũ.
*   **Cách tìm ra ma trận $W$:** Trong thực tế, chúng ta không tự chọn các con số trong ma trận $W$ theo cách thủ công. Chúng ta sử dụng một mạng nơ-ron (Neural Network) với nhiều lớp (layers) để tự động học ra các trọng số này:
    $$\text{Input (784 chiều)} \rightarrow \text{Layer 1 (256 chiều)} \rightarrow \text{Layer 2 (64 chiều)} \rightarrow \text{Output } K \text{ chiều}$$
    *   Mỗi layer sẽ có một ma trận trọng số $W$ và vector bias $b$ tương ứng.

#### Khái niệm Latent Space (Không gian ẩn):
Latent space là không gian biểu diễn ẩn trừu tượng mà mô hình học máy tự học ra. 
*   **Ví dụ:** Ảnh MNIST từ $784$ số sau khi đi qua hàm nhúng $\phi(x)$ chỉ còn $20$ số ($z \in \mathbb{R}^{20}$). 
*   Các chiều trong không gian $20$ chiều này không nhất thiết phải mang ý nghĩa vật lý rõ ràng (như chiều cao, cân nặng), mà là các đặc trưng biểu diễn trừu tượng giúp phân biệt các chữ số hiệu quả nhất.

---

### 3. Tóm tắt lý thuyết (Abstract của bài báo)
Hầu hết các phương pháp học liên bang (FL) cá nhân hóa đều giả định rằng dữ liệu thô của tất cả các client được định nghĩa trên một không gian con chung, tức là tất cả client lưu trữ dữ liệu theo cùng một schema/cấu trúc. Đối với các ứng dụng trong thế giới thực, giả định này khá hạn chế, bởi các client có thể sử dụng những hệ thống riêng để thu thập và lưu trữ dữ liệu, dẫn đến việc dữ liệu được biểu diễn theo những cách không đồng nhất.

Nhằm giải quyết khoảng trống này, chúng tôi đề xuất một framework tổng quát có tên **FLIC**, trong đó dữ liệu của mỗi client được ánh xạ vào một không gian đặc trưng chung thông qua các hàm embedding cục bộ. Không gian đặc trưng chung này được học theo cách thức liên kết giữa các client (federated) bằng cách sử dụng **Wasserstein barycenter**; trong khi các hàm embedding cục bộ được huấn luyện trên từng client thông qua cơ chế **căn chỉnh phân phối (distribution alignment)**.

Chúng tôi tích hợp cơ chế căn chỉnh phân phối này vào một phương pháp Federated Learning và trình bày các thuật toán của FLIC. Chúng tôi so sánh hiệu năng của phương pháp với các phương pháp FL chuẩn được sử dụng trong những tình huống mà các client có không gian đặc trưng đầu vào không đồng nhất. Ngoài ra, chúng tôi cung cấp các phân tích lý thuyết nhằm hỗ trợ và làm rõ tính phù hợp của phương pháp được đề xuất.

---

## PHẦN 2: PHẦN GIỚI THIỆU CHI TIẾT (1. INTRODUCTION — GIỚI THIỆU)

### 1. Giới thiệu chung về Federated Learning (FL)
Federated Learning (FL) là một mô hình học máy trong đó các mô hình được huấn luyện từ nhiều tập dữ liệu độc lập thuộc sở hữu của những tác nhân riêng biệt, được gọi là client, mà không yêu cầu đưa dữ liệu thô về một máy chủ trung tâm, thậm chí không cần chia sẻ dữ liệu thô theo bất kỳ hình thức nào. Framework này gần đây đã thu hút sự quan tâm mạnh mẽ cả trong công nghiệp lẫn nghiên cứu học thuật.

Một trong những lý do chính giúp FL phát triển nhanh chóng là:
*   Tránh được chi phí truyền thông lớn do phải chuyển dữ liệu thô qua mạng.
*   Cho phép tất cả client cùng hưởng lợi từ việc tham gia vào quá trình học tập chung mà không sợ mất tính cạnh tranh hay lộ thông tin riêng tư.
*   Cung cấp các bảo đảm bảo mật và tính bảo mật thông tin (confidentiality) ở mức cơ bản.
*   Các bảo đảm này có thể được tăng cường thêm bằng các công nghệ nâng cao quyền riêng tư (privacy-enhancing technologies) như: **differential privacy (bảo mật vi sai)** hoặc **secure multi-party computation (tính toán đa bên an toàn)**.

Đặc điểm cốt lõi của FL là client vẫn giữ quyền sở hữu dữ liệu của mình. Đồng thời, FL về mặt cấu trúc đã áp dụng nguyên tắc tối thiểu hóa trao đổi dữ liệu bằng cách chỉ truyền đi những cập nhật cần thiết của các mô hình đang được học (như các gradient hoặc trọng số mô hình).

Tùy thuộc vào cách dữ liệu được phân chia và ứng dụng mục tiêu, nhiều dạng FL đã được đề xuất:
*   **Horizontal Federated Learning (HFL - Học liên bang ngang):** Các client sở hữu những mẫu dữ liệu tương ứng với những người dùng khác nhau nhưng có chung một không gian đặc trưng.
*   **Vertical Federated Learning (VFL - Học liên bang dọc):** Các client sở hữu những tập con đặc trưng khác nhau nhưng tương ứng với cùng những người dùng.

Gần đây, các nghiên cứu về HFL tập trung nhiều vào **Personalised Federated Learning (PFL - Học liên bang cá nhân hóa)** nhằm giải quyết vấn đề không đồng nhất về mặt thống kê (statistical heterogeneity) — tức sự khác biệt về phân phối dữ liệu (nhãn hoặc đặc trưng) giữa các client — bằng cách sử dụng các mô hình cục bộ được điều chỉnh linh hoạt cho dữ liệu riêng của từng client.

---

### 2. Vấn đề của các phương pháp hiện tại
Các phương pháp horizontal personalised FL hiện có đều giả định rằng dữ liệu thô trên tất cả client có cùng cấu trúc và được định nghĩa trong một không gian đặc trưng chung. Tuy nhiên, trong thực tế, dữ liệu được thu thập bởi các client có thể có những cấu trúc rất khác nhau do:
*   Các client sử dụng các hệ thống phần cứng, cảm biến hoặc quy trình nghiệp vụ khác nhau để thu thập thông tin.
*   Một số feature quan trọng bị thiếu hoặc không được lưu trữ ở một số client nhất định.
*   Dữ liệu đặc trưng đã bị biến đổi qua các quy trình tiền xử lý cục bộ khác nhau, chẳng hạn bằng: chuẩn hóa (normalization), co giãn (scaling), hoặc tổ hợp tuyến tính (linear combination).

Do đó, một vấn đề vô cùng quan trọng xuất hiện: **Làm thế nào thực hiện Federated Learning khi các client có những không gian đặc trưng không đồng nhất (heterogeneous feature spaces)?**

Trong nghiên cứu này, *"heterogeneous feature spaces"* được hiểu là các client có thể có dữ liệu đầu vào khác nhau về **số chiều (dimensions)**, hoặc **ý nghĩa của các tọa độ trong vector đặc trưng**. Theo hiểu biết của các tác giả, FLIC là framework Personalised Federated Learning đầu tiên được thiết kế chuyên biệt để giải quyết trọn vẹn tình huống khó khăn này.

---

### 3. Phương pháp đề xuất (Proposed Approach)
Framework và thuật toán FLIC được trình bày dựa trên một ý tưởng cốt lõi: **Trước khi thực hiện quá trình FL hiệu quả, cần ánh xạ dữ liệu thô của các client vào một không gian con chung.**

Đây là một bước bắt buộc và tối quan trọng trước khi tiến hành học liên bang. Lý do là khi dữ liệu của các client đã được đưa về cùng một không gian ẩn chung có số chiều tương đồng, server trung tâm mới có thể xây dựng một cơ chế tổng hợp (aggregation) có ý nghĩa cho các tham số mô hình (chẳng hạn như tính trung bình trọng số - weighted averaging). 

Nói cách khác, nếu mỗi client sử dụng một hệ tọa độ hoàn toàn khác nhau thì việc lấy trung bình tham số giữa các mô hình cục bộ sẽ bị mất đi ý nghĩa hình học và toán học. Do đó, các tác giả ánh xạ dữ liệu thô của từng client vào một không gian ẩn (latent space) chung có số chiều thấp thông qua các hàm nhúng đặc trưng có thể học được và được huấn luyện cục bộ:
$$\{\phi_i\}_{i\in[b]}$$
trong đó $\phi_i$ là hàm nhúng của client $i$.

#### Tại sao chỉ đưa dữ liệu về cùng một không gian là chưa đủ?
Sau khi thực hiện nhúng (embedding), có một vấn đề sâu sắc hơn phát sinh: làm sao để những dữ liệu mang cùng thông tin ngữ nghĩa (semantic information) — ví dụ cùng một nhãn lớp (label) — được đưa đến cùng một vùng cục bộ trong latent space?

*   **Ví dụ minh họa:** Giả sử Client 1 sở hữu ảnh số 7 (MNIST kích thước $28 \times 28$), Client 2 cũng sở hữu ảnh số 7 (USPS kích thước $16 \times 16$). Cho dù hai client ban đầu sử dụng hai không gian đặc trưng thô có kích thước hoàn toàn khác nhau, sau khi đi qua các hàm nhúng $\phi_1$ và $\phi_2$, các ảnh biểu diễn chữ số 7 của cả hai bên vẫn nên nằm gần nhau trong không gian ẩn chung $\Phi$. Nếu không làm được điều này, server sẽ không thể học được một mô hình phân loại toàn cục có ý nghĩa ngữ nghĩa.

Để đảm bảo tính chất đồng nhất ngữ nghĩa này, các tác giả đề xuất căn chỉnh các phân phối đặc trưng sau nhúng của các client thông qua một **phân phối neo ẩn (latent anchor distribution)** được học liên bang và chia sẻ chung giữa các client.

#### Latent Anchor Distribution (Phân phối neo ẩn):
Có thể hình dung anchor distribution giống như một hệ tọa độ chuẩn hoặc một “thước đo chung vạn năng” để các client căn chỉnh dữ liệu của mình. 
*   Mỗi client có một phân phối dữ liệu riêng biệt.
*   Client đưa dữ liệu của mình qua hàm nhúng cục bộ $\phi_i$.
*   Phân phối sau nhúng được căn chỉnh ép tiến sát tới phân phối neo chung $\mu_c$ cho từng lớp $c$.
*   Bản thân phân phối neo $\mu_c$ cũng được học theo cơ chế liên bang:
    1.  Mỗi client cập nhật phiên bản phân phối neo cục bộ của mình bằng cách làm cho nó gần hơn với phân phối dữ liệu sau nhúng cục bộ.
    2.  Server trung tâm thu thập và tìm phần tử trung bình tối ưu nhất — hay còn gọi là **trọng tâm Wasserstein (Wasserstein barycenter)** — từ các phân phối neo cục bộ này để tạo ra phân phối neo toàn cục mới.
    3.  Nói cách khác, mỗi client đưa ra một "ý kiến" về việc không gian ẩn nên trông như thế nào, và server tổng hợp các ý kiến đó thành một latent space chung thống nhất.

#### Cá nhân hóa (Personalization):
Khi cơ chế căn chỉnh phân phối (distribution alignment) dựa trên các hàm nhúng cục bộ và phân phối neo ẩn đã được xác định, nó sẽ được tích hợp mượt mà vào một framework Personalized FL. 

Phần cá nhân hóa đóng vai trò giải quyết phần không đồng nhất thống kê (statistical heterogeneity) còn sót lại giữa dữ liệu của các client. Các tác giả không giả định rằng việc căn chỉnh phân phối sẽ xóa nhòa hoàn toàn mọi sự khác biệt giữa các client. Thay vào đó, quy trình hoạt động của FLIC chia làm 2 giai đoạn rõ rệt:
1.  **Giai đoạn 1:** FLIC giải quyết triệt để sự khác biệt về không gian đặc trưng dị dạng đầu vào (feature space heterogeneity).
2.  **Giai đoạn 2:** Học liên bang cá nhân hóa (Personalised FL) tiếp tục xử lý sự khác biệt về phân phối nhãn/thống kê (statistical heterogeneity).

Trong bài báo này, các tác giả tích hợp framework căn chỉnh này vào một phương pháp personalized FL tương tự phương pháp của Collins et al. (2021).

---

### 4. Các ý tưởng liên quan (Related Ideas)
Ý tưởng giải quyết FL trên các không gian đặc trưng không đồng nhất đã được nghiên cứu một phần trong các công trình trước đây:
*   **Về mặt lý thuyết:** Các nghiên cứu về khoảng cách Gromov-Wasserstein hoặc các biến thể của nó tìm cách so sánh các phân phối nằm trong những không gian không thể so sánh trực tiếp, nhưng chủ yếu tập trung trong bối cảnh học tập tập trung (centralised learning), chứ không phải môi trường học liên bang (FL).
*   **Các hướng nghiên cứu khác:** Sử dụng các ý tưởng tương tự về căn chỉnh phân phối (distribution alignment) trong autoencoder, nhúng từ (word embeddings), hoặc học liên bang dưới điều kiện statistical heterogeneity rất cao để hiệu chuẩn các bộ trích xuất đặc trưng (feature extractors) và bộ phân loại (classifiers).

---

### 5. Những đóng góp chính (Contributions)
Nghiên cứu mang lại 4 đóng góp chính cốt lõi:

1.  **Formal hóa toán học bài toán:** Đây là công trình đầu tiên chính thức hóa bài toán personalized horizontal FL trên các không gian đặc trưng không đồng nhất của client. Khác với các phương pháp trước đây, FLIC cho phép mỗi client tận dụng dữ liệu từ các client khác ngay cả khi chúng không sở hữu cùng cách biểu diễn dữ liệu thô đầu vào.
2.  **Khung căn chỉnh phân phối (Distribution alignment framework):** Đề xuất một framework căn chỉnh phân phối, thuật toán học các hàm nhúng cục bộ $\{\phi_i\}$ cùng một phân phối neo ẩn $\{\mu_c\}$ học liên bang. Các thành phần này được tích hợp chặt chẽ vào một thuật toán personalized FL hoàn chỉnh.
3.  **Hỗ trợ thuật toán và toán lý thuyết vững chắc:** Cung cấp cả phân tích thuật toán và cơ sở toán học lý thuyết vững chắc. Trong một kịch bản học đơn giản hóa, nhóm tác giả chứng minh toán học rằng FLIC có khả năng khôi phục chính xác không gian con tiềm ẩn thực tế (latent subspace) với tốc độ hội tụ tuyến tính phi tiệm cận.
4.  **Thực nghiệm toàn diện:** Các thí nghiệm trên dữ liệu mô phỏng (toy datasets) và dữ liệu thực tế chứng minh FLIC đạt hiệu năng vượt trội hơn hẳn các phương pháp đối sánh FL trong các thiết lập dị dạng không gian đặc trưng.

---

### 6. Quy ước và ký hiệu toán học chính
*   $\|\cdot\|$: Chuẩn Euclid trên không gian vector $\mathbb{R}^d$.
*   $|S|$: Số phần tử của tập hợp $S$.
*   $\mathbb{N}^* = \mathbb{N} \setminus \{0\}$: Tập hợp các số nguyên dương.
*   Với $n \in \mathbb{N}^*$, ký hiệu $[n] = \{1, 2, \dots, n\}$.
*   $\mathcal{N}(m, \Sigma)$: Phân phối Gauss (Gaussian distribution) với vector trung bình $m$ và ma trận hiệp phương sai $\Sigma$.
*   $X \sim \nu$: Biến ngẫu nhiên $X$ được lấy mẫu từ phân phối xác suất $\nu$.
*   **Khoảng cách Wasserstein bậc 2** giữa hai độ đo xác suất $\mu$ và $\nu$ trên không gian $\mathbb{R}^d$ có mô-men bậc hai hữu hạn được định nghĩa là:
    $$W_2(\mu, \nu) = \left( \inf_{\zeta \in \mathcal{T}(\mu, \nu)} \int_{\mathbb{R}^d \times \mathbb{R}^d} \|\theta - \theta'\|^2 \, d\zeta(\theta, \theta') \right)^{1/2}$$
    Trong đó, $\mathcal{T}(\mu, \nu)$ đại diện cho tập hợp tất cả các transference plans giữa $\mu$ và $\nu$.

---

### 7. Hiểu phần Giới thiệu (Introduction) qua ví dụ trực quan

Hãy tưởng tượng hệ thống gồm 3 client học liên bang phân loại chữ số:
*   **Client 1:** Sử dụng tập dữ liệu MNIST với kích thước ảnh $28 \times 28 \rightarrow x_1 \in \mathbb{R}^{784}$.
*   **Client 2:** Sử dụng tập dữ liệu USPS với kích thước ảnh $16 \times 16 \rightarrow x_2 \in \mathbb{R}^{256}$.
*   **Client 3:** Sử dụng tập dữ liệu SVHN với kích thước ảnh $32 \times 32 \rightarrow x_3 \in \mathbb{R}^{1024}$.

Nếu sử dụng thuật toán học liên bang thông thường, server trung tâm không thể tổng hợp mô hình trực tiếp qua phép cộng trung bình tham số $\text{model}_{\text{global}} = \sum w_i \text{model}_i$ vì các không gian đặc trưng đầu vào hoàn toàn không khớp nhau về cấu trúc kích thước.

#### Cách giải quyết tuần tự của FLIC:
1.  **Chuyển đổi không gian thô về không gian ẩn:**
    $$X_1 \xrightarrow{\phi_1} \Phi \quad ; \quad X_2 \xrightarrow{\phi_2} \Phi \quad ; \quad X_3 \xrightarrow{\phi_3} \Phi$$
    Đưa dữ liệu đầu vào có kích thước khác nhau ($784, 256, 1024$) về một không gian ẩn chung $\Phi$ có số chiều cố định bằng các hàm nhúng học được.
2.  **Căn chỉnh phân phối ngữ nghĩa:**
    $$\phi_i(X_i | Y=c) \longrightarrow \text{anchor distribution } \mu_c$$
    Sử dụng khoảng cách Wasserstein bậc 2 để ép dữ liệu nhúng của các client có cùng nhãn lớp tiến sát tới phân phối neo tham chiếu chung, đảm bảo sự đồng nhất ngữ nghĩa.
3.  **Học liên bang cá nhân hóa (Personalised FL):**
    Thực hiện huấn luyện mô hình cá nhân hóa trên không gian ẩn chung $\Phi$ đã được căn chỉnh hoàn hảo.

#### Tóm gọn xương sống cấu trúc của FLIC:
$$\text{Không gian đặc trưng dị dạng (Dữ liệu thô)} \rightarrow \text{Hàm nhúng cục bộ} \rightarrow \text{Không gian ẩn chung} \rightarrow \text{Căn chỉnh phân phối bằng Wasserstein} \rightarrow \text{Học liên bang cá nhân hóa}$$
