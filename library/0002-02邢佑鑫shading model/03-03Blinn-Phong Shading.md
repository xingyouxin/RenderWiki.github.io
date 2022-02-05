## 简介

Lambertian着色与视线方向无关：从任何方向观察着色表面，其着色结果都相同。很多真实表面可以显示出一定程度的光泽感，产生高光或者镜面反射现象，而且这些反射会随着视点的变化而移动。Lambertian着色不会产生任何高光，并且会导致非常暗，类似石灰白般的外观，因此许多着色模型为Lambertian着色添加了镜面反射部分，将Lambertian着色视为漫反射部分。

Phong（Phong，1975）提出了一种非常简单且广泛使用的镜面高光模型，后来由Blinn（J. F. Blinn，1976）更新为今天最常用的形式，即Blinn-Phong着色模型。

## 1.Blinn-Phong着色

### 1.1基本概念

Blinn-Phong着色的基本思路是在视线向量$v$和光线向量$l$与法线对称时，即当发生镜面反射时产生最亮的反射。当向量偏离镜像位置时，反射将会平滑减小。

半向量$h$（平分$v$和$l$之间夹角的向量）与表面法线可以被用来比较判断视线向量$v$和光线向量$l$与法线接近镜面对称的程度，如图1.1。

![Blinn-Phong着色示意图](https://renderwiki.github.io/ImageResources/shading model/Blinn-Phong着色示意图.png)

<center/>图1.1 Blinn-Phong着色示意图<center>

如果半向量接近表面法线，镜面反射部分应该变得非常亮；如果半向量远离法线，则镜面反射部分会变暗。通过计算$h$和$n$之间的点积来对其进行评估。由于两个向量都是单位向量，所以当两个向量相等时，$n·h$将达到最大值1。对点乘结果取$p$次幂（$p$大于1）来确保这一结果更快衰减。该幂指数或者说Phong指数用于控制表面的光泽程度。

几种具有代表性的$p$值：

- 10——蛋壳漆；
- 100——轻微有光泽；
- 1000——非常有光泽；
- 10,000——近乎镜面。

### 1.2半向量的计算方法

半向量很容易计算：由于$v​$和$l​$的长度相同，因此对它们求和恰好对应一个将它们之间角度均分的向量，只需要对二者之和进行归一化便可以产生$h​$。

### 1.3公式表述

综上所述，Blinn-Phong着色模型如下所示：

​                                   $$\pmb{h}=\frac{(\pmb{v+l})}{||\pmb{v+l}||}  \tag{1}$$

​                                      $$L=k_dImax(0, \pmb{n·l})+k_sImax(0, \pmb{n·h})^p  \tag{2}$$

其中$k_s$是表面的镜面系数或镜面反射颜色。

参考文献：

[1] Marschner S ,  Shirley P . Fundamentals of computer graphics. 4th edition.[J]. World Scientific Publishers Singapore, 2009, 9(1):83.