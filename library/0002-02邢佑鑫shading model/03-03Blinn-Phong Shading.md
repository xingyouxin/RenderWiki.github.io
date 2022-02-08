## 简介

**Lambertian着色与视线方向无关**：从任何方向观察着色表面，其着色结果都相同。很多真实表面可以显示出一定程度的光泽感，产生高光或者镜面反射现象，而且这些反射会随着视点的变化而移动。Lambertian着色不会产生任何高光，并且会导致非常暗，类似石灰白般的外观，因此许多着色模型**为Lambertian着色添加了镜面反射部分**，**将Lambertian着色视为漫反射部分**。

Phong（Phong，1975）提出了一种非常简单且广泛使用的镜面高光模型，后来由Blinn（J. F. Blinn，1976）更新为今天最常用的形式，即**Blinn-Phong着色模型**。

## 1.Blinn-Phong着色

### 1.1基本概念

**Blinn-Phong着色的基本思路**是在视线向量![](http://latex.codecogs.com/svg.latex?v)和光线向量![](http://latex.codecogs.com/svg.latex?l)与法线对称时，即当发生镜面反射时产生最亮的反射。当向量偏离镜像位置时，反射将会平滑减小。

半向量![](http://latex.codecogs.com/svg.latex?h)（平分![](http://latex.codecogs.com/svg.latex?v)和![](http://latex.codecogs.com/svg.latex?l)之间夹角的向量）与表面法线可以被用来比较判断视线向量![](http://latex.codecogs.com/svg.latex?v)和光线向量![](http://latex.codecogs.com/svg.latex?l)与法线**接近镜面对称的程度**，如图1。

<div align=center>![Blinn-Phong着色示意图](https://renderwiki.github.io/ImageResources/shading model/Blinn-Phong着色示意图.png)</div>

<center>图1 Blinn-Phong着色示意图</center>

如果半向量**接近**表面法线，镜面反射部分应该变得非常亮；如果半向量**远离**法线，则镜面反射部分会变暗。通过计算![](http://latex.codecogs.com/svg.latex?h)和![](http://latex.codecogs.com/svg.latex?n)之间的点积来对其进行评估。由于两个向量都是单位向量，所以当两个向量相等时，![](http://latex.codecogs.com/svg.latex?n \cdot h)将达到最大值1。对点乘结果取![](http://latex.codecogs.com/svg.latex?p)幂（![](http://latex.codecogs.com/svg.latex?p)大于1）来确保这一结果更快衰减。该**幂指数**或者说**Phong指数**用于控制表面的光泽程度。

**几种具有代表性的![](http://latex.codecogs.com/svg.latex?p)值**：

- 10——蛋壳漆；
- 100——轻微有光泽；
- 1000——非常有光泽；
- 10,000——近乎镜面。

### 1.2半向量的计算方法

**半向量很容易计算**：由于![](http://latex.codecogs.com/svg.latex?v)和![](http://latex.codecogs.com/svg.latex?l)的长度相同，因此对它们求和恰好对应一个将它们之间角度均分的向量，只需要对二者之和进行归一化便可以产生![](http://latex.codecogs.com/svg.latex?h)。

<div align=center>![半向量h计算公式](https://renderwiki.github.io/ImageResources/shading model/半向量h计算公式.png)</div>

### 1.3公式表述

综上所述，**Blinn-Phong着色模型如下所示**：

<div align=center>![BlinnPhong着色模型公式](https://renderwiki.github.io/ImageResources/shading model/BlinnPhong着色模型公式.png)</div>

其中![](http://latex.codecogs.com/svg.latex?k_s)是表面的镜面系数或镜面反射颜色。



ac.wiki所需的公式：

<math>\pmb{h}=\frac{(\pmb{v+l})}{||\pmb{v+l}||}  \tag{1}</math>

<math>L=k_dImax(0, \pmb{n·l})+k_sImax(0, \pmb{n·h})^p  \tag{2}</math>



参考文献：

[1] Marschner S ,  Shirley P . Fundamentals of computer graphics. 4th edition.[J]. World Scientific Publishers Singapore, 2009, 9(1):83.
