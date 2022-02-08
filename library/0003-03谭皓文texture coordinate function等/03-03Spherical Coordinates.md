## 1球坐标（Spherical Coordinates）

使用纬度、经度参数化球体是众所周知且被广泛使用的方法。球体的两个极点附近有失真现象，但是它覆盖了整个球体，且只有沿一条纬度线不连续。

球体表面可以通过纹理坐标函数将映射表面上的点和球面上的点进行映射从而完成参数化。该函数使用径向投影（radial projection），也就说从映射平面上对应点出发，和球体中心进行连线并找到其与球面的交点。这个交点的球坐标也就是映射平面对应点的纹理坐标。我们通过下面的公式来表示其纹理坐标函数：

![](http://latex.codecogs.com/svg.latex?\phi(x,y,z)=(\left[ \pi+atan2(y,x) \right] /2\pi,\\ \left[ \pi-acos2(z/\begin{Vmatrix}x\end{Vmatrix} ) \right] /\pi ))

<math>\phi(x,y,z)=(\left[ \pi+atan2(y,x) \right] /2\pi,\\ \left[ \pi-acos2(z/\begin{Vmatrix}x\end{Vmatrix} ) \right] /\pi ) \tag{1}</math>

除了极点处，一个球坐标映射在所有地方都是双射的。极点附近同样也会出现经纬度映射过程中类似的变形。下图1.1展示了使用球坐标映射纹理，渲染的结果。

<div align=center>![figure-1](https://renderwiki.github.io/ImageResources/Texture_Coordinate_Function/figure_7.png)</div>

<center>图1.1 使用球坐标映射后的渲染结果。同时可以注意到图中一些远离中心的区域被放大了，靠近中心的点被缩小了。</center>

参考文献：

[1] Marschner S ,  Shirley P . Fundamentals of computer graphics. 4th edition.[J]. World Scientific Publishers Singapore, 2009, 9(1):249.