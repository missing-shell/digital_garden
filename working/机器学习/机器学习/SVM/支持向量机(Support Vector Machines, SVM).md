#SVM #监督学习 #回归分析
支持向量机(Support Vector Machines, SVM)是一种监督学习模型，主要用于分类和回归分析。SVM通过==寻找最佳的决策边界==——即**最大间隔超平面**，来实现数据分类。**代价最小**

### 优化目标
SVM 与对数几率回归（logistic regression）的不同视角：

* SVM目标--**最大化间隔**：
		**SVM的目标是在保证分类正确的同时，最大化分类边界与最近样本点的距离，从而得到最大的间隔（margin）。如果$y^{(i)} = 1$，我们希望$y^{(i)}(\theta^Tx^{(i)}) \geq 1$；如果$y^{(i)} = -1$，我们同样希望$y^{(i)}(\theta^Tx^{(i)}) \geq 1$。**
	
- **对数几率**：$$ P(y=1|x) = \frac{1}{1 + e^{-\theta^T x}} $$
### ### SVM 假设函数
#假设函数
- 基本形式：
  $$ h_\theta(x) = \begin{cases} 
  1 & \text{if } \theta^T x \geq 0 \\
  0 & \text{otherwise}
  \end{cases} $$

### 成本函数

对于单个训练样本的成本函数，如果$y^{(i)} = 1$（希望分类正确），成本函数为0；如果$y^{(i)} = -1$（分类错误），成本函数为无穷大。SVM的成本函数可以表示为：
$$
\begin{align*}
\text{Cost}_{\text{SVM}}(y, h_\theta(x)) &= \left\{
\begin{array}{ll}
0 & \text{if } y(\theta^Tx) \geq 1 \\
\infty & \text{otherwise}
\end{array} \right.
\end{align*}
$$

### 大间距直觉

SVM寻找能够将两类数据最大程度分开的决策边界，这种决策边界称为大间距分类器。它通过最小化以下函数实现：
$$
\begin{align*}
\min_{\theta} \quad &\frac{1}{2}\sum_{j=1}^{n}\theta_j^2 \\
\text{s.t.} \quad &y^{(i)}(\theta^Tx^{(i)}) \geq 1 \quad \forall 1 \leq i \leq m
\end{align*}
$$
### SVM 决策边界数学
- 线性可分情况下的 SVM 目标函数：
  $$ J(\theta) = \lambda \sum_{i=1}^m y^{(i)} \theta^T x^{(i)} - \sum_{j=1}^n \theta_j^2 $$
- 其中，$\lambda$ 是正则化参数，控制模型复杂度。

### 向量内积与投影
- 向量 $u$ 在向量 $v$ 上的投影长度 $p$：
  $$ p = \frac{u_1 v_1 + u_2 v_2}{||v||} $$
### 核函数（Kernels）

对于非线性可分的情况，SVM通过核函数(kernel function)将原始特征映射到更高维空间，使得数据在该空间中变得线性可分。常见的核函数包括高斯核函数（Gaussian Kernel）：
$$
K(x, l) = \exp(-\frac{\|x-l\|^2}{2\sigma^2})
$$
#### 高斯核函数（Gaussian Kernel）
- 定义为：
  $$ K(x_1, x_2) = e^{-\frac{||x_1 - x_2||^2}{2\sigma^2}} $$
- 其中，$\sigma$ 是核函数的参数。

#### 核函数与相似度
- 核函数可以视为度量数据点间的相似度。

### 决策边界

SVM的决策边界是$\theta^Tx = 0$，这意味着$\theta$向量与$x$向量垂直。
### SVM 多类分类
- 使用一对多（one-vs.-all）方法或软件包内置的多类分类功能。

### 对比对数几率回归与 SVM
- 根据特征数（$m$）与训练样本数（$n$）选择合适的模型：
  - $m$ 大且 $n$ 相对较小时：使用对数几率回归或线性 SVM。
  - $m$ 中等且 $n$ 较大时：使用高斯核 SVM。
  - $m$ 小且 $n$ 很大时：添加更多特征，然后使用对数几率回归或线性 SVM。
### SVM参数

SVM有两个主要参数：$C$和$\sigma$。$C$控制了误分类的惩罚程度，$C$越大，对误分类的容忍度越低；$\sigma$控制了核函数的宽度，$\sigma$越小，核函数对距离敏感度越高。

### 使用SVM

在使用SVM软件包（如liblinear, libsvm）时，需要指定参数$C$和选择核函数（如线性核、高斯核等）。如果特征数量$n$相对于样本数量$m$较大，推荐使用逻辑回归或SVM无核函数（线性核）。如果$n$较小，$m$介于中等大小，使用带有高斯核的SVM。如果$n$较小，但$m$很大，可以通过增加更多特征，然后使用逻辑回归或无核函数的SVM。

### SVM与神经网络比较

SVM相比于神经网络的优势在于其代价函数是**凸函数**，确保了*全局最优解*，而且运行速度较快。然而，神经网络具有更好的通用性，能够在各种问题上取得较好的效果，尽管训练速度可能较慢。