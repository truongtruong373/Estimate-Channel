## Base
Phương trình tại băng tần cơ sở
$$y = hx + n$$
h sẽ tương ứng với tín hiệu kênh truyền (channel), x là dữ liệu truyền đi (signal) và n là noise, phần lớn là nhiễu trắng (AWGN) 
Thông thường thì n sẽ được ước lượng dựa trên SNR, còn x sẽ được ước lượng dựa trên cả 2 đầu máy phát và máy thu (Pilot, Known Sequense, Reference Sequense)
Phần tử h chính là phần tử mà chúng ta cần ước lượng bằng cách gửi một số training data mà bên nhận đã biết 
Phân tích 1 chút :
$$ \hat{h} \ =\ \frac{y}{x} \ =\ h\ +\ \frac{n}{x}$$
Estimation error = $\hat {h} - h$ = n/x
## MMSE (Minimum Mean Squares Error) là gì ?
MMSE là phương pháp hoạt động dựa trên nguyên tắc ước lượng thông qua việc cực tiểu hóa MSE hoặc SER của kênh. Ước lượng kênh MMSE được sử dụng rộng rãi trong các hệ
thống truyền thông vô tuyến, chẳng hạn như hệ thống truyền dẫn OFDM. Việc ra đời của MMSE đã cải thiện hiệu quả truyền thông, tăng cường khả năng chống nhiễu và giảm thiểu
lỗi truyền.
- Ước tính sai số bình phương trung bình tối thiểu (MMSE)  là một phương pháp ước tính giúp giảm thiểu sai số bình phương tối thiểu (MSE), là thước đo để đánh giá xem estimation có tốt hay không, tức là chất lượng của việc ước tính
$$ MSE\ =E[(\hat h-h)^{2}]=\ ||\ \hat{h_{1}} \ -\ h\ ||^{2} \ =\ \frac{|n_{1} |^{2}}{|x_{1} |^{2}} \ =\ \frac{\sigma ^{2}}{P} \ =\ \frac{1}{SNR}$$
	 Trong đó  P là Công suất năng lượng chuỗi Pilot P 
		$\sigma ^{2}$ là công suất nhiễu
- Có thể thấy, khi SNR $\uparrow$  thì MSE $\downarrow$ và $\hat{h_{1}} \uparrow$  tức là chất lượng ước lượng kênh càng tốt 
- MMSE là 1 Bayasian Estimate, là ước lượng giảm thiểu giá trị kỳ vòng sau của loss function.
	- h là 1 biến ngẫu nhiên với hàm mật độ xác suất P(h)
	- h và x tồn tại trong pdf P(h,x) với h là 1 unknown variable còn x là data đã có sẵn.
- MMSE estimator của h là 1 hàm $\hat h=g(x)$ , là hàm tượng trưng cho việc minizies the MSE.
- Property of MMSE Estimation
	- The MMSE is unbiased
	$E[\hat h-h]=E[E[h|x]-h] =E[E[h|x]]-E[h]=E[h]-E[h]=0$
	- It has the lowest MSE among all estimators
## LS(Least Square)
Ước tính kênh Least Squares (LS) là một phương pháp ước tính các tham số của một mô hình kênh dựa trên nguyên tắc cực tiểu giá trị bình phương của lỗi. Phương pháp này được sử dụng rộng rãi trong lĩnh vực truyền thông và xử lý tín hiệu. Kỹ thuật này sử dụng các bit truyền đã biết để ước lượng CIR (Phản hồi xung kênh). Ưu điểm của phương pháp ước tính
kênh LS là tính toán đơn giản và dễ hiểu. Tuy nhiên, nó có thể bị ảnh hưởng bởi nhiễu và sai số trong dữ liệu đầu vào. Do đó, các phương pháp ước tính kênh LS thường được kết hợp với các kỹ thuật khác như regularized LS, recursive LS, hoặc phương pháp ước tính tối ưu hơn để cải thiện độ chính xác và ổn định của ước tính kênh

## SRCNN (Super Resolution Convolutional Neural Network)

- Cho ảnh X là một ảnh high-resolution (độ phân giải cao) và Y là một phiên bản low-resolution (độ phân giải thấp) của X. Gọi hàm được dùng để upscale ảnh input là F. Bài toán upscale ảnh sẽ hướng đến việc tìm F sao cho F(Y)≈X hay ảnh upscale từ Y gần giống X nhất có thể.
- SRCNN là mạng CNN sẽ được dùng để học end-to-end mapping giữa ảnh low res và high res.
- Cấu trúc mạng SRCNN rất đơn giản. Nó chỉ có 3 layer tất cả: patch extraction and representation, non-linear mapping và reconstruction.
 ![[Pasted image 20241224100228.png]]
### Patch extraction and representation
- Layer đầu tiên này sẽ có nhiệm vụ lấy các patch (mảnh) overlap nhau ở trên ảnh input bằng cách trượt một kernel có kích thước cố định ở trên ảnh. Sau đó, nó biểu diễn feature map của từng patch dưới dạng một vector nhiều chiều. Số lượng feature map sẽ tương ứng với số chiều của vector (còn 1 vài khái niệm nhưng chắc ko nên đi sâu hơn) 
- Output của layer này là một vector n1​ chiều tương ứng với n1​ feature map.
### Non-linear mapping
- Sau khi lấy được vector feature của ảnh low-resolution, ta sẽ cho nó qua layer tích chập thứ hai. Layer này có nhiệm vụ lấy map (ánh xạ) vector n1​ chiều ở layer trước tới một vector n2​ chiều
- Output của layer này là một vector n2 chiều ứng với n2​ feature map. Mỗi một feature map trong vector này sẽ là biểu diễn của một high-resolution patch được dùng để khôi phục ảnh.
- Theo tác giả bài báo, chúng ta có thể thêm nhiều layer hơn ở giữa để tăng tính phi tuyến tính (non-linearity). Tuy nhiên, việc tăng số lượng layer lên sẽ làm mô hình trở nên phức tạp hơn và tốn nhiều thời gian train để mô hình hội tụ. Đồng thời, kết quả trong bài báo cho thấy việc tăng số lượng layer (lớn hơn tổng 3 layer của cả model) cũng không làm tăng đáng kể chất lượng của output nên trong phần cài đặt, bài viết này sẽ chỉ cài đặt một layer ở giữa.
### Reconstruction
- Layer cuối này sẽ được dùng để khôi phục ảnh high-resolution từ vector n2​ chiều ở layer trước.
- Kết quả của layer này sẽ là ảnh đã được upscale, có kích thước bằng kích thước ảnh input (do ảnh kích thước đã được upscale từ đầu).

## DnCNN (Denoising Convolutional Neural Network)
- **Mạng có thể xử lý khử nhiễu Gauss với mức độ nhiễu không xác định** (tức là khử nhiễu Gauss mù).
- **Một mạng duy nhất được đào tạo có thể xử lý 3 tác vụ** : Khử nhiễu hình ảnh, Siêu phân giải hình ảnh đơn và Gỡ chặn JPEG.
- **Học tập dư thừa, có nguồn gốc từ** [**ResNet**](https://towardsdatascience.com/review-resnet-winner-of-ilsvrc-2015-image-classification-localization-detection-e39402bfa5d8) **và batch normalization** , **có nguồn gốc từ** [**Inception-v2**](https://medium.com/@sh.tsang/review-batch-normalization-inception-v2-bn-inception-the-2nd-to-surpass-human-level-18e2d0f56651) **, được sử dụng.** Với chiến lược học tập dư thừa, DnCNN ngầm loại bỏ hình ảnh sạch tiềm ẩn trong các lớp ẩn.
![[Pasted image 20241224101445.png]]
- Kích thước của bộ lọc tích chập (conv) được đặt là 3x3
- - **(i) Conv+ReLU** : Đối với lớp đầu tiên, 64 bộ lọc có kích thước 3×3× _c_ được sử dụng để tạo 64 bản đồ đặc điểm. _c_ = 1 đối với ảnh xám và _c_ = 3 đối với ảnh màu.
- **(ii) Conv+BN+ReLU** : đối với các lớp 2 đến (D-1), 64 bộ lọc có kích thước 3×3×64 được sử dụng và chuẩn hóa theo lô được thêm vào giữa phép tích chập và ReLU.
- **(iii) Conv** : đối với lớp cuối cùng, bộ lọc _c_ có kích thước 3×3×64 được sử dụng để tái tạo đầu ra. 
-  Bằng cách kết hợp tích chập với ReLU, DnCNN **có thể dần dần tách cấu trúc hình ảnh khỏi quan sát nhiễu** thông qua các lớp ẩn.

## OFDM 

Đọc trong này tiếng việt cho dễ hiểu: https://luanvan.net.vn/luan-van/do-an-tim-hieu-ve-ky-thuat-uoc-luong-kenh-truyen-trong-he-thong-ofdm-22251/

## LTE (Long term Evolution)
- là một tiêu chuẩn truyền thông không dây được phát triển bởi 3GPP để cung cấp tốc độ cao hơn và hiệu suất tốt hơn cho các mạng di động so với 3G
- Đặc điểm chính
	- Tốc độ cao : downlink 300Mbps , uplink 75Mbps phù hợp với video streaming, chơi game trực tuyến
	- Độ trễ thấp :
	- Kiến trúc mạng đơn giản hóa 

## Pilot
- Pilot là các tín hiệu đã biết trước được chèn vào tín hiệu truyền đi,  chúng đóng vai trò tham chiếu để giúp máy thu ước lượng các tham số của kênh truyền như độ trễ, độ lợi kênh, hiệu ứng fading
- Trong mạng lưới đáp ứng thời gian-tần số, pilot được chèn vào một mạng lưới (grid) trong cả thời gian và tần số, đảm bảo bao phủ toàn bộ khung tín hiệu
- Được sử dụng để hỗ trợ máy thu ước lượng đáp ứng kênh (channel response) nhằm khắc phục các vấn đề về nhiễu , fading và méo tín hiệu
