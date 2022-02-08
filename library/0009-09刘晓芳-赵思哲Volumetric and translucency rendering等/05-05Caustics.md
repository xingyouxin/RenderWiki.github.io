## 简介

当光线遇到透明物体的弯曲表面会形成焦散，许多方法提供了此类现象的近似计算来实现实时渲染。

整体参考自[1]

## 1.焦散（Caustics）

### 1.1基本概念

#### 1.1.1定义

焦散是透明物体的弯曲表面使得光线偏离其直线路径的视觉结。光线会从某些区域失焦产生阴影，同时会在其他区域聚焦，使这个区域的光线路径边的更加密集，也更亮。

一个典型的反射产生的焦散例子如图1所示，可以在咖啡杯内看到心形焦散。折射产生的焦散会更加明显，例如光通过玻璃杯形成的焦散现象。

<div align=center>![](https://renderwiki.github.io/ImageResources/Caustics/真实世界的焦散现象.png)</div>

<center>图1 真实世界的焦散现象 </center>

### 1.2渲染方法

#### 1.2.1图像空间技术

Wyman[2，3]提出了一种用于焦散渲染的图像空间技术。首先，它通过使用背景折射技术（background refraction technique）评估光子的位置以及通过透明物体正面和背面折射后的入射方向。其中纹理不是用来存储折射后的辐射度，而是用来存储场景中的交点位置、折射后的入射方向以及由于菲涅尔效应产生的透射率。每个纹理存储一个光子，然后以正确的强度发射回视图空间，有两种可能的方法来实现这一目标：

- 一种方法在视图空间或者光照空间以四倍频率散射光子，并进行高斯衰减。
- McGuire和Mara[4]提出了一种更简单的方法，根据透明表面的法线改变透射率。如果垂直于入射表面，则透射率更高，否则，由于Fresnel效应透射率更低。

#### 1.2.2水体焦散

光线经由弯曲的水面反射和折射也会出现焦散现象，无论是水面上还是水底，如图2。当光线汇聚时，光线会集中在不透明的表面上，并产生焦散。当在水面下方，汇聚的光路将在水体中变得可见，这就是光子穿过水粒子散射而产生的光轴（light shafts）。

<div align=center>![](https://renderwiki.github.io/ImageResources/Caustics/水中的焦散现象.png)</div>

<center>图2 水中的焦散现象(图片来自WebGL水的演示，由Evan Wallace [5]提供) </center>

为了从水面上生成焦散，可以应用离线生成的焦散动画纹理作为表面的光照贴图。许多游戏使用了这种方法，例如在CryEngine上运行的《孤岛危机3》。一个关卡中的水域是由水体（water volumes）制作的。水体的表面可以使用凹凸贴图纹理动画或物理模拟来实现。在水面上方和下方进行垂直投影时，凹凸贴图产生的法线可以用来从它们的方向映射到radiance贡献以产生焦散。

同样的，动态水面也可以用于水中的焦散渲染。Lanza[6]提出了一个两步法来生成光轴(light shafts)。首先，从光源的角度对光的位置和折射方向进行渲染，并保存到纹理中。然后，从水面开始，沿着视图中的折射方向将线条栅格化。通过累加混合法(additive blending)累积，最后进行后期模糊来模糊结果，以掩盖低数量的线条。

参考文献：

[1] Tomas Akenine-Mller, Eric Haines, and Naty Hoffman. 2018. Real-Time Rendering, Fourth Edition (4th. ed.). A. K. Peters, Ltd., USA.

[2] Wyman, Chris, “Interactive Refractions and Caustics Using Image-Space Techniques,” in Wolfgang Engel, ed., ShaderX5, Charles River Media, pp. 359–371, 2006.

[3] Wyman, Chris, “Hierarchical Caustic Maps,” in Proceedings of the 2008 Symposium on Interactive 3D Graphics and Games, ACM, pp. 163–172, Feb. 2008.

[4] McGuire, Morgan, and Michael Mara, “Phenomenological Transparency,” IEEE Transactions of Visualization and Computer Graphics, vol. 23, no.5, pp. 1465–1478, May 2017.

[5] Wallace, Evan, “Rendering Realtime Caustics in WebGL,” Medium blog, Jan. 7, 2016.

[6] Lanza, Stefano, “Animation and Rendering of Underwater God Rays,” in Wolfgang Engel, ed., ShaderX5, Charles River Media, pp. 315–327, 2006.

