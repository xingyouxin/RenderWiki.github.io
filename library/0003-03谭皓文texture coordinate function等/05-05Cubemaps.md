## 1.立方体贴图（Cubemaps）

使用球坐标来参数化球形或者类似球形的几何对象，会导致在极点附近的纹理映射出现极端的形状和尺寸变形，而这也会导致肉眼可见的失真现象。于是就出现了立方体贴图作为解决方案，使用这个解决方案使得结果更加的整齐，减少形状和尺寸变形，但同时也会带来更多的不连续性。

立方体贴图的基本思想是：将纹理投影到一个立方体上，而不是之前的球体，然后对立方体的六个面使用六个独立的正方形的纹理。而这六个正方形纹理的纹理集合被称为立方体贴图（Cubemaps），通常它们被打包在一个图像中进行存储，像一个立方体被展开似进行排列。在这些正方形纹理的边缘交界处会出现不连续现象，但是它保持着形状和尺寸映射的低失真。

计算立方体贴图的纹理坐标的代价要小于球坐标，这是因为投影到一个平面上仅仅需要一个除法——这与观察透视投影在本质上是相同的的。拿投影在立方体 ![](http://latex.codecogs.com/svg.latex?+z) 面的一个点来举例：

![](http://latex.codecogs.com/svg.latex?(x,y,z) \rightarrow(x/z,y/z))

<math>(x,y,z) \rightarrow(x/z,y/z) \tag{1}</math>
### 1.1 OpenGL中Cubemaps方向约定

对于立方体贴图需要建立被定义在六个面上的 ![](http://latex.codecogs.com/svg.latex?u,v) 的方向的约定，我们可以任意合理的约定规则，但是这个约定会影响到纹理的内容，因此需要制定一个标准化。因为立方体贴图经常被引用在立方体内部进行观察的纹理（比如：环境映射），这个常见的约定包含定向的 ![](http://latex.codecogs.com/svg.latex?u,v) 轴，所以从内部看 ![](http://latex.codecogs.com/svg.latex?u) 是相对于 ![](http://latex.codecogs.com/svg.latex?v) 的顺时针方向。OpenGL中使用的约定规则可以通过以下公式表示：


![](http://latex.codecogs.com/svg.latex?\phi_{-x}=\frac{1}{2}[1+(+z,-y)/|x|],)

<math>\phi_{-x}=\frac{1}{2}[1+(+z,-y)/|x|], \tag{2}</math>

![](http://latex.codecogs.com/svg.latex?\phi_{+x}=\frac{1}{2}[1+(-z,-y)/|x|],)

<math>\phi_{+x}=\frac{1}{2}[1+(-z,-y)/|x|], \tag{3}</math>

![](http://latex.codecogs.com/svg.latex?\phi_{-y}=\frac{1}{2}[1+(+x,-z)/|y|],)

<math>\phi_{-y}=\frac{1}{2}[1+(+x,-z)/|y|], \tag{4}</math>

![](http://latex.codecogs.com/svg.latex?\phi_{+y}=\frac{1}{2}[1+(+x,-z)/|y|],)

<math>\phi_{+y}=\frac{1}{2}[1+(+x,-z)/|y|], \tag{5}</math>

![](http://latex.codecogs.com/svg.latex?\phi_{-z}=\frac{1}{2}[1+(-x,-y)/|z|],)

<math>\phi_{-z}=\frac{1}{2}[1+(-x,-y)/|z|], \tag{6}</math>

![](http://latex.codecogs.com/svg.latex?\phi_{+z}=\frac{1}{2}[1+(+x,-y)/|z|].)

<math>\phi_{+z}=\frac{1}{2}[1+(+x,-y)/|z|]. \tag{7}</math>

上面公式中 ![](http://latex.codecogs.com/svg.latex?\phi) 的下标表示的是投影到哪一个面，例如：![](http://latex.codecogs.com/svg.latex?\phi_{-x}) 被用于在 ![](http://latex.codecogs.com/svg.latex?x=+1) 时投影到立方表面某个点。可以通过查看坐标的最大绝对值来判断一个点被投影到了立方体的哪个面上，例如：当 ![](http://latex.codecogs.com/svg.latex?|x|>|y|) 且 ![](http://latex.codecogs.com/svg.latex?|x|>|z|) 时，点被投影到了 ![](http://latex.codecogs.com/svg.latex?+x) 或者 ![](http://latex.codecogs.com/svg.latex?-x) 面上，而具体被投影到了哪个面上，则要通过 ![](http://latex.codecogs.com/svg.latex?x) 的符号来决定。


<div align=center>![figure-1](https://renderwiki.github.io/ImageResources/Texture_Coordinate_Function/figure_9.png)</div>

<center>图1.1 一个被投影到一个立方体贴图的表面。表面上的点从中间往外部投影，这些点被投影到立方体贴图六个面中的其中一个面。</center>

参考文献：

[1] Marschner S ,  Shirley P . Fundamentals of computer graphics. 4th edition.[J]. World Scientific Publishers Singapore, 2009, 9(1):250-251.