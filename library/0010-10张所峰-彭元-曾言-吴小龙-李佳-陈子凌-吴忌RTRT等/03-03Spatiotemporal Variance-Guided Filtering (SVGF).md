## SVGF（Spatiotemporal Variance-Guided Filtering）

目前实时后处理去噪领域的基准（baseline）算法是SVGF（Spatiotemporal Variance-Guided Filtering），使用类似于时间反走样TAA（temporal anti-aliasing）的思想，结合时间和空间上的信息做滤波（filter），获得重建序列。

<div align=center>![SVGF结果](https://renderwiki.github.io/ImageResources/SVGF/SVGF结果.png)</div>

<center>图1 1spp噪声输入（左）、SVGF输出（中）和2048spp收敛图（右）对比</center>


## 1.算法和流程

SVGF算法的主要流程如图1.1所示。首先对于从路径追踪渲染器中获得的1spp噪声输入，分解成直接光照和间接光照，解调反照率（除以albedo）获得辐照度（irradiance），对直接光照和间接光照的辐照度分别应用时空混合滤波后进行合并，再把albedo乘回来，恢复原本的纹理信息，并应用色调映射（Tone Mapping）。最后还要采用TAA以去除时间上的抖动，保证序列的时间稳定性。

<div align=center>![SVGF流程图](https://renderwiki.github.io/ImageResources/SVGF/SVGF流程图.png)</div>

<center>图2 SVGF流程图</center>

SVGF的核心之一在于时空混合滤波，它包含一个空间的A-Trous小波滤波和一个时域滤波。接下来分别对两种滤波进行解释：

## 2.滤波

### 2.1空间滤波：A-Trous小波滤波

SVGF使用时域累积的方法，对每像素颜色流明的方差进行估计，用于检测噪声，对于很少或者几乎没有噪声的区域尽量不改变，而对于更多噪声的区域加强滤波。
空间的A-Trous小波滤波与交叉双边滤波（联和双边滤波）类似，它是对交叉双边滤波在小波分解频域上的空间快速近似。双边滤波与高斯滤波、均值滤波等线性滤波不同，双边滤波是局部非线性的，它通过一个边缘停止函数（edge stopping），可以实现在平滑区域和边缘区域的不同响应，避免将边缘信息也模糊掉。交叉/联和双边滤波（Cross/Joint Bilateral Filtering）则是同时考虑多种边缘停止函数。
由于从光栅化阶段获得的几何缓存（G buffer）是完全无噪声的，因此可以用来作为辅助输入指导滤波的设计。SVGF算法实现了边缘感知的A-Trous小波分解，公式如下：

<div align=center>![公式1](https://renderwiki.github.io/ImageResources/SVGF/公式1.png)</div>

<math>\hat{c}_{i+1}(p)=\frac{\sum_{q \in \Omega} h(q) \cdot w(p, q) \cdot \hat{c}_{i}(q)}{\sum_{q \in \Omega} h(q) \cdot w(p, q)}</math>

该公式用于时域上的颜色积累，其中，i为第i帧，![](http://latex.codecogs.com/svg.latex?h=(\frac{1}{16},\frac{1}{4},\frac{3}{8},\frac{1}{4},\frac{1}{16}))是5×5的联合双边滤波核，![](http://latex.codecogs.com/svg.latex?w(p, q))是像素![](http://latex.codecogs.com/svg.latex?p)和![](http://latex.codecogs.com/svg.latex?q)之间的的权重函数。根据这一公式，将小波滤波应用到每一次的时间累积颜色上。假设方差样本是不相关的，方差估计由以下公式进行滤波：

<div align=center>![公式2](https://renderwiki.github.io/ImageResources/SVGF/公式2.png)</div>

<math>\operatorname{Var}\left(\hat{c}_{i+1}(p)\right)=\frac{\sum_{q \in \Omega} h(q)^{2} \cdot w(p, q)^{2} \cdot \operatorname{Var}\left(\hat{c}_{i}(q)\right)}{\left(\sum_{q \in \Omega} h(q) \cdot w(p, q)\right)^{2}}</math>

利用上式的结果将边缘停止函数导向下一层的a-trous变换。使用一个5层的小波变换，获得一个高效的65×65像素滤波足迹（footprint）。从第一个小波迭代中输出滤波结果作为颜色历史，用于累积到未来帧。

以往的权重函数主要为结合了基于几何和颜色的边缘停止函数，而SVGF方法使用屏幕空间深度、世界空间法向和输入的上一帧亮度（流明）的方差：

<div align=center>![公式3](https://renderwiki.github.io/ImageResources/SVGF/公式3.png)</div>

<math>w_{i}(p, q)=w_{z} \cdot w_{n} \cdot w_{l}</math>

其中，![](http://latex.codecogs.com/svg.latex?w_{z})是深度边缘停止函数，![](http://latex.codecogs.com/svg.latex?w_{n})是法向边缘停止函数，![](http://latex.codecogs.com/svg.latex?w_{l})是流明边缘停止函数。每个函数的拒绝样本的能力由参数![](http://latex.codecogs.com/svg.latex?\sigma_{z}),![](http://latex.codecogs.com/svg.latex?\sigma_{n})和![](http://latex.codecogs.com/svg.latex?\sigma_{l})分别控制。


#### 2.1.1深度边缘停止函数

<div align=center>![公式4](https://renderwiki.github.io/ImageResources/SVGF/公式4.png)</div>

<math>w_{z}=\exp \left(-\frac{|z(p)-z(q)|}{\sigma_{z}\left|\nabla_{z}(p) \cdot(p-q)\right|+\varepsilon}\right)</math>

![](http://latex.codecogs.com/svg.latex?\nabla Z)是裁剪空间的相对于屏幕空间的深度的梯度，一个![](http://latex.codecogs.com/svg.latex?\varepsilon)为了防止除以零的很小的值。![](http://latex.codecogs.com/svg.latex?pq)的深度差别越大，函数值越小。

#### 2.1.2法向边缘停止函数

<div align=center>![公式5](https://renderwiki.github.io/ImageResources/SVGF/公式5.png)</div>

<math>w_{n}=\max (0, n(p) \cdot n(q))^{\sigma_{n}}</math>

![](http://latex.codecogs.com/svg.latex?n(p))为图像平面上的点![](http://latex.codecogs.com/svg.latex?p)的输入法向，![](http://latex.codecogs.com/svg.latex?n(q))为点![](http://latex.codecogs.com/svg.latex?q)的法向，![](http://latex.codecogs.com/svg.latex?pq)两点所处的平面的法向差距越大，函数值越小，当超过90度时则为0。

#### 2.1.3流明边缘停止函数

<div align=center>![公式6](https://renderwiki.github.io/ImageResources/SVGF/公式6.png)</div>

<math>w_{l}=\exp \left(-\frac{\left|l_{i}(p)-l_{i}(q)\right|}{\sigma_{l} \sqrt{g_{3 \times 3}\left(\operatorname{Var}\left(l_{i}(p)\right)\right)}+\varepsilon}\right)</math>

![](http://latex.codecogs.com/svg.latex?g_{3 \times 3})为3×3的高斯核，用于预先滤波流明方差以防止噪声。需要注意的是，这个预滤波的高斯函数只用于驱动流明边缘停止函数，而不用于传播到小波变换下一次迭代的方差图像。

<div align=center>![是否使用高斯核](https://renderwiki.github.io/ImageResources/SVGF/是否使用高斯核.png)</div>

<center>图3 1spp噪声输入（左）、SVGF输出（中）和2048spp收敛图（右）对比</center>


### 2.2时间滤波

<div align=center>![公式7](https://renderwiki.github.io/ImageResources/SVGF/公式7.png)</div>

<math>\bar{c}_{\mathrm{i}}(x, y)=\alpha \cdot \tilde{c}_{\mathrm{i}}(x, y)+(1-\alpha) \cdot \bar{c}_{\mathrm{i}-1}^{\text {warped }}(x, y)</math>

![](http://latex.codecogs.com/svg.latex?-)为根据上述公式混合的结果，![](http://latex.codecogs.com/svg.latex?\sim)代表应用滤波，![](http://latex.codecogs.com/svg.latex?warped)为重投影（利用运动矢量进行时间重用，具体为扭曲上一帧使之当前帧对齐），上述公式可以看成是当前帧的滤波结果和重投影后的上一帧的混合结果的线性组合，![](http://latex.codecogs.com/svg.latex?\alpha)为混合标量权重，SVGF将其设置为定值0.2。注意，SVGF使用的是传统的基于屏幕空间的运动矢量。为了减小拖尾，使用一个2×2的tap双线性滤波器去重新采样![](http://latex.codecogs.com/svg.latex?\bar{C}_{\mathrm{i}-1}^{\text {warped }})，如果一个tap中含有不一致的几何形状，样本将被丢弃，其权重将在一致的tap上均匀地重新分配。如果没有tap保持一致，则使用一个更大的3×3滤波器来帮助寻找的细碎的几何形状。如果仍然无法找到一致的几何形状，则该样本处于运动遮挡处，因此丢弃时间历史，令：

<div align=center>![公式8](https://renderwiki.github.io/ImageResources/SVGF/公式8.png)</div>。

<math>\bar{c}_{\mathrm{i}}(x, y)=\tilde{c}_{\mathrm{i}}(x, y)</math>

参考文献：

[1]Christoph Schied, Anton Kaplanyan, Chris Wyman, Anjul Patney, Chakravarty R. Alla Chaitanya, John Burgess, Shiqiu Liu, Carsten Dachsbacher, Aaron Lefohn, and Marco Salvi. 2017. Spatiotemporal variance-guided filtering: real-time reconstruction for path-traced global illumination. In Proceedings of High Performance Graphics (HPG '17). Association for Computing Machinery, New York, NY, USA, Article 2, 1–12. DOI:https://doi.org/10.1145/3105762.3105770
