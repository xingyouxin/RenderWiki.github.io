## 简介

体积渲染不仅要考虑光线与物体表面的交互，还要考虑与空间中大量微粒的交互。本文介绍了体积渲染的积分方程形式的光线传输方程。

整体参考自[1]

## 1.Volumatric Rendering Equation

### 1.1体积渲染方程

假设光线来自于点光源，根据参与介质中光的四种相互作用，评估散射涉及的计算：

![](http://latex.codecogs.com/svg.latex?L_{i}(\mathbf{c},-\mathbf{v})=T_{r}(\mathbf{c}, \mathbf{p}) L_{o}(\mathbf{p}, \mathbf{v})+\int_{t=0}^{\|\mathbf{p}-\mathbf{c}\|} T_{r}(\mathbf{c}, \mathbf{c}-\mathbf{v} t) L_{\text {scat }}(\mathbf{c}-\mathbf{v} t, \mathbf{v}) \sigma_{s} d t)

其中![](http://latex.codecogs.com/svg.latex?T_{r}(\mathbf{c}, \mathbf{x}))是给定点![](http://latex.codecogs.com/svg.latex?\mathbf{x})和相机位置![](http://latex.codecogs.com/svg.latex?\mathbf{c})之间的透射率，![](http://latex.codecogs.com/svg.latex?L_{\text {scat }}(\mathbf{x}, \mathbf{v}))是沿视图方向上给定点![](http://latex.codecogs.com/svg.latex?\mathbf{x})的散射光。计算的不同部分如图1所示。

<div align=center>![](https://renderwiki.github.io/ImageResources/Volumatric Rendering Equation/来自点光源的单次散射积分的说明.png)</div>

<center>图1 来自点光源的单次散射积分的说明。 </center>

### 1.2Transmittance

#### 1.2.1定义

透射率（transmittance）![](http://latex.codecogs.com/svg.latex?T_{r})表示通过介质内一定距离的光的比率，这种关系也被成为比尔-朗伯定律(Beer-Lambert Law)：

![](http://latex.codecogs.com/svg.latex?T_{r}\left(\mathbf{x}_{a}, \mathbf{x}_{b}\right)=e^{-\tau}, \quad \text { where } \quad \tau=\int_{\mathbf{x}=\mathbf{x}_{a}}^{\mathbf{x}_{b}} \sigma_{t}(\mathbf{x})\|d \mathbf{x}\|)

其中光学深度![](http://latex.codecogs.com/svg.latex?\tau)是无单位的，表示光的衰减量。消光系数或穿过的距离越大，光学深度就越大，能够通过介质的光就越少。一个光学深度（![](http://latex.codecogs.com/svg.latex?\tau=1))将去除大约60%的光。

当厚度发生变化时，上式可以简化为![](http://latex.codecogs.com/svg.latex?T_{r}=e^{-d \sigma_{t}})，其中![](http://latex.codecogs.com/svg.latex?d)是通过材质体的距离，物理消光参数![](http://latex.codecogs.com/svg.latex?\sigma_{t})表示光通过介质时的衰减速度。例如，在RGB中![](http://latex.codecogs.com/svg.latex?\sigma_{t}=(0.5,1,2))，那么通过深度![](http://latex.codecogs.com/svg.latex?d=1m)的光将为![](http://latex.codecogs.com/svg.latex?T_{r}=e^{-d \sigma_{t}} \approx(0.61,0.37,0.14))

为了让艺术家直观地进行创作，Bavoil[2]将目标颜色![](http://latex.codecogs.com/svg.latex?t_{c})设定为某个给定距离![](http://latex.codecogs.com/svg.latex?d)的透光率量，那么消光率![](http://latex.codecogs.com/svg.latex?\sigma_{t})可以恢复为

![](http://latex.codecogs.com/svg.latex?\sigma_{t}=\frac{-\log \left(\mathbf{t}_{c}\right)}{d})

#### 1.2.2应用场景

透射率主要应用于：

​        i.     不透明表面的出射辐射度![](http://latex.codecogs.com/svg.latex?L_{o}(\mathbf{p}, \mathbf{v}))

​        ii.     由向内散射事件产生的辐射度![](http://latex.codecogs.com/svg.latex?L_{\text {scat }}(\mathbf{x}, \mathbf{v}))

​         iii.      从一个散射事件到光源的每个路径

从视觉上看，(i)会导致表面上一些雾状的遮挡；(ii)会导致散射光的遮蔽，提供了另一种有关介质厚度的视觉提示；以及(iii)会导致参与介质的体积自阴影。

#### 1.2.3半透明表面距离![](http://latex.codecogs.com/svg.latex?d)的计算

对于半透明表面（表面由一层半透明材质薄层组成），沿着它的法线或切线看，将导致不同程度的背景遮挡。即路径长度会随角度发生变化。Drobot[3]提出了这样一种方法，透射率![](http://latex.codecogs.com/svg.latex?T_{r})被评估为

![](http://latex.codecogs.com/svg.latex?\mathbf{T}_{r}=e^{-\boldsymbol{\sigma}_{t} d}, \quad \text { where } \quad d=\frac{t}{\max (0.001, \mathbf{n} \cdot \mathbf{v})})

<div align=center>![](https://renderwiki.github.io/ImageResources/Volumatric Rendering Equation/半透明表面距离d的计算.png)</div>

<center>图2 半透明表面距离d的计算示意图 </center>

一个常见的方法是渲染视图射线离开体积的表面，这个表面可以是水晶球的背面或者是海底（即水的尽头）。这个表面的深度或位置被存储起来。然后渲染进入体积的表面。在着色器中访问存储的深度或位置，然后计算其与当前表面的距离，从而计算透射率。该方法只适用于体积是封闭的且是凸的，即它在每个像素都有一个入射点和出射点。

对于更复杂的模型，例如玻璃雕塑或其他具有凹面的表面，对于光线在每个像素可能跨过两个或更多层。如果使用深度peeling，可以按照精确的从后到前的顺序渲染体积表面。当每个正面被渲染时，通过体积的距离被计算出来并用于计算透射率。依次计算可以得到最终的透射率。如果所有体积都是由相同浓度的相同材质制成的且表面没有反射，可以直接在最后用距离之和计算一次透射率。

参考文献：

[1] Tomas Akenine-Mller, Eric Haines, and Naty Hoffman. 2018. Real-Time Rendering, Fourth Edition (4th. ed.). A. K. Peters, Ltd., USA.

[2] Bavoil, Louis, Steven P. Callahan, Aaron Lefohn, Jo˜ao L. D. Comba, and Cl´audio T. Silva, “Multi-Fragment Effects on the GPU Using the k-Buffer,” in Proceedings of the 2007 Symposium on Interactive 3D Graphics and Games, ACM, pp. 97–104, Apr.–May 2007.

[3] Drobot, Micha l, “Practical Multilayered Materials in Call of Duty Infinite Warfare,” SIGGRAPHPhysically Based Shading in Theory and Practice course, Aug. 2017. 
