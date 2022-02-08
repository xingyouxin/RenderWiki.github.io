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

- 吸收（Absorption）：光子被介质中的物质吸收，从而转化为热或其他形式的能量的过程，其是有关吸收系数![](http://latex.codecogs.com/svg.latex?\sigma_{a})的函数。
- 向外散射（Out-scattering）：光子通过与介质中的颗粒发生反弹从而向其他方向散射出去的过程，其是有关散射系数![](http://latex.codecogs.com/svg.latex?\sigma_{s})来描述。
- 自发光（Emission）：光从其他形式的能量转化为可见光，如火的黑体辐射。
- 向内散射（In-scattering）：来自四面八方的光子在与介质中的颗粒发生散射时，可以散射到当前光路中，并对这一光路的最终辐射度做出贡献。

### 1.4相关参数

<div align=center>![有关散射和参与介质的术语](https://renderwiki.github.io/ImageResources/Participating Media/有关散射和参与介质的术语.png)</div>

<center>图2 有关散射和参与介质的术语 </center>

吸收和向外散射会导致该光路上的光子减少， 通常可以将其合并为消光系数![](http://latex.codecogs.com/svg.latex?\sigma_{t}=\sigma_{a}+\sigma_{s})。

散射系数和吸收系数决定了介质的反照率ρ，定义为![](http://latex.codecogs.com/svg.latex?\rho=\frac{\sigma_{s}}{\sigma_{s}+\sigma_{a}}=\frac{\sigma_{s}}{\sigma_{t}})

![](http://latex.codecogs.com/svg.latex?\rho)代表了介质中散射相对于吸收在每个可见光谱范围内的重要性，也就是介质的整体反射率，其值在[0, 1]范围内。越接近0表示更多的光被吸收，从而导致介质浑浊，如黑暗的废气烟雾。反之越接近1表示大部分光线被散射，导致介质较亮，如空气、云或地球的大气层。

现实世界中的参与介质的系数已经被测量和公布[2]。例如牛奶具有很高的散射系数且反照率![](http://latex.codecogs.com/svg.latex?\rho>0.999)，产生浑浊和呈现出不透明的白色。而红酒具有极高的吸收率，几乎没有散射，从而呈现出半透明和彩色的外观。

这些参数和之前光在介质中的相互作用实际上都与光的波长有关。即在一个特定的参与介质中，不同频率的光会以不同的概率被吸收或散射。所以理论上应该在渲染时使用光谱值，但在实时渲染中（除了少数例外[3]），我们使用RGB值来提高渲染效率。

参考文献：

[1] Tomas Akenine-Mller, Eric Haines, and Naty Hoffman. 2018. Real-Time Rendering, Fourth Edition (4th. ed.). A. K. Peters, Ltd., USA.

[2]Narasimhan, Srinivasa G., Mohit Gupta, Craig Donner, Ravi Ramamoorthi, Shree K. Nayar, and Henrik Wann Jensen, “Acquiring Scattering Properties of Participating Media by Dilution,” ACM Transactions on Graphics (SIGGRAPH 2006), vol. 25, no. 3, pp. 1003–1012, Aug. 2006. 

[3]Hanika, Johannes, “Manuka: Weta Digital’s Spectral Renderer,” SIGGRAPH Path Tracing in Production course, Aug. 2017.