## 简介
最简单的着色模型基于$Lambert$在18世纪做的一项观察实验：落在表面区域的光线的能量强弱取决于表面和光线之间的夹角大小。

# 1Lambertian Shading

### 1.1基本概念
直接朝向光源的表面将会获得最大程度的光线照射；与光线方向相切（或者背朝光源）的表面不会受到任何光照；处于上述两种情况之间的光照强度与表面法线和光线之间夹角θ的余弦成正比，如图xxx。

<div align=center>![Lambertian着色示意图](https://renderwiki.github.io/ImageResources/shading model/Lambertian着色示意图.png)</div>

<center>图1.1 Lambertian着色示意图</center>

### 1.2公式表达
这就引出了![](http://latex.codecogs.com/svg.latex?Lambertian)![](http://latex.codecogs.com/svg.latex?shading)![](http://latex.codecogs.com/svg.latex?model)：

![](http://latex.codecogs.com/svg.latex?L=k_dImax(0, nl))

<math>L=k_dImax(0, \pmb{n·l}) \tag{1}</math>

其中，![](http://latex.codecogs.com/svg.latex?L)是像素颜色；![](http://latex.codecogs.com/svg.latex?k_d)是漫反射系数![](http://latex.codecogs.com/svg.latex?diffuse)![](http://latex.codecogs.com/svg.latex?coefficient)，或为物体表面颜色；![](http://latex.codecogs.com/svg.latex?I)是光源的强度（真实世界光源的强度将会随着距离平方下降，这比在简单渲染器中麻烦的多）。

因为![](http://latex.codecogs.com/svg.latex?n)和![](http://latex.codecogs.com/svg.latex?l)是单位矢量，所以我们可以使用n·l作为cosθ的简化写法。

该方程分别适用于RGB三个颜色通道，因此像素值的红色分量对应红色漫反射分量，红色光源强度以及点积乘积，其它颜色分量也是如此。

向量$l$是通过光源位置减去光线与表面交点位置来计算的。不要忘记![](http://latex.codecogs.com/svg.latex?v)、![](http://latex.codecogs.com/svg.latex?l)和![](http://latex.codecogs.com/svg.latex?n)都必须是单位矢量，未规范化这些向量是着色计算中非常常见的错误。



参考文献：

[1] Marschner S ,  Shirley P . Fundamentals of computer graphics. 4th edition.[J]. World Scientific Publishers Singapore, 2009, 9(1):82.