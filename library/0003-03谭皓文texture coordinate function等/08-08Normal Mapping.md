## 简介

另外一种普遍应用的凹凸贴图技术是直接存储法线，从而被称为法线贴图，它和Bllin 原始的凹凸贴图方法得到的效果是一致的，不同的地方在于存储的格式以及在像素着色器中不同的计算方式。

## 1.法线贴图（Normal Mapping）

在法线贴图中，将 ![](http://latex.codecogs.com/svg.latex?(x,y,z)) 映射到区间 ![](http://latex.codecogs.com/svg.latex?[-1,1])。比如说对于一张单通道8位的贴图，0表示-1.0，而255表示1.0。 对于下图左边的法线贴图中某一个纹素 ![](http://latex.codecogs.com/svg.latex?[128,128,255]),在颜色上体现为淡蓝色，映射成法线后为 ![](http://latex.codecogs.com/svg.latex?[0,0,1]),也就是较为平坦的平面。

法线图最初是定义在世界空间系下的，但是在实践过程中却很少这么使用。使用法线贴图进行法线的扰动非常简单，对于每个着色的像素从纹理中检索出对应的法线，带入渲染方程中，从而计算出像素的正确着色，但是世界和物体空间的表示都将纹理绑定到特定方向的特定形状，从而限制了纹理的重复使用。

因此我们通常在切线空间上检索并扰动法线，这种方法能够允许最大程度的纹理重复使用。

<div align=center>![figure-1](https://renderwiki.github.io/ImageResources/Texture_Coordinate_Function/figure_13.png)</div>

<center>图1.1 使用左图的法线贴图，并应用在立方体上得到右图的渲染结果。</center>

使用法线贴图可以提供更高的真实感结果，如下图所示。

<div align=center>![figure-2](https://renderwiki.github.io/ImageResources/Texture_Coordinate_Function/figure_14.png)</div>

<center>图1.2 左上图：未使用法线贴图的渲染结果 左下图：使用右边两张法线贴图的渲染结果 右图：两种不同的法线贴图。</center>

参考文献：

[1] Marschner S ,  Shirley P . Fundamentals of computer graphics. 4th edition.[J]. World Scientific Publishers Singapore, 2009, 9(1):82.
[2] Tomas Akenine-Moller, Eric Haines, Naty Hoffman .Real-Time Rendering 4rd.

