## 简介

## 1.平面投影（Planer Projection）

### 1.1正交投影

对于三维几何形状简单的物体或者在某些特殊情况下，可以通过几何坐标点进行计算得到纹理坐标。其中从三维空间到二维空间，最简单的映射方式是正交投影（图1.1所示）

<div align=center>![figure-1](https://renderwiki.github.io/ImageResources/Texture_Coordinate_Function/figure_4.png)</div>

<center>图1.1 使用正交投影的方式将三维空间映射到二维空间。</center>


通过正交投影生成纹理坐标的方式可以通过以下简单的矩阵乘法得到，其中 ![](http://latex.codecogs.com/svg.latex?M_{t}) 表示一个仿射变换。
 
![](http://latex.codecogs.com/svg.latex? \phi(x,y,z)=(u,v),where \left[
     \begin{matrix} u\\ v \\ * \\ 1  
     \end{matrix}
 \right]= M_{t}\left[ 
     \begin{matrix} x\\ y \\ z \\ 1  
     \end{matrix}
     \right])
<math>\phi(x,y,z)=(u,v),where \left[ \begin{matrix} u\\ v \\ * \\ 1  
      \end{matrix}
      \right]= M_{t}\left[ 
      \begin{matrix} x\\ y \\ z \\ 1  
      \end{matrix}
      \right] \tag{1}
</math>


图1.2中左边的测试纹理图像使用正交投影的计算纹理坐标的方式被映射到立方体几何表面上。从右边的渲染结果可以看出，这种纹理坐标函数在平面，且平面的法线变化不大的地方可以得到很好的结果，通过计算法线的平均值可以得到一个较为满意的投影方向。但是这种平面投影并不具有很好的双射性，在立方体的上方和侧方边缘处都被映射到了纹理空间中的相同点。

<div align=center>![figure-2](https://renderwiki.github.io/ImageResources/Texture_Coordinate_Function/figure_5.png)</div>

<center>图1.2 使用正交投影计算纹理坐标，会存储一对多的映射，同时在投影方向和几何表面相切处会出现极端变形。</center>

### 1.2透视投影

还可以如图1.3中所示将正交投影替换为透视投影从而计算得到纹理坐标函数。

<div align=center>![figure-2](https://renderwiki.github.io/ImageResources/Texture_Coordinate_Function/figure_5.png)</div>

<center>图1.3 使用透视投影的方式将三维空间映射到二维空间。</center>

参考文献：

[1] Marschner S ,  Shirley P . Fundamentals of computer graphics. 4th edition.[J]. World Scientific Publishers Singapore, 2009, 9(1):248-249.