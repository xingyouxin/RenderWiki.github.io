## 简介

顾名思义，参与介质指参与光传输的介质。下面简单介绍参与介质，然后主要介绍光在参与介质中发生的四种交互作用及相关参数。

整体参考[1]。

## 1.Participating Media

### 1.1定义

参与介质是用来描述充满颗粒（如杂质、悬浮的微小粒子等）的体积（volume），例如水、牛奶、雾、蒸汽甚至是空气等。光线在这些介质中传播时，其中的颗粒会与光进行相互作用。光在其中从介质中的颗粒上进行反弹从而产生散射（scattering），或者被介质吸收（absorption）。

### 1.2分类

根据参与介质中颗粒的密度分布是否均匀，可以分为各向同性的和各向异性的。如果介质内部颗粒分布均匀，被称为各向同性的（homogeneous），如空气和水；当颗粒分布随空间位置的变化而变化时，这种不均匀的介质就是各向异性的（heterogeneous），例如云和蒸汽。

### 1.3光在介质中发生的相互作用

<div align=center>![光在介质中发生的四种相互作用](https://renderwiki.github.io/ImageResources/Participating Media/光在介质中发生的四种相互作用.png)</div>

<center>图1 光在介质中发生的四种相互作用 </center>

影响光在参与介质中传播的事件可以总结为以下四种类型。如图所示：

- 吸收（Absorption）：光子被介质中的物质吸收，从而转化为热或其他形式的能量的过程，其是有关吸收系数![](http://latex.codecogs.com/svg.latex?\sigma_{a})的函数。（清华公式：<math>\sigma_{a}</math>)
- 向外散射（Out-scattering）：光子通过与介质中的颗粒发生反弹从而向其他方向散射出去的过程，其是有关散射系数![](http://latex.codecogs.com/svg.latex?\sigma_{s})来描述。（<math>\sigma_{s}</math>)
- 自发光（Emission）：光从其他形式的能量转化为可见光，如火的黑体辐射。
- 向内散射（In-scattering）：来自四面八方的光子在与介质中的颗粒发生散射时，可以散射到当前光路中，并对这一光路的最终辐射度做出贡献。

### 1.4相关参数

<div align=center>![有关散射和参与介质的术语](https://renderwiki.github.io/ImageResources/Participating Media/有关散射和参与介质的术语.png)</div>

<center>图2 有关散射和参与介质的术语 </center>

吸收和向外散射会导致该光路上的光子减少， 通常可以将其合并为消光系数![](http://latex.codecogs.com/svg.latex?\sigma_{t}=\sigma_{a}+\sigma_{s})。（<math>\sigma_{t}=\sigma_{a}+\sigma_{s}</math>)

散射系数和吸收系数决定了介质的反照率ρ，定义为![](http://latex.codecogs.com/svg.latex?\rho=\frac{\sigma_{s}}{\sigma_{s}+\sigma_{a}}=\frac{\sigma_{s}}{\sigma_{t}})（<math>\rho=\frac{\sigma_{s}}{\sigma_{s}+\sigma_{a}}=\frac{\sigma_{s}}{\sigma_{t}}</math>)

![](http://latex.codecogs.com/svg.latex?\rho)代表了介质中散射相对于吸收在每个可见光谱范围内的重要性，也就是介质的整体反射率，其值在[0, 1]范围内。越接近0表示更多的光被吸收，从而导致介质浑浊，如黑暗的废气烟雾。反之越接近1表示大部分光线被散射，导致介质较亮，如空气、云或地球的大气层。（<math>\rho</math>)

现实世界中的参与介质的系数已经被测量和公布[2]。例如牛奶具有很高的散射系数且反照率![](http://latex.codecogs.com/svg.latex?\rho>0.999)，产生浑浊和呈现出不透明的白色。而红酒具有极高的吸收率，几乎没有散射，从而呈现出半透明和彩色的外观。（<math>\rho>0.999</math>)

这些参数和之前光在介质中的相互作用实际上都与光的波长有关。即在一个特定的参与介质中，不同频率的光会以不同的概率被吸收或散射。所以理论上应该在渲染时使用光谱值，但在实时渲染中（除了少数例外[3]），我们使用RGB值来提高渲染效率。

### 1.5 相函数
参与介质由许多直径不一的粒子构成，粒子尺寸的分布会影响光向特定方向散射的概率。相函数从宏观上描述了光向各个方向散射的概率分布，如图x所示，相函数的参数![](http://latex.codecogs.com/svg.latex?\theta)为光前进方向（蓝色矢量）和散射方向（绿色矢量）之间的夹角，在图片中相机B方向上相函数值大于相机A方向上的相函数值，因此光更可能散射到B方向，B方向上的radiance比A方向上的radiance更强。

（此处应有一张图）

根据粒子尺寸和波长的相对大小![](http://latex.codecogs.com/svg.latex?s_p)值，介质的散射行为会不同：

![](http://latex.codecogs.com/svg.latex?s_p=\frac{2\pi r}{\lambda})

其中![](http://latex.codecogs.com/svg.latex?r)为粒子直径，![](http://latex.codecogs.com/svg.latex?\lambda)为光的波长。

* 当![](http://latex.codecogs.com/svg.latex?s_p\ll1)时，发生的是Rayleigh散射（如空气）。
* 当![](http://latex.codecogs.com/svg.latex?s_p\approx1)时，发生的是Mie散射。
* 当![](http://latex.codecogs.com/svg.latex?s_p\gg1)时，发生的是几何散射（衍射、折射等，这里不做介绍）。

#### 相函数的性质

由于相函数是一个概率密度函数，他在球面上的积分为1，这也保证了能量守恒。

下面分别介绍这些散射和其对应的相函数。

#### 1.5.1均匀散射

理想下的各向同性散射，其相函数为最简单的相函数：

![](http://latex.codecogs.com/svg.latex?p(\theta)=\frac{1}{4\pi})

在这种理想情况下，光会向各个方向均匀散射。

#### 1.5.2Rayleigh散射

Rayleigh散射描述的是空气的散射情况，其散射和波长有关，散射系数![](http://latex.codecogs.com/svg.latex?\sigma_s)和波长的四次方成反比：

![](http://latex.codecogs.com/svg.latex?\sigma_{s}\left(\lambda\right)\propto\frac{1}{\lambda^{4}})

这一关系表明，波长较短的蓝光和紫光的散射量会比波长较长的红光散射量更大，用RGB三通道可表示为![](http://latex.codecogs.com/svg.latex?\sigma_s=(0.490, 1.017, 2.339))，该散射系数归一化到亮度为1，可以根据需要进行缩放。

（此处应有一张图）

而其相函数和波长无关，主要有两个lobe，如图x所示，根据散射方向分别为前向散射和后向散射，其函数为：

![](http://latex.codecogs.com/svg.latex?p(\theta)=\frac{3}{16 \pi}\left(1+\cos ^{2} \theta\right))

#### 1.5.3Mie散射
Mie散射描述的是粒子尺寸与光波长相当时介质的散射情况，其散射和波长无关，特征是具有很强的方向性。其相函数具有一个明显的方向性lobe，形式比较复杂因此计算非常昂贵。

（这里应有一张图）

Mie散射常用的相函数为Henyey-Greenstein（HG）函数，通常用在烟雾或灰尘类型的强方向性参与介质中，其公式为：

![](http://latex.codecogs.com/svg.latex?p_{h g}(\theta, g)=\frac{1-g^{2}}{4 \pi\left(1+g^{2}-2 g \cos \theta\right)^{1.5}})

其中参数![](http://latex.codecogs.com/svg.latex?g\in[-1, 1])用来控制散射的方向，当![](http://latex.codecogs.com/svg.latex?g=0)时散射退化为均匀散射，当![](http://latex.codecogs.com/svg.latex?g\gt0)时主要向前散射，反之当![](http://latex.codecogs.com/svg.latex?g\lt0)时主要向后散射。

另一个常用的相函数为HG相函数的近似，被称为Schlick相函数：

![](http://latex.codecogs.com/svg.latex?p(\theta, k)=\frac{1-k^{2}}{4 \pi(1+k \cos \theta)^{2}}, \quad) where ![](http://latex.codecogs.com/svg.latex?\quad k \approx 1.55 g-0.55 g^{3})

如图x所示，Schlick函数与HG函数非常接近，但其形式更简单计算的更快，在实际应用中效果非常接近。


参考文献：

[1] Tomas Akenine-Mller, Eric Haines, and Naty Hoffman. 2018. Real-Time Rendering, Fourth Edition (4th. ed.). A. K. Peters, Ltd., USA.

[2]Narasimhan, Srinivasa G., Mohit Gupta, Craig Donner, Ravi Ramamoorthi, Shree K. Nayar, and Henrik Wann Jensen, “Acquiring Scattering Properties of Participating Media by Dilution,” ACM Transactions on Graphics (SIGGRAPH 2006), vol. 25, no. 3, pp. 1003–1012, Aug. 2006. 

[3]Hanika, Johannes, “Manuka: Weta Digital’s Spectral Renderer,” SIGGRAPH Path Tracing in Production course, Aug. 2017.