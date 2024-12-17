## Abstract
Trong bài báo cáo đã trình bày một thuật toán học sâu để ước lượng kênh trong các hệ thống thông tin. Chúng tôi coi đáp ứng thời gian–tần số của một kênh thông tin liên lạc biến đổi nhanh như một hình ảnh 2D. Mục tiêu là tìm các giá trị chưa biết của đáp ứng kênh bằng cách sử dụng một số giá trị đã biết tại các vị trí pilot. Để đạt được điều này, một quy trình tổng quát sử dụng các kỹ thuật deep image processing, image super-resolution (SR) và image restoration (IR). Quy trình này xem các giá trị pilot như một hình ảnh có độ phân giải thấp và sử dụng một mạng SR kết hợp với một mạng IR khử nhiễu để ước lượng kênh. Lỗi ước lượng cho thấy thuật toán được trình bày có thể so sánh với phương pháp MMSE (Minimum Mean Square Error ) khi có đầy đủ thông tin thống kê kênh, tốt hơn so với phương pháp xấp xỉ MMSE tuyến tính. Các kết quả xác nhận rằng quy trình này có thể được sử dụng hiệu quả trong việc ước lượng kênh.

## I. Introduction

Phương pháp điều chế ghép kênh phân chia tần số trực giao (OFDM) đã được sử dụng rộng rãi trong các hệ thống thông tin liên lạc để giải quyết hiện tượng fading chọn lọc tần số trong các kênh không dây. Trong một kênh thông tin liên lạc, tín hiệu nhận được thường bị méo do đặc tính của kênh. Để khôi phục các ký hiệu được truyền, ảnh hưởng của kênh phải được ước lượng và bù trừ tại bộ thu.

Thông thường, bộ thu ước lượng kênh bằng cách sử dụng một số ký hiệu được gọi là "pilot" (kênh tham chiếu), mà vị trí và giá trị của chúng trong miền thời gian-tần số được cả máy phát và máy thu biết trước. Tùy thuộc vào cách sắp xếp các ký hiệu pilot, ba cấu trúc khác nhau có thể được xem xét: loại khối (block-type), loại lược (comb-type), và loại lưới (lattice-type).

- Trong sắp xếp loại khối, các pilot được truyền định kỳ ở đầu mỗi khối OFDM tại tất cả các sóng mang con.
- Trong sắp xếp loại lược, các pilot chỉ xuất hiện ở một số sóng mang con trong một số ký hiệu OFDM.
- Trong sắp xếp loại lưới, các pilot được chèn dọc theo cả trục thời gian và tần số theo một chòm sao hình thoi.

Các phương pháp ước lượng kênh dựa trên pilot truyền thống như Least Square (LS) và Minimum Mean Square Error (MMSE) sử dụng các giá trị pilot trong miền thời gian-tần số để tìm các giá trị chưa biết của đáp ứng kênh. Các thuật toán này đã được tối ưu hóa trong nhiều điều kiện khác nhau. Ngược lại với phương pháp LS, không yêu cầu thông tin về thống kê của kênh, phương pháp MMSE đạt hiệu suất tốt hơn bằng cách sử dụng thống kê của kênh và phương sai nhiễu.

Gần đây, học sâu (Deep Learning - DL) đã thu hút nhiều sự chú ý trong các hệ thống thông tin liên lạc. Trong các hệ thống thông tin liên lạc dựa trên DL, một số phương pháp đã được đề xuất để cải thiện hiệu suất của các thuật toán truyền thống, bao gồm modulation recognition, signal detection , channel equalization, channel state information (CSI), và channel estimation.

Hệ thống thông tin liên lạc được xem như một "hộp đen" và một kiến trúc DL từ đầu đến cuối được sử dụng để truyền/nhận tín hiệu. Bộ mã hóa, giải mã, ước lượng kênh và tất cả các hàm chức năng khác của một kênh truyền được đặt trong DL-block. Mặc dù vậy, phương pháp này không thể tìm đáp ứng thời gian-tần số của kênh một cách rõ ràng, do đó không hiệu quả cho các ứng dụng cần có đầy đủ đáp ứng kênh. 

Vì lý do này, trong bài viết này, chúng tôi đề xuất một khung DL để ước lượng kênh trong các hệ thống OFDM. Trong phương pháp này, miền thời gian-tần số của đáp ứng kênh được mô hình hóa như một hình ảnh 2D, chỉ biết tại các vị trí pilot. Channel grid cùng với một số pilot, được xem như một hình ảnh độ phân giải thấp (LR) và kênh ước lượng là một hình ảnh độ phân giải cao (HR). Chúng tôi trình bày một cách tiếp cận hai giai đoạn để ước lượng lưới kênh:

1. Sử dụng thuật toán siêu phân giải hình ảnh (SR) để cải thiện độ phân giải của dữ liệu đầu vào LR.
2. Sau đó, sử dụng phương pháp phục hồi hình ảnh (IR) để loại bỏ nhiễu.

Hai thuật toán CNN là SRCNN và DnCNN, đã được sử dụng cho các mạng SR và IR tương ứng.
## II. Background
### A. Ước lượng kênh

Trong hệ thống OFDM, mối quan hệ giữa tín hiệu đầu vào và đầu ra tại khe thời gian thứ $k$ và sóng mang con thứ $i$ được biểu diễn như sau:
$$Y_{i,k} = H_{i,k}X_{i,k} + Z_{i,k}.$$
Với một khung con OFDM có kích thước $N_S \times N_D$, chỉ số khe thời gian $k$ nằm trong khoảng $[0, N_D - 1]$, và phạm vi của chỉ số sóng mang con $i$ là $[0, N_S - 1]$. Trong phương trình (1), $Y_{i,k}$, $X_{i,k}$  và $Z_{i,k}$ lần lượt là tín hiệu nhận, ký hiệu OFDM được truyền và nhiễu Gaussian trắng. $H_{i,k}$ là phần tử $(i, k)$ của ma trận $H \in \mathbb{C}^{N_S \times N_D}$, biểu diễn đáp ứng thời gian–tần số của kênh cho tất cả các sóng mang con và khe thời gian.

Để ước lượng kênh, đặc biệt trong các kênh có fading, đáp ứng miền thời gian được biểu diễn dưới dạng $H = \{h_1, h_2, \dots, h_{N_D}\}$, trong đó mỗi $h_k$ là đáp ứng kênh trong miền tần số tại khe thời gian thứ $k$.

Phương pháp Least Square (LS) ước lượng kênh tại các vị trí pilot. Nếu ma trận kênh LS ước lượng tại các vị trí pilot được biểu diễn dưới dạng một ma trận đường chéo $H_p^{LS} \in \mathbb{C}^{N_P \times N_P}$, thì $H_p^{LS}$ có thể được ước lượng bằng:
$$\hat{H}_p^{LS} = \underset{H_p}{\text{arg min}} \|y_p - H_p x_p\|_2^2$$
trong đó $\hat{H}_p^{LS} \in \mathbb{C}^{N_P \times N_P}$ là ma trận của đáp ứng kênh truyền ước lượng. $x_p$ chứa các giá trị pilot đã biết, và $y_p$ là các tín hiệu quan sát tương ứng. 

Một lựa chọn tốt hơn LS là phương pháp MMSE, được tính bằng cách nhân các giá trị LS tại vị trí các ký hiệu pilot với ma trận  $A_{MMSE} \in \mathbb{C}^{N_L \times N_P}$:
$$\hat{h}_d^{MMSE} = A_{MMSE} \times \hat{h}_p^{LS}$$

trong đó $\hat{h}_d^{MMSE} \in \mathbb{C}^{N_L \times 1} (N_L = N_S \times N_D)$ là vector ước lượng MMSE của đáp ứng kênh $H$ trong khung con $d$. Để xác định ma trận lọc, sai số trung bình bình phương (MSE) được tính như sau:
$$\varepsilon = \mathbb{E}\{\|\hat{h}_d - A_{MMSE} \hat{h}_p^{LS}\|_2^2\}$$

Việc tối thiểu hóa $\varepsilon$ dẫn đến:
$$A_{MMSE} = R_{h_dh_p} (R_{h_ph_p} + \sigma_n^2 (xx^H)^{-1})^{-1}$$

trong đó ma trận $R_{h_dh_p} = \mathbb{E}\{h_d h_p^H\}$ biểu diễn ma trận tương quan kênh giữa khung con mong muốn và các ký hiệu pilot, và $R_{h_ph_p} = \mathbb{E}\{h_p h_p^H\}$ là ma trận tương quan tại các vị trí pilot. Có thể thấy, phương pháp MMSE chỉ hữu ích khi ma trận tương quan kênh $R$ được biết đầy đủ.
### B. Siêu phân giải và Phục hồi hình ảnh

Khi xử lý một hình ảnh có độ phân giải thấp (LR) và chứa nhiễu, nhiều kỹ thuật đã được đề xuất để tái tạo một hình ảnh có độ phân giải cao hơn (HR) và ít nhiễu hơn. Siêu phân giải hình ảnh (SR) là một nhóm các kỹ thuật được sử dụng để nâng cao độ phân giải của hình ảnh. Các thuật toán dựa trên học sâu, đặc biệt là mạng nơ-ron tích chập (CNN), đã đạt được hiệu suất cao trong việc khôi phục hình ảnh HR từ dữ liệu đầu vào LR.

Gần đây, mạng SRCNN (Super-resolution convolutional neural network) được đề xuất để ánh xạ giữa hình ảnh LR và HR theo cách end-to-end. Ngoài ra, các thuật toán phục hồi hình ảnh (IR) cũng có thể được áp dụng để loại bỏ/giảm nhiễu trong hình ảnh. Một ví dụ là mạng DnCNN (Denoising CNN), sử dụng học phần dư và chuẩn hóa để tăng tốc quá trình huấn luyện.

## III. Channel Net

### A. Channel Image

Trong bài báo cáo này tập trung vào một liên kết giữa một cặp ăng-ten truyền (Tx) và ăng-ten nhận (Rx), tức là chúng tôi xét liên kết truyền thông single-input, single-output (SISO). Đối với liên kết này, ma trận đáp ứng thời gian–tần số của kênh $H$ (kích thước $N_S \times N_D$) giữa một máy phát và một máy thu. Ma trận $H$ này có các giá trị phức có thể được biểu diễn thành hai hình ảnh 2D (một hình ảnh 2D cho phần thực và một hình ảnh 2D cho phần ảo).

Ví dụ về hình ảnh 2D được chuẩn hóa cho phần thực và phần ảo của một lưới thời gian–tần số kênh mẫu với $N_D = 14$ khe thời gian và $N_S = 72$ sóng mang con (theo tiêu chuẩn LTE) được hiển thị trong Hình 1.

![[Pasted image 20241215093558.png]](Pasted%20image%2020241215093558.png)

### B. Cấu trúc mạng

Tổng quan về quy trình DL để ước lượng kênh, gọi là Channel Net, được minh họa trong Hình 2. Mục tiêu là ước lượng toàn bộ thời gian–tần số của kênh sử dụng các pilot được truyền. Tương tự như tiêu chuẩn LTE, cách sắp xếp pilot dạng lưới (lattice-type) đã được sử dụng để truyền pilot.

Giá trị ước lượng của kênh tại các vị trí pilot $\hat{h}_p^{LS}$ (có thể nhiễu) được coi là phiên bản có độ phân giải thấp (LR) và chứa nhiễu của hình ảnh kênh. Để thu được hình ảnh kênh hoàn chỉnh, quy trình huấn luyện hai giai đoạn:

* Trong giai đoạn đầu, một mạng SR được triển khai, lấy $\hat{h}_p^{LS}$ dưới dạng hình ảnh đầu vào có độ phân giải thấp (được vector hóa, lần lượt cho phần thực và phần ảo) và ước lượng các giá trị chưa biết của đáp ứng kênh $H$.
* Giai đoạn hai là một mạng IR khử nhiễu được ghép nối với mạng SR để loại bỏ các hiệu ứng nhiễu.

![[Pasted image 20241215101126.png]](Pasted%20image%2020241215101126.png)

Trong các mạng SR và IR, chúng tôi lần lượt sử dụng SRCNN [10] và DnCNN [11]. SRCNN trước tiên sử dụng một sơ đồ nội suy để tìm các giá trị xấp xỉ của hình ảnh độ phân giải cao (kênh), sau đó cải thiện độ phân giải bằng cách sử dụng một mạng tích chập ba lớp.

- Lớp tích chập đầu tiên sử dụng 64 bộ lọc kích thước $9 \times 9$.
- Lớp thứ hai sử dụng 32 bộ lọc kích thước $1 \times 1$, cả hai đều kết hợp hàm kích hoạt ReLU.
- Lớp cuối cùng sử dụng một bộ lọc kích thước $5 \times 5$ để tái tạo hình ảnh.

DnCNN là một mạng học phần dư với 20 lớp tích chập.

- Lớp đầu tiên sử dụng 64 bộ lọc kích thước $3 \times 3 \times 1$ với ReLU.
- 18 lớp tiếp theo sử dụng 64 bộ lọc kích thước $3 \times 3 \times 64$, kết hợp batch normalization và ReLU.
- Lớp cuối cùng sử dụng một bộ lọc kích thước $3 \times 3 \times 64$ để tái tạo đầu ra.

### C. Huấn luyện

Gọi tập hợp tất cả các tham số mạng là $\Theta = \{\Theta_S, \Theta_R\},$ trong đó $\Theta_S$ và $\Theta_R$ lần lượt là các tham số của mạng SR và IR.

Đầu vào của Channel Net là vector giá trị pilot $\hat{h}_p^{LS},$ và đầu ra là ma trận kênh ước lượng $\hat{H}$ được định nghĩa như sau:
$$\hat{H} = f(\Theta; \hat{h}_p^{LS}) = f_R(f_S(\Theta_S; \hat{h}_p^{LS}); \Theta_R)$$
với $f_S$ và $f_R$ lần lượt là các hàm SR và IR.

Hàm loss của mạng là sai số trung bình bình phương (MSE) giữa kênh ước lượng và kênh thực, được tính như sau:
$$C = \frac{1}{||T||} \sum_{h_p \in T} \|f(\Theta; \hat{h}_p^{LS}) - H\|_2^2$$
trong đó $T$ là tập dữ liệu huấn luyện và $H$ là kênh lý tưởng.

Để đơn giản hóa quá trình huấn luyện, chúng tôi áp dụng thuật toán huấn luyện hai giai đoạn:

1. **Giai đoạn đầu**: Tối ưu hóa mạng SR bằng cách tối thiểu hóa hàm mất mát $C_1$:
$$C_1 = \frac{1}{||T||} \sum_{h_p \in T} \|Z - H\|_2^2$$
trong đó $Z = f_S(\Theta_S; \hat{h}_p^{LS})$ là đầu ra của mạng SR.

2. **Giai đoạn hai**: Giữ cố định trọng số của mạng SR và tối ưu hóa mạng IR bằng cách xác định $\hat{H} = f_R(Z; \Theta_R)$ và tối thiểu hóa hàm mất mát $C_2$:
$$C_2 = \frac{1}{||T||} \sum_{h_p \in T} \|\hat{H} - H\|_2^2$$

Lưu ý rằng, trọng số tối ưu của mạng phụ thuộc vào giá trị SNR. Để có giải pháp toàn diện, mạng cần được huấn luyện lại cho mỗi giá trị SNR. 

## IV. Kết quả mô phỏng
Trong phần này, thực hiện huấn luyện mạng và đánh giá sai số trung bình bình phương (MSE) trên một dải giá trị SNR, đồng thời so sánh kết quả với các thuật toán cơ sở phổ biến.

Chúng tôi xem xét một hệ thống truyền thông có một ăng-ten ở phía phát và một ăng-ten ở phía nhận. Trong mô phỏng, chúng tôi sử dụng trình giả lập LTE phổ biến của Đại học Vienna (Vienna LTE-A Simulator) để mô hình hóa kênh và truyền pilot. Việc triển khai mô hình đề xuất được thực hiện bằng Keras và Tensorflow với GPU làm backend.

Đối với mạng SR và IR, tốc độ học được đặt là 0.001, kích thước batch là 128 và số lượng tối đa 500 lần lặp. Các tập huấn luyện, kiểm thử và xác thực lần lượt gồm 32.000, 4.000 và 4.000 mẫu kênh.

Trong các mô phỏng, mỗi khung gồm 14 khe thời gian với 72 sóng mang con (theo chuẩn LTE). Hai mô hình kênh không dây được xem xét:

1. **Vehicular-A (VehA)**: Kênh với độ trễ ngắn.
2. **SUI5**: Mô hình kênh với độ trễ dài, băng thông 1.6 MHz, tần số sóng mang là 2.1 GHz, và tốc độ thiết bị người dùng (UE) là 50 km/h.

### Hiệu suất ước lượng kênh

Để đánh giá hiệu suất, chúng tôi so sánh độ chính xác của phương pháp được đề xuất với ba thuật toán tiên tiến:

- **MMSE lý tưởng**: Sử dụng ma trận tương quan kênh chính xác (không lỗi).
- **MMSE ước lượng**: Sử dụng ước lượng ma trận tương quan kênh.
- **ALMMSE lý tưởng**: Phiên bản xấp xỉ tuyến tính của MMSE lý tưởng.

MSE giữa kênh ước lượng và kênh thực được sử dụng làm chỉ số đánh giá hiệu suất.

#### Kết quả cho mô hình VehA

Hình 3 trình bày kết quả MSE của mô hình VehA.

- **MMSE lý tưởng** có hiệu suất tốt nhất và cung cấp ngưỡng dưới của MSE. Tuy nhiên, phương pháp này yêu cầu biết đầy đủ ma trận tương quan kênh, điều này không thực tế.
- Với các giá trị SNR thấp, mạng ChannelNet được huấn luyện tại SNR = 12 dB (gọi là **mạng deep low-SNR**) có hiệu suất tương đương với MMSE lý tưởng và vượt trội so với ALMMSE lý tưởng và MMSE ước lượng.
- Với SNR cao hơn một ngưỡng nhất định, mạng ChannelNet được huấn luyện tại SNR = 22 dB (**mạng deep high-SNR**) bắt đầu hoạt động tốt hơn mạng deep low-SNR.

Do đó, chúng tôi chia phạm vi SNR thành hai vùng. Với SNR thấp, mạng deep low-SNR được sử dụng; khi vượt quá ngưỡng SNR, mạng deep high-SNR được sử dụng. Tuy nhiên, khi SNR lớn hơn 23 dB, hiệu suất của mạng deep high-SNR giảm dần, điều này cho thấy cần huấn luyện thêm một mạng mới cho các giá trị SNR cao hơn. Trong phạm vi SNR dưới 20 dB, hai mạng này là đủ.

#### Kết quả cho mô hình SUI5

Hình 4 trình bày kết quả MSE cho mô hình kênh SUI5.

- Do tính phức tạp cao hơn của kênh, hiệu suất của tất cả các phương pháp đều giảm so với mô hình VehA.
- Tại SNR trên 5 dB, các phương pháp như ALMMSE và MMSE ước lượng bị suy giảm đáng kể, trong khi mạng DL đề xuất vẫn có khả năng khám phá các thống kê tiềm ẩn và đạt được MSE chấp nhận được.
- Như kỳ vọng, MMSE lý tưởng vẫn là phương pháp tốt nhất, nhưng không khả thi trong các ứng dụng thực tế vì cần đầy đủ thông tin thống kê kênh.

#### Kết quả với số lượng pilot khác nhau

Để đánh giá thêm, Hình 5 minh họa kết quả MSE với mô hình VehA ở mức SNR = 20 dB khi số lượng pilot thay đổi.

- Mạng ChannelNet, được huấn luyện tại mức SNR cụ thể, vượt trội so với phương pháp MMSE ước lượng và ALMMSE lý tưởng, đồng thời có hiệu suất tương đương MMSE lý tưởng.

---

### Kết luận

Các kết quả trên cho thấy phương pháp ChannelNet hoạt động tốt trong cả các kênh đơn giản (VehA) và phức tạp (SUI5), đặc biệt ở các giá trị SNR khác nhau. Phương pháp này có thể thay thế hiệu quả cho MMSE lý tưởng trong thực tế, nơi thông tin thống kê đầy đủ của kênh không thể có được.