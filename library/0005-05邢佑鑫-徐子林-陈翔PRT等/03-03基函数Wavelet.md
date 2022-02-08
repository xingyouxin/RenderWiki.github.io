## 简介

小波（Wavelet）是一组基函数，通过母小波（mother wavelet）和尺度函数（scaling function）经过平移和缩放生成而来，被广泛应用于信息处理和数据压缩中。
不同的母小波和尺度函数会产生不同的小波基函数，在渲染领域，小波通常指哈尔小波（Haar Wavelet）。

## 1.哈尔小波

### 1.1 一维哈尔小波
哈尔小波是最早提出且最简单的小波函数，其母小波 ![](http://latex.codecogs.com/svg.latex?\psi(x)) 和尺度函数 ![](http://latex.codecogs.com/svg.latex?\phi(x)) 定义如下：

<div align=center>![](ImageResources/Wavelet/mother.png)</div>

<div align=center>![](ImageResources/Wavelet/scaling.png)</div>

对原始的母小波和尺度函数进行平移和缩放，可以得到 ![](http://latex.codecogs.com/svg.latex?j) 阶的哈尔小波基函数公式：

<div align=center>![](ImageResources/Wavelet/motherL2.png)</div>

<div align=center>![](ImageResources/Wavelet/scalingL2.png)</div>

下面给出1阶哈尔母小波 ![](http://latex.codecogs.com/svg.latex?\psi^1(x)) 和 2 阶哈尔尺度函数 ![](http://latex.codecogs.com/svg.latex?\phi^2(x)) 的示意图：

<div align=center>![](ImageResources/Wavelet/motherL2Fig.png)</div>

<div align=center>![](ImageResources/Wavelet/scalingL2Fig.png)</div>

将函数投影到小波基上可以得到一组系数，这个过程也被称为小波变换（wavelet transform）。

可以看到，基函数阶数越高，其能够表示的信息频率也越高，因此哈尔小波基具有表示全频率（All-frequency）的性质。

同时不难发现哈尔小波基之间也具备正交（orthogonal）性。

### 1.2 二维哈尔小波
二维哈尔小波仅仅是对一维哈尔小波的简单拓展，即对行和列分别进行一维哈尔小波变换。
行列变换顺序的不同会得到一组完全不同的小波基，在渲染领域，一般采用行列交替变换的方法（这种方法也被称作 non-standard decomposition）：
<div align=center>![](ImageResources/Wavelet/nonStrandDecomp.png)</div>

### 1.3 系数压缩
将一个函数（通常是一张图片）经过小波变换后，得到的系数往往是稀疏的，即大多数系数都是0或很小的值。

一般来说，保留其中最大 ![](http://latex.codecogs.com/svg.latex?0.5\% \sim 10\%) 的系数就足以还原出接近原函数的结果。

下图是使用相同系数下哈尔小波（W）与 SH 和 Reference 的比较：
<div align=center>![](ImageResources/Wavelet/compareSH.png)</div>

## 2 应用
小波基函数在渲染中被广泛应用于 PRT。其最早被 Ng 等人[1]引入，一开始只能用于 diffuse 场景和固定视角的 glossy 场景的环境光 relighting。

很快 Ng 等人[2]提出了 Triple product wavelet integral 算法，分离了 BRDF 项，使其支持可变视角的 glossy 场景 relighting。 

之后 Sun 等人[3]提出了 Affine Double and Triple Product Wavelet Integrals 算法，用于 near-field lighting。

参考文献：

[1] Ren Ng, Ravi Ramamoorthi, and Pat Hanrahan. 2003. All-Frequency Shadows using Non-Linear Wavelet Lighting Approximation. ACM Transactions on Graphics (Proceedings of SIGGRAPH) 22, 3 (2003), 376–381.

[2] Ren Ng, Ravi Ramamoorthi, and Pat Hanrahan. 2004. Triple product wavelet integrals for all-frequency relighting. In ACM SIGGRAPH 2004 Papers. 477–487.

[3] Bo Sun and Ravi Ramamoorthi. 2009a. Affine Double and Triple Product Wavelet Integrals for Rendering. ACM Transactions on Graphics 28, 2 (2009), 14:1–14:17.

[4] E. J. Stollnitz, A. D. DeRose and D. H. Salesin, "Wavelets for computer graphics: a primer.1," in IEEE Computer Graphics and Applications, vol. 15, no. 3, pp. 76-84, May 1995, doi: 10.1109/38.376616.
