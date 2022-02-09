## 简介

混合常用来绘制透明或半透明物体。

整体参考自[1]

## 1.Coverage

透明是物体（或物体的一部分）不光呈现出自身的颜色，同时还会将背后的物体的外观一定程度地呈现出来，例如可以透过彩色透明玻璃看到其背后的场景。透明物体可以是完全透明（背景颜色完全穿透）或者半透明的（背景颜色穿透的同时也显示自身颜色）。一个物体的覆盖率，也称为透明度，用α表示，通常设置为颜色向量的第四个元素。

## 2.Blending

假设输出颜色、表面辐射度和背景颜色分别为![](http://latex.codecogs.com/svg.latex?c_{o})、![](http://latex.codecogs.com/svg.latex?c_{s})和![](http://latex.codecogs.com/svg.latex?c_{b})，那么透明表面的混合公式为：

<div align=center>![](https://renderwiki.github.io/ImageResources/Coverage and Blending_noPictures/混合公式.svg)</div>

对于半透明表面（通常是指具有高吸收和低散射系数的材质），混合操作为：

<div align=center>![](https://renderwiki.github.io/ImageResources/Coverage and Blending_noPictures/半透明表面的混合公式.svg)</div>

其中![](http://latex.codecogs.com/svg.latex?c_{s})包含固体表面的镜面反射，即玻璃或凝胶。![](http://latex.codecogs.com/svg.latex?T_{r})是一个三通道的透射率颜色向量。

一般可以使用通用的混合操作来同时处理上述情况，其使用的混合函数是

<div align=center>![](https://renderwiki.github.io/ImageResources/Coverage and Blending_noPictures/两式混合公式.svg)</div>

参考文献：

[1] Tomas Akenine-Mller, Eric Haines, and Naty Hoffman. 2018. Real-Time Rendering, Fourth Edition (4th. ed.). A. K. Peters, Ltd., USA.
