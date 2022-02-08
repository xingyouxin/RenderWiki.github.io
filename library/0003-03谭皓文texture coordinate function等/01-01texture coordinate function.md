## 简介

当人们尝试还原真实世界时，意识到几乎任何物质的表面都是有特征的。木材长有纹理、皮肤会有褶皱、布料会有编制结构，即使是光滑的塑料也会有凹凸不平的痕迹。在图形学中，这些特征被称为空间上表面属性的变化（Spatially Varying Surface Properties），也就是说物体表面的属性在每一处地方都发生着变化，但却没有真正意义上改变物体的几何形状，考虑到这些因素，建模系统或者渲染系统提供了纹理映射（Texture Mapping）方法用于解决这一问题：通过被称为纹理贴图的一组或一张图像存储想要在物体表面展示的细节特征，然后通过数学处理，把这些纹理图像中存储的细节特征映射到材质表面上。

## 1.纹理坐标函数（Texture Coordinate Functions）

首先我们考虑这样一个简单的纹理映射的应用场景（如下图所示），我们有一个球的场景，我们希望球的漫反射颜色由一张世界地图样式的纹理图像控制。不管是我们在光线追踪中用于计算射线表面交点的颜色，还是光栅化中计算光栅化器生成片段/像素的颜色，都需要知道着色点处的纹理颜色，以便作为着色模型中的漫反射颜色。

为了得到这个纹理值，着色器程序需要执行纹理查询（Texture Lookup）：它计算出纹理图像的在当前纹理空间（Texture Space）中的坐标值，然后读出纹理图像在这个坐标处的颜色值，也就是纹理样品（Texture Sample），然后被应用在着色之中。图像空间（Image Space）的球场景中每个着色点都执行了在不同纹理坐标位置下的纹理查询，从而使得图像出现不同的颜色特征。

在着色器代码中，着色器查询几何表面在纹理中的位置，对于我们想要进行着色的每个着色点，这个纹理都要能够回应这个查询请求。因此纹理映射中的一个关键因素是：需要一个映射函数，使得从三维几何表面映射到二维纹理中，这样就可以轻松地进行像素的着色计算，这个映射函数也就是纹理坐标函数（Texture Coordinate Functions）。

<div align=center>![figure-1](https://renderwiki.github.io/ImageResources/Texture_Coordinate_Function/figure_1.png)</div>

<center>图1.1 纹理坐标函数![](http://latex.codecogs.com/svg.latex?\phi)是映射世界空间表面![](http://latex.codecogs.com/svg.latex?S)和纹理空间![](http://latex.codecogs.com/svg.latex?T)。恰当的![](http://latex.codecogs.com/svg.latex?\phi)是所有纹理映射应用的基础。</center>

从图1.1可以看出纹理坐标函数![](http://latex.codecogs.com/svg.latex?\phi)是一个世界空间的三维几何表面![](http://latex.codecogs.com/svg.latex?S)和二维纹理空间![](http://latex.codecogs.com/svg.latex?T)之间的映射关系。我们使用以下数学表达形式表示它：

![](http://latex.codecogs.com/svg.latex?\phi: S \rightarrow T)
<math>\phi: S \rightarrow T \tag{1}</math>

![](http://latex.codecogs.com/svg.latex?:(x,y,z) \rightarrow (u,v))
<math>:(x,y,z) \rightarrow (u,v) \tag{2}</math>


集合T被称为纹理空间，通常只是一个包含图像的矩形，通过 ![](http://latex.codecogs.com/svg.latex?u,v) 且 ![](http://latex.codecogs.com/svg.latex?(u,v)\in[0,1]^{2})表示这个矩形中的坐标点。图1.1中![](http://latex.codecogs.com/svg.latex?\pi)被称为观察投影，它是用来映射三维场景中的点和渲染图像上的点。![](http://latex.codecogs.com/svg.latex?\pi) 和 ![](http://latex.codecogs.com/svg.latex?\phi) 都是三维空间到二维空间的映射，但![](http://latex.codecogs.com/svg.latex?\pi)几乎总是透视投影或者正交投影，而![](http://latex.codecogs.com/svg.latex?\phi) 则可以采用不同的方式，同时场景中的每个几何对象都可以采用不同的纹理坐标函数。

对于图2中的地板场景，如果地板上的每个几何点的 ![](http://latex.codecogs.com/svg.latex?(x,y,z)_{floor})的 ![](http://latex.codecogs.com/svg.latex?z) 值相同，那么可以采用如下的映射方式,其中 ![](http://latex.codecogs.com/svg.latex?a,b) 是放缩因子。

![](http://latex.codecogs.com/svg.latex?u=ax)
<math>u=ax \tag{3}</math>

![](http://latex.codecogs.com/svg.latex?v=by)
<math>v=by \tag{4}</math>

<div align=center>![figure-2](https://renderwiki.github.io/ImageResources/Texture_Coordinate_Function/figure_2.png)</div>

<center>图1.2 直接使用几何坐标作为简单的纹理坐标函数，得到的渲染结果。</center>

### 1.1纹理坐标函数设计不合理导致的错误

设计一个好的纹理坐标函数![](http://latex.codecogs.com/svg.latex?\phi)是获取好的纹理映射结果的关键所在。图3中将一个高分辨率的纹理映射到几何平面上，并渲染得到低分辨率的图像，可以看到很明显的锯齿失真现象。

<div align=center>![figure-3](https://renderwiki.github.io/ImageResources/Texture_Coordinate_Function/figure_3.png)</div>

<center>图1.3 对于一个大的平面，使用和图1.2中相同的纹理坐标函数映射高分辨率的纹理，渲染低分率图像，结果中出现很明显的失真走样现象。</center>

### 1.2设计纹理坐标函数考虑的因素

纹理坐标函数的定义过程中需要考虑多种要素，它包括：

- 双射性（Bijectivity）：![](http://latex.codecogs.com/svg.latex?\phi) 应该使得几何表面上每个点被映射到纹理空间上的不同点。如果想将纹理在几何表面重复出现，这种多对一的映射方式也是有意义的。 
- 尺寸变形（Size Distortion）：几何表面上的相邻两个点映射到纹理空间上，也应该有相同的距离，也就是说 ![](http://latex.codecogs.com/svg.latex?\phi) 的导数应该变化不大。
- 形状变形（Shape Distortion）：纹理不应该出现很严重的形状变形，渲染在表面的一个小圆映射到纹理空间一个合理的圆形形状，而不是一个被压扁或被拉长的形状，也就是说，![](http://latex.codecogs.com/svg.latex?\phi) 在不同方向上的导数应该相差不大。
- 连续性（Continuity）：不应该有过多的接缝，几何表面上的相邻点应该映射到纹理空间上的相邻点，也就说说 ![](http://latex.codecogs.com/svg.latex?\phi) 应该是连续的，或有较少的不连续情况。

实际应用过程中，纹理坐标函数通过两种方式定义：
- 利用三维空间上的空间坐标值通过几何计算得到。
- 对于模型网格，通过在顶点上存储纹理坐标值，存储在模型文件中。

参考文献：

[1] Marschner S ,  Shirley P . Fundamentals of computer graphics. 4th edition.[J]. World Scientific Publishers Singapore, 2009, 9(1):243-248.