## 1圆柱坐标（Cylinder Coordinates）

对于柱状而非球状的物体，通过一个轴到圆柱的投影可能得到比一个点到球的投影更好的结果，如下图1.1场景中的圆柱状花瓶，右侧花瓶使用圆柱坐标映射对于左侧使用球形坐标映射能得到更好的结果。

这种坐标映射方式和球坐标映射是类似的，不同的是需要使用圆柱坐标，并且舍弃半径这一个属性特征。

<div align=center>![figure-1](https://renderwiki.github.io/ImageResources/Texture_Coordinate_Function/figure_8.png)</div>

<center>图1.1 对于形状和球形差距较大的花瓶，使用球形坐标映射纹理会出现很明显的形状变形（左侧），而使用圆柱坐标映射却能得到较为不错的结果（右侧）。</center>

对应的纹理坐标函数可以通过下面的公式进行表示：

![](http://latex.codecogs.com/svg.latex?\phi(x,y,z)=(\left[ \pi+atan2(y,x) \right] /2\pi,\left[ 1+z\right] /2 ))

<math>\phi(x,y,z)=(\left[ \pi+atan2(y,x) \right] /2\pi,\left[ 1+z\right] /2 ) \tag{1}</math>

参考文献：

[1] Marschner S ,  Shirley P . Fundamentals of computer graphics. 4th edition.[J]. World Scientific Publishers Singapore, 2009, 9(1):249-250.