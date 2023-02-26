**实对称矩阵的对角化（特征值分解）** 是 **奇异值分解** 的基础，因此，本文在介绍奇异值分解之前先对实对陈矩阵的对角化进行回顾。最后介绍如何利用奇异值分解进行图像压缩。

## 1 实对称矩阵的对角化
**定理** 对于 $n$ 阶实对陈矩阵 $A$, 必存在正交矩阵 $P$ ,使 $P^{-1}AP = P^TAP = \Lambda$ , 其中 $\Lambda$ 是以 $A$ 的 $n$ 个特征值为对角元素的对角阵。

也就是说，若 $A$ 为 $n$ 阶实对陈矩阵，则可以将其表示/分解为以下形式：
$$
\begin{aligned}
A
&=
P\Lambda P^{-1} \\
&=
P\Lambda P^T \\
&=
P
\begin{bmatrix}
\lambda_1 & & & \\
& \lambda_2 & & \\
& & \ddots & \\
& & & \lambda_n
\end{bmatrix}
P^{-1}
\end{aligned}
$$
其中，$P$ 为正交矩阵， $\lambda_i$ 为 $A$ 的特征值。

## 2 奇异值分解
### 2.1 定义
实际上，对于任意 $m \times n$ 的实矩阵 $A$ ，即使 $m != n$, 我们也可以对其进行“对角化“，也就是所谓的奇异值分解，其定义如下：

矩阵 $A$ 可以被分解为以下形式：
$$A = U \Lambda V^T$$
其中 $U$ 和 $V$ 均为正交矩阵； $U \in R^{m \times m}$ 称为左奇异矩阵， $V \in R^{n \times n}$ 称为右奇异矩阵；$\Lambda \in R^{m \times n}$ 仅在主对角线上有元素，这些元素被称为奇异值， $\Lambda$ 一般有如下形式：

$$
\Lambda
=
\begin{bmatrix}
\sigma_1 & 0 & 0 & 0 & 0 \\
0 & \sigma_2 & 0 & 0 & 0 \\
0 & 0 & \ddots & 0 & 0 \\
0 & 0 & 0 & \sigma_m & 0
\end{bmatrix}
$$
奇异值 $\sigma_i$ 是按从大到小的顺序排的，即 $\sigma_1 >= \sigma_2 >= ... >= \sigma_m$。



### 2.2 求解
那么 如何求解 $U, V, \Lambda$ 呢？考虑以下性质：
$$
AA^T = U \Lambda V^T V\Lambda^TU^T = U\Lambda\Lambda^TU^T \\
A^TA = V\Lambda^TU^TU\Lambda V^T = V\Lambda^T\Lambda V^T
$$

首先明确两点
- $AA^T$ 和 $A^TA$ 均为实对称矩阵；
- $\Lambda\Lambda^T$ 和 $\Lambda^T\Lambda$ 均为对角阵；

可见：
- $U$ 实际上是实对称矩阵 $AA^T$ 对角化所使用的正交矩阵；
- $V$ 实际上是实对称矩阵 $A^TA$ 对角化所使用的正交矩阵；

因此，我们可以：
- 先计算实对称矩阵 $AA^T$ ,然后对其进行对角化，得到正交矩阵 $U$ ；
- 先计算实对称矩阵 $A^TA$, 然后对其进行对角化，得到正交矩阵 $V$ ；
- 最终，得到 $A$ 的奇异值分解 $A = U\Lambda V^T$

## 3 奇异值分解压缩图像
图像本质上可以看作是一个二维矩阵，因此，我们也可以对其进行 SVD 分解，这没毛病。但是利用它来压缩图片是怎么回事呢，下面我们就来看看。

假设，一幅图像，其像幅大小为 $m * n, m > n$ ，用矩阵 $M$ 该图像，其 SVD 分解为 $M = U\Lambda V^T$ 。

把 $U$ 用其列向量 $u_i$ （ $M$ 的左奇异向量）表示为：
$$U = (u_1, u_2, ..., u_n)$$

把 $V$ 用其列向量 $v_i$ （ $M$ 的右奇异向量）表示为：
$$V = (v_1, v_2, ..., v_n)$$

假设，对角阵 $\Lambda$ 的对角线上元素为 $d_i$ （ $M$ 的奇异值）,那么 $M$ 可被表示为：
$$
\begin{aligned}
M
&=
d_1u_1v_1^T + d_2u_2v_2^T + ... + d_nu_nv_n^T\\
&=
\sum_{i = 1}^{n}d_iu_iv_i^T \\
&=
\sum_{i = 1}^{n}A_i
\end{aligned}
$$
其中 $A_i = d_iu_iv_i^T$ 是一个 $m \times n$ 的矩阵，也就是说，我们把原来的矩阵 $M$ 表示成了 $n$ 个矩阵的和。

每个矩阵 $A_i$ 对 $M$ 的“贡献”是不同的，而奇异值 $d_i$ 一定程度上可以用来衡量其对应项 $A_i$ 对 $M$ “贡献”的大小。

SVD图像压缩方法的思路就是：**从 $A_i$ 中提取出那些对 $M$ 贡献较大的项（奇异值 $d_i$ 较大的项），把它们的和作为对 $M$ 的近似**。

也就是说，用：
$$M_k = \sum_{i = 1}^k A_i$$
来对 $M_n \equiv M$ 进行近似。

接下来讨论压缩效率：
- 未压缩前：图像需要存储 $m * n$ 个元素；
- 压缩后：只需要存储 $U$ 的前 $k$ 列 $u_1,u_2,...,u_k$ 、 $V$ 的前 $k$ 列 $v_1,v_2,...,v_k$ 、$\Lambda$ 的前 $k$ 个元素 $d_1,d_2,...,d_k$ ，共需存储 $k(m + n + 1)$ 个元素；

因此，图像的压缩比公式为：
$$\rho = \frac{k(m + n + 1)}{mn}$$


本文基于 OpenCV 进行了实现，代码如下：
```cpp
#include <opencv2/core/core.hpp>
#include <opencv2/highgui/highgui.hpp>

#include <vector>
using namespace cv;
using namespace std;

Mat svd_compress(const Mat& src_channel, int k) {
    Mat temp;
    src_channel.convertTo(temp, CV_32FC1);
    
    Mat U, w, Vt;
    //opencv得到的V已经经过转置了
    //w为一个行矩阵，而非对角阵
    SVD::compute(temp, w, U, Vt);

    Mat res(src_channel.size(), CV_32FC1, Scalar(0));
    for(int i = 0; i < k; ++i)
    {
        res += w.ptr<float>(i)[0] * (U.col(i) * Vt.row(i));
    }
    
    /*Mat W(w.rows, w.rows, CV_32FC1, Scalar(0));
    for (int i = 0; i < k; i++)
        W.ptr<float>(i)[i] = w.ptr<float>(i)[0];
    res = U * W * Vt;*/

    return res;
}

int main() {
    Mat src = imread("../data/1.jpg");
    imshow("原图", src);
    vector<Mat> src_channels;
    split(src, src_channels);

    vector<Mat> dst_channels(src.channels());
    const int k = 20;
    for(int i = 0; i < src_channels.size(); ++i)
    {
        dst_channels[i] = svd_compress(src_channels[i], k);
        dst_channels[i].convertTo(dst_channels[i], CV_8UC1);
    }
    Mat dst;
    merge(dst_channels, dst);
    imshow("压缩图", dst);
    waitKey(0);

    
    return 0;
}
```
以下为实现效果图：

原图，1080 * 1920：

{% asset_img 0.jpg %}

$k = 1$ , 压缩比： $0.0014$
{% asset_img 1.jpg %}

$k = 5$ , 压缩比： $0.0072$
{% asset_img 5.jpg %}

$k = 100$ , 压缩比： $0.1447$
{% asset_img 100.jpg %}

$k = 500$, 压缩比： $0.7236$
{% asset_img 500.jpg %}

当 k = 500 时，压缩后的图像与原图看起来基本一样，此时压缩比为 0.7236，可见，压缩效率还是挺好的。


思考：假设，我们使用 $||M_k - M_n||_2$ 来作为评价近似结果好坏的指标； 显然，当 $k = n$ 时，近似效果最好；那么当 $k != n$ 时，$k$ 取什么值的时候，近似效果最好？

## 参考
[https://www.cnblogs.com/endlesscoding/p/10033527.html](https://www.cnblogs.com/endlesscoding/p/10033527.html)

[https://cosx.org/2014/02/svd-and-image-compression](https://cosx.org/2014/02/svd-and-image-compression)