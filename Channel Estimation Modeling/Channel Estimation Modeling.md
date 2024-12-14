# 1. Giới thiệu
Các kênh vô tuyến trong hệ thống thông tin di động thường là các kênh fading đa đường, thường gây ra hiện tượng nhiễu ISI. Để loại bỏ  nhiễu ISI khỏi tín hiệu, nhiều loại bộ cân bằng có thể được sử dụng. Các thuật toán phát hiện như MLSE hoặc MAP cung cấp hiệu suất máy thu tốt, nhưng vẫn yêu cầu tính toán không quá phức tạp. Do đó, các thuật toán này hiện đang khá phổ biến.

Tuy nhiên, các bộ phát hiện này đòi hỏi thông tin về đáp ứng xung kênh (CIR), thông tin này có thể được cung cấp bởi một bộ ước lượng kênh riêng biệt. Thông thường, ước lượng kênh dựa trên chuỗi bit đã biết, các bit này là duy nhất cho một máy phát nhất định và được lặp lại trong mỗi khung truyền. Do đó, bộ ước lượng kênh có thể ước tính CIR cho từng khung riêng biệt bằng cách sử dụng các bit truyền đã biết và các mẫu nhận tương ứng.

Dưới đây trước tiên cung cấp một số thông tin nền tảng về ước lượng kênh. Sau đó giới thiệu các kỹ thuật ước lượng kênh theo phương pháp bình phương tối thiểu (Least-Squares - LS). Một số nhận xét về mô phỏng hệ thống ước lượng kênh. Sau đó là phương pháp ước lượng kênh lặp lại (iterative).. Ý tưởng là sử dụng các ký hiệu được giải mã để huấn luyện bộ ước lượng kênh, từ đó cải thiện chất lượng ước lượng. 
# 2. Nền tảng về ước lượng kênh
Hình 1 minh họa bố cục mô phỏng chung cho một hệ thống di động dựa trên TDMA, trong đó sử dụng ước lượng kênh và phát hiện tín hiệu trong việc cân bằng tín hiệu. Nguồn tín hiệu số thường được bảo vệ bằng cách mã hóa kênh và xen kẽ (interleaved) chống lại hiện tượng fading, sau đó tín hiệu được điều chế và truyền qua kênh fading đa đường. Nhiễu cộng được thêm vào tín hiệu truyền đi và được thu ở bên nhận.

Do kênh đa đường (multipath channel), tín hiệu nhận được bị nhiễu liên ký tự (ISI). Vì vậy, một bộ phát hiện tín hiệu (detector) (như MLSE hoặc MAP) cần biết các đặc điểm của đáp ứng xung kênh (CIR) để đảm bảo cân bằng thành công (loại bỏ ISI). Sau khi phát hiện, tín hiệu được giải xen kẽ (deinterleaved) và giải mã kênh để có được bản tin gốc.
![[Pasted image 20241213232858.png]](Pasted%20image%2020241213232858.png)

Thông thường, CIR được ước tính dựa trên tín các bit huấn luyện đã biết, được truyền trong mỗi khung truyền như minh họa ở Hình 2 cho hệ thống GSM hiện tại. Bộ thu có thể sử dụng các bit huấn luyện đã biết và các mẫu nhận tương ứng để ước tính CIR, thường cho từng khung riêng biệt. Có một số cách tiếp cận khác nhau để ước lượng kênh, chẳng hạn như phương pháp Bình phương tối thiểu (LS) hoặc phương pháp Sai số Bình phương Trung bình Tuyến tính Tối thiểu (LMMSE).

![[Pasted image 20241213234556.png]](Pasted%20image%2020241213234556.png)

# 3. Ước lượng kênh bằng phương pháp bình phương tối thiểu (LS)
## a. Bộ ước lượng kênh cho tín hiện đơn giản
Xét một hệ thống truyền thông chỉ bị nhiễu bởi nhiễu cộng như trong Hình 3. Tín hiệu số $a$ được truyền qua kênh fading đa đường $h_L$. Nhiễu nhiệt được sinh ra tại bộ thu và được mô hình hóa dưới dạng nhiễu Gaussian trắng $n$. Vấn đề giải điều chế ở đây là phát hiện các bit truyền $a$ từ tín hiệu nhận được $y$. Ngoài tín hiệu nhận được, bộ phát hiện cũng cần ước lượng kênh $\hat{h}$, được cung cấp bởi một thiết bị ước lượng kênh.
![[Pasted image 20241214002439.png]](Pasted%20image%2020241214002439.png)

Tín hiệu nhận được $y$ có thể được biểu diễn như sau:
$$
y = Mh +n
$$
với $h$ là CIR của tín hiệu mong muốn
$$
h = \begin{bmatrix}h_{0}&h_{1}& ... &h_{L}\end{bmatrix} ^T
$$
và $n$ là nhiễu. Trong mỗi khung truyền, bên phát gửi một chuỗi bit duy nhất được chia thành một đoạn tham chiếu dài $P$ và một khoảng bảo vệ dài $L$, ký hiệu là:
$$
m = \begin{bmatrix}m_{0}&m_{1}& ... &m_{L + P + 1}\end{bmatrix} ^T
$$
có $m_i$ nhận giá trị $\{-1,1\}$ . Ma trận $M$ có dạng như sau
$$
 M = \begin{bmatrix} m_L & m_{L-1} & \cdots & m_0 \\ m_{L+1} & m_L & \cdots & m_1 \\ \vdots & \vdots & \ddots & \vdots \\ m_{L+P-1} & m_{L+P-2} & \cdots & m_{P-1} \end{bmatrix} 
$$
Ước lượng kênh LS được xác định bằng cách tối thiểu hóa bình phương.
$$\hat{h} \ =\ arg\ min\Vert y\ -\ Mh\Vert ^{2}$$

Dưới giả định nhiễu Gaussian trắng, nghiệm của bài toán này là:
$$\hat h_{LS} = (M^HM)^{-1}M^{H}y$$
Nếu hàm tự tương quan tuần hoàn (ACF) của chuỗi huấn luyện là lý tưởng, ma trận $M^HM$ trở thành ma trận chéo, giúp đơn giản hóa nghiệm thành:
 $$\hat h = \frac{1}{P} M^{H} y$$
Với các chuỗi huấn luyện GSM, nghiệm này chỉ đơn giản là mối tương quan giữa tín hiệu nhận và chuỗi bit huấn luyện.

---

### B. Bộ ước lượng kênh đồng thời cho 2 tín hiệu

Xét một hệ thống truyền thông có nhiễu đồng kênh truyền như Hình 4. Hai tín hiệu đồng bộ đồng kênh có đáp ứng xung kênh phức $h_{L,n} = [h_{0,n},h{1,n},\cdots​, h_{L,n}], n=1,2$ độc lập với nhau. Tổng của hai tín hiệu và nhiễu $n$ được lấy mẫu tại bộ thu. Vấn đề giải điều chế đồng thời là phát hiện các luồng bit truyền $a_1$​ và $a_2$​ từ tín hiệu nhận $y$. Để hỗ trợ việc này, bộ ước lượng kênh đồng thời cung cấp ước lượng $\hat h_1$​ và $\hat h_2$.
![[Pasted image 20241214113955.png]](Pasted%20image%2020241214113955.png)


Các đáp ứng xung kênh (CIR) của hai tín hiệu đồng kênh được biểu diễn bằng một vector:
$$
\tilde{h} = \begin{bmatrix} h_{L,1} \\ h_{L,2} \end{bmatrix}
$$
Các chuỗi huấn luyện của hai tín hiệu được biểu diễn bằng:
$$
h_{L,n} =\begin{bmatrix}
h_{0,n}\\
h_{1,n}\\
\vdots \\
h_{L,n}
\end{bmatrix} \ ,\ n\ =\ 1,2
$$
Hai ma trận chuỗi huấn luyện tương ứng $M_1$ và $M_2$​ được hợp nhất thành:
$$
\tilde{M} = [M_1, M_2]
$$
Tín hiệu nhận $y$ được viết lại dưới dạng:
$$
y = \tilde{M} \tilde{h} + n
$$
Ước lượng kênh LS đồng thời được tìm bằng cách tối thiểu hóa đại lượng lỗi bình phương:
$$
\hat{h} \ =\ \ \underset{h}{\arg\min}\left\Vert y\ -\ \tilde{M}\tilde{h}\right\Vert ^{2} \ =\ \left(\tilde{M}^{H}\tilde{M}\right)^{-1}\tilde{M}^{H} y
$$
Khi thiết kế chuỗi huấn luyện, việc giảm tương quan chéo giữa hai chuỗi là rất quan trọng để giảm nhiễu.

## IV. Ước lượng kênh lặp (Iterative Channel Estimation)
![[Pasted image 20241214120624.png]](Pasted%20image%2020241214120624.png)
Ý tưởng ngắn gọn là sử dụng tín hiệu đã được giải mã để đưa lại vào bộ ước lượng kênh, từ đó cập nhật ước lượng kênh trước đó, giả sử toàn bộ khung truyền hiện đã được bộ thu biết ( Hình 7).
### A. Vòng lặp đầu tiên

Giả sử kênh fading khối (không đổi trong một khung truyền), tín hiệu nhận được $r$ được biểu diễn dưới dạng:
$$
r\ =\ Ah\ +\ w\ =\ \begin{bmatrix}
r_{c1}\\
r_{m}\\
r_{c2}
\end{bmatrix} \ ,\ A\ =\ \begin{bmatrix}
A_{1}\\
M\\
A_{2}
\end{bmatrix} \ 
$$
trong đó h là CIR được ước lượng. $r_{c1}$ và $r_{c2}$ là các mẫu nhận được tương ứng với các khối dữ liệu $c_1$ và $c_2$ , $r_m$ là chuỗi bit dữ liệu đã biết. $A_1$ và $A_2$ là chuỗi dữ liệu đã truyền và $M$ là các bit huấn luyện. 
$$r_m = Mh + w$$
Có nghiệm là :
$$\hat{h}_{LS} = (M^H M)^{-1} M^H r_m$$

Sau đó, bộ phát hiện MLSE sử dụng $\hat{h}_{LS}$ để phát hiện dữ liệu mã hóa $c$:
$$\hat{c} = \arg\max_c p(r \mid c, \hat h)$$
Tiếp theo, dữ liệu đã mã hóa được đưa qua bộ giải mã để tìm ra thông tin gốc $u$:
$$\hat{u} = \arg\max_u p(\hat c \mid u)$$
### B. Các vòng lặp tiếp theo

Với dữ liệu mã hóa đã được giải mã $\hat{c}$, quá trình mã hóa lại (re-encoding) được thực hiện để tái tạo chuỗi ký hiệu. Ma trận $A$ mới được tạo ra, chứa cả chuỗi huấn luyện và dữ liệu đã giải mã:
$$A = \begin{bmatrix} A_1 & M & A_2 \end{bmatrix}$$

Dựa trên ma trận $A$, bộ ước lượng kênh có thể thực hiện lại ước lượng CIR. Nếu sử dụng phương pháp LS trên toàn bộ khung, nghiệm mới sẽ là:
$$\hat{h}_{LS}^{extend} = (A^H A)^{-1} A^H r$$
Tuy nhiên, việc tính toán nghịch đảo ma trận $A^H A$ có thể rất phức tạp. Vì vậy, quy tắc thích nghi như Least Mean Square (LMS) được sử dụng để cập nhật ước lượng kênh:
$$\hat{h}_{k+1} = \hat{h}_k + \mu A_k^H (A_k \hat{h}_k - r)$$
trong đó $\hat h_{k+1}$ là ước lượng mới, $A_k$ là mà trận dữ liệu bao gồm những tín hiệu đã biết (data + training), $r$ là những mẫu nhận được và $\mu$ là số bước lặp.


### C. Mô phỏng hệ thống lặp
![[Pasted image 20241214123506.png]](Pasted%20image%2020241214123506.png)

Hệ thống lặp có thể được mô phỏng theo cấu trúc pipeline như trong Hình 8. Nhiều module xử lý được đặt nối tiếp nhau, mỗi module tương ứng với một vòng lặp. Điều này cho phép bộ thu bắt đầu xử lý khối tín hiệu tiếp theo trong khi khối trước đó vẫn đang được xử lý trong các vòng lặp bổ sung. Cấu trúc này cũng giúp đánh giá hiệu suất bộ thu (như BER, BLER) sau mỗi vòng lặp trong mô phỏng.
