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