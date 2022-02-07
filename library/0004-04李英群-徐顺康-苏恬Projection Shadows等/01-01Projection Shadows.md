# 简介
在投影阴影中，三维物体通过二次渲染创建阴影。

# 1.Projection Shadows
## 1.1基本概念
投影阴影的基本思路是将投影矩阵应用于应在平面π上投射阴影的三维物体，然后使用暗色和无照明渲染该投影对象。

## 1.2投影矩阵
如图1.1所示，光源位于l，投影的顶点在v，投影顶点在p。我们首先推导出阴影平面y＝0时的投影矩阵，然后将该结果推广至任意平面。
<div align=center>![图1.1](https://renderwiki.github.io/ImageResources/Projection Shadows/Figure1.1.png)</div>
<center>图1.1 左：阴影投射到平面y=0上。右：阴影投射到平面π:n·x+d=0上。</center>
首先推导x坐标的投影。从图1.1左侧的相似三角形中，可以得到：
<div align=center>![公式1.1](https://renderwiki.github.io/ImageResources/Projection Shadows/公式1.1.png)</div>
z坐标的推导方式相同，y坐标为零。此时，可以得到投影矩阵M：
<div align=center>![公式1.1](https://renderwiki.github.io/ImageResources/Projection Shadows/公式1.2.png)</div>
在一般情况下，我们应该将阴影投射到平面π：n·x+d=0上，如图1.1右侧所示。从l发出的光线穿过v，与平面π相交,产生了点p：
<div align=center>![公式1.1](https://renderwiki.github.io/ImageResources/Projection Shadows/公式1.3.png)</div>
该方程也可以转换为投影矩阵，如方程1.2所示，满足Mv=p：
<div align=center>![公式1.1](https://renderwiki.github.io/ImageResources/Projection Shadows/公式1.4.png)</div>
## 1.3基本算法
在实际使用时，必须避免在接收投影三角形的曲面下方渲染投影三角形。一种方法是在我们投射的平面上添加一些偏置，这样阴影三角形总是会呈现在表面前面。
更安全的方法是先绘制地平面，然后在z-buffer关闭的情况下绘制投影三角形，然后再像往常一样渲染其余的几何图形。因为没有进行深度比较，投影三角形总是会绘制在地平面的上方。如果地平面有界限，例如它是一个矩形，投影的阴影可能会落在它的外面。为了解决这个问题，可以使用一个stencil-buffer。首先，将接受平面绘制到屏幕和stencil-buffer。然后，在z-buffer关闭的情况下，只在了绘制了接受平面的区域绘制投影三角形，然后正常渲染场景的其余部分。
另一种阴影算法是将投影三角形渲染成纹理，然后将其应用于地平面。这个纹理是光照贴图的一种，可以调节表面的光照强度。这种将阴影投射到纹理上的做法可以产生半影和本影，但是有一个缺点，单个texel有时会覆盖多个像素。如果阴影情况在帧间没有发生变化，即光源和物体没有发生相对移动，就可以复用该纹理。
<div align=center>![图1.2](https://renderwiki.github.io/ImageResources/Projection Shadows/Figure1.2.png)</div>
<center>图1.2 左侧为正确的阴影，而在右侧，出现了antishadow。</center>
所有遮蔽物必须位于光源和接受平面之间。如图1.2所示，如果光源位于物体的下方，就会生成一个antishadow，因为每个顶点都会通过光源进行投影。如果投射一个位于接收平面下方的物体，也会发生错误。

参考文献：
[1] Blinn, Jim, “Me and My (Fake) Shadow,” IEEE Computer Graphics and Applications, vol. 8, no. 1, pp. 82–86, Jan. 1988. Also collected in [165]. Cited on p. 225, 227
[2] Tessman, Thant, “Casting Shadows on Flat Surfaces,” Iris Universe, pp. 16–19, Winter 1989. Cited on p. 225