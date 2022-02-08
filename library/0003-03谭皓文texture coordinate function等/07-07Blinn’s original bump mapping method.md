## 简介

凹凸贴图（Bump Mapping）是一系列小尺度细节表示技术的集合，它最早是由Jim Blinn提出的。这种技术都是在逐像素着色过程中实现的，不同的凹凸贴图技术的区别在于如何表示几何表面的细节特征，它给光滑的平面在不改变几何形状也就是不增加顶点的情况下，增加凹凸不平的视觉效果，如褶皱、波浪等效果。由于凹凸贴图在不增加额外几何信息的同时，能够增强渲染表面的细节效果，因此得到了广泛的应用。

## 1 凹凸贴图(Bump mapping)
### 1.1 原始凹凸贴图方法（Bllin Bump mapping method）

Blinn在1978年提出了在纹理中编码细观尺度特征（meso-features）的想法。他观察到，在着色的过程中使用稍微扰动的表面法线代替真实的表面，则几何表面具有小范围的细节特征，他将描述表面法线扰动的数据存储在数组中。这种方法的关键思想是：不适用纹理来改变渲染方程中的颜色分量，而是访问纹理来修改表面法线，模型实际的几何并没有改变。

对于凹凸贴图技术来说，法线必须相对于某个坐标参考系进行变换，因此将切线空间基底（Tangent-Space Basis）存储在每个顶点上，次参考系用于将光源方向转换到局部坐标系下，从而计算扰动法线后的效果，除了顶点法线外，在多边形表面上应用了法线贴图的情况下，还存储了切向量(Tangent)以及副切向量(Bitangent)。切向量和副切向量表示法线贴图本身在对象空间中的轴，因为目标是将光照方向转换为相对于贴图的光照。

<div align=center>![figure-1](https://renderwiki.github.io/ImageResources/Texture_Coordinate_Function/figure_10.png)</div>

<center>图1.1 展示了一个球形三角形，其切线框显示在每个角上。形状像圆球和圆环的，具有自然的切线空间基底。</center>

法线 ![](http://latex.codecogs.com/svg.latex?n)，切线 ![](http://latex.codecogs.com/svg.latex?t) 和副切线 ![](http://latex.codecogs.com/svg.latex?b) 这三个向量可以形成一个基本矩阵，这个矩阵通常也被称为TBN矩阵，对于给定顶点，它将光源方向从世界坐标系转化为切线空间。

![](http://latex.codecogs.com/svg.latex?\begin{pmatrix}
t_{x}&t_{y}&t_{z}&0\\
b_{x}&b_{y}&b_{z}&0\\
n_{x}&n_{y}&n_{z}&0\\
0&0&0&1\\
\end{pmatrix})

<math>\begin{pmatrix}
t_{x}&t_{y}&t_{z}&0\\
b_{x}&b_{y}&b_{z}&0\\
n_{x}&n_{y}&n_{z}&0\\
0&0&0&1\\
\end{pmatrix} \tag{1}</math>

Blinn最初的凹凸贴图方法是在纹理的每个纹素中存储的是两个有符号的值 ![](http://latex.codecogs.com/svg.latex?b_{u}) 和 ![](http://latex.codecogs.com/svg.latex?b_{v})。这两个值对应于沿 ![](http://latex.codecogs.com/svg.latex?u) 和 ![](http://latex.codecogs.com/svg.latex?v) 轴改变法线的值，描述了曲面在该点面向哪个方向（如下图所示），这种类型的凹凸贴图被称为偏移向量凹凸贴图（Offset Vector Bump Mapping）或者偏移贴图（Offset Map）。

<div align=center>![figure-2](https://renderwiki.github.io/ImageResources/Texture_Coordinate_Function/figure_11.png)</div>

<center>在左图中，法向量 ![](http://latex.codecogs.com/svg.latex?n) 通过从偏移贴图中获取的 ![](http://latex.codecogs.com/svg.latex?(b_{u},b_{v})) 在 ![](http://latex.codecogs.com/svg.latex?u v)两个方向上进行修改，得到未归一化的法线 ![](http://latex.codecogs.com/svg.latex?n^{'})。右图中展示了一个高度场中对着色法线进行插值从而得到更加平滑的结果。</center>

### 1.2 使用高度场的凹凸贴图
另外一种凹凸贴图的方法是使用高度场（Heightfield）来修改表面法线的值，高度图（Height Map）是单通道的纹理，纹素中存储的是高度，因此在纹理中，白色表示的是高度区，黑色表示的是低度区，在原始法向量的基础上添加一个扰动，这个扰动可以从这个高度图中获取，从而得到一个修正后的法向量，从而在光照计算的时候，增加光影的变化，从而产生凹凸不平的视觉效果。

通过高度图可以计算当前像素和周围像素的高度差，从而获取高度的梯度信息从而进行法线的扰动。通过高度图应用凹凸贴图的过程可以通过以下伪代码表示，其中(u,v)表示的是参数空间下的纹理坐标，切线空间中的切向量（Tangent）方向为T，负法线（Bitangent）方向为B，高度贴图中对应的高度值为Height(u,v)，高度贴图的大小为Heightmap.xy，该几何点处原先的法线为normal，扰动后得到的新的法线值为new_normal。

```
du= 1/Heightmap.x; // u,v 方向上的微分
dv=1/Heightmap.y;
u_gradient=Height(u-du,v)-Height(u+du,v); // 获取u方向上的纹理梯度值
v_gradient=Height(u,v-dv)-Height(u,v+dv); // 获取v方向上的纹理梯度值
new_normal=normal+T*u_gradient+B*v_gradient; // 扰动后的新的法线值
```

<div align=center>![figure-3](https://renderwiki.github.io/ImageResources/Texture_Coordinate_Function/figure_12.png)</div>

<center>图1.3 使用左边的高度图作为凹凸贴图，并应用在球形物体上得到右图的渲染结果。</center>

参考文献：

[1] Marschner S ,  Shirley P . Fundamentals of computer graphics. 4th edition.[J]. World Scientific Publishers Singapore, 2009, 9(1):82.
[2] Tomas Akenine-Moller, Eric Haines, Naty Hoffman .Real-Time Rendering 4rd.

