## 简介

在考虑参与介质的存在时，折射对真实世界的渲染起着重要作用，其发生于光从一种介质射入到另一种介质时。最简单的例子如空气/介质，其中/表示表面。

整体参考自[1]

## 1.折射（Refraction）

### 1.1斯涅尔定律（Snell’s law）

#### 1.1.1定义

斯涅尔定律描述了当光通过两种不同的介质之间发射折射时，光是如何改变方向的。其入射光和折射光都处于同一平面，称为“入射平面”，并且与界面法线的夹角满足如下关系：

![](http://latex.codecogs.com/svg.latex?\sin \theta_{i} \cdot n_{1}=\sin \theta_{t} \cdot n_{2})

（<math>\sin \theta_{i} \cdot n_{1}=\sin \theta_{t} \cdot n_{2}</math>）

<div align=center>![](https://renderwiki.github.io/ImageResources/Refraction/折射示意图.png)</div>

<center>图1 折射示意图 </center>

#### 1.1.2适用范围

此定律是几何光学的基本实验定律，它适用于由均匀的各向同性的介质。

### 1.2Radiance的计算

由于能量守恒，任何没有被反射的光都是透射的。假设反射光量与入射光量的比例是![](http://latex.codecogs.com/svg.latex?F)，透射光量与入射光量的比例是![](http://latex.codecogs.com/svg.latex?1-F)。在给定入射角度![](http://latex.codecogs.com/svg.latex?\theta_{i})下，出射角度为![](http://latex.codecogs.com/svg.latex?\theta_{t})的辐射度可以通过以下公式进行计算：（<math>F</math>)、（<math>1-F</math>)、（<math>\theta_{i}</math>)、（<math>\theta_{t}</math>)

<div align=center>![](https://renderwiki.github.io/ImageResources/Refraction/radiance公式1.svg)</div>

(<math>L_{t}=\left(1-F\left(\theta_{i}\right)\right) \frac{\sin ^{2} \theta_{i}}{\sin ^{2} \theta_{t}} L_{i}</math>)

应用斯涅尔定律，该公式也可写为：

<div align=center>![](https://renderwiki.github.io/ImageResources/Refraction/基于斯涅尔定律的radiance公式.svg)</div>

(<math>L_{t}=\left(1-F\left(\theta_{i}\right)\right) \frac{n_{2}^{2}}{n_{1}^{2}} L_{i}</math>)

其中![](https://latex.codecogs.com/svg.image?F(\theta_{i}))为入射角度![](https://latex.codecogs.com/svg.image?\theta_{i})下的菲涅尔系数，![](https://latex.codecogs.com/svg.image?L_{i})为入射辐射度，![](https://latex.codecogs.com/svg.image?n_{1})和![](https://latex.codecogs.com/svg.image?n_{2})分别为入射时光所处介质的折射率与出射时光所处介质的折射率。(<math>F(\theta_{i})</math>)、(<math>L_{i}</math>)、（<math>n_{1}</math>)、（<math>n_{2}</math>)

### 1.3折射向量的计算方法

Bec[2]提出了一种有效的方法来计算折射向量：

<div align=center>![](https://renderwiki.github.io/ImageResources/Refraction/折射向量计算公式.svg)</div>

（<math>\mathbf{t}=(w-k) \mathbf{N}-n \mathbf{l}</math>）

其中

<div align=center>![](https://renderwiki.github.io/ImageResources/Refraction/折射公式附加说明.svg)</div>

（<math>\begin{aligned}
w &=n(\mathbf{l} \cdot \mathbf{N}) \\
k &=\sqrt{1+(w-n)(w+n)}
\end{aligned}</math>）

得到的折射向量![](https://latex.codecogs.com/svg.image?\mathbf{t})被归一化，![](https://latex.codecogs.com/svg.image?\mathbf{N})为表面法线，![](https://latex.codecogs.com/svg.image?\mathbf{l})为光的入射方向。![](https://latex.codecogs.com/svg.image?n=n_{1}/n_{2})为相对折射率，也是传统上用于斯涅尔方程中的折射率。水的折射率大约为1.33，玻璃的折射率通常为1.5，而空气的折射率为1.0。（<math>n=n_{1}/n_{2}</math>)

### 1.4实时渲染方法

#### 1.4.1环境贴图（Environment Map，EM)

渲染折射的一般方法是，从发生折射的位置生成一个立体的环境贴图（EM），当这个物体被渲染时，可以通过正表面计算的折射方向来查询EM。

#### 1.4.2屏幕空间方法（Screen-space Approach）

Sousa[3]提出了一种屏幕空间的方法，它是基于使用当前的缓冲区作为折射图，然后在纹理坐标上添加扰动来模拟折射。

首先，把场景中所有没有发生折射的物体，像往常一样渲染到场景纹理![](https://latex.codecogs.com/svg.image?\mathbf{s})中。这个纹理可以用来确定哪些物体在折射物体后面是可见的，这些物体将在随后的渲染过程中被渲染出来。

其次，将所有的折射物体渲染到![](https://latex.codecogs.com/svg.image?\mathbf{s})的alpha通道（已被清除为1)。如果像素通过了深度测试，即折射物体在该像素中的最前面，那么就在alpha通道的该像素中写入0值。

最后，折射物体被完全渲染。在像素着色器中，![](https://latex.codecogs.com/svg.image?\mathbf{s})根据像素在屏幕上的位置进行扰动采样，其扰动的偏移量来自于如缩放的表面法线的红色和绿色xy分量，从而模拟折射。被扰动的采样的颜色只有其alpha值为0才会被考虑。这样是为了避免使用来自折射物体前面的表面的样本，从而使它们的颜色被采用，就像它们在后面一样。

#### 1.4.3粗糙材质表面的渲染

对于粗糙的折射表面，根据材质粗糙度模糊背景，以模拟由微几何法线分布引起的折射方向上的漫反射是很重要的。在游戏《DOOM》（2016）中，场景首先像往常一样被渲染。然后，它被下采样到一半的分辨率，并进一步降到四个mipmap级别。每个mipmap级别模仿GGX BRDF lobe的高斯模糊度进行下采样。最后在 渲染时，将材质的粗糙度映射到mipmap级别，并在相应级别的mipmap上采样，再将背景合成到表面的后面。表面越是粗糙，背景就越是模糊。

参考文献：

[1] Tomas Akenine-Mller, Eric Haines, and Naty Hoffman. 2018. Real-Time Rendering, Fourth Edition (4th. ed.). A. K. Peters, Ltd., USA.

[2] Bec, Xavier, “Faster Refraction Formula, and Transmission Color Filtering,” Ray Tracing News, vol. 10, no. 1, Jan. 1997.

[3] Sousa, Tiago, “Generic Refraction Simulation,” in Matt Pharr, ed., GPU Gems 2, AddisonWesley, pp. 295–305, 2005.

