## SSDO（screenspace directional occlusion）

SSAO（screen space ambient occlusion）是近似实时全局光照的方法，其解耦了可见性和照明，只允许做出一个粗糙的近似，忽略入射光的所有的方向性的信息。

SSDO（screenspace directional occlusion）是基于SSAO的改进，用于近似屏幕空间中的直接光照和一个bounce的间接光照。屏幕空间的方法都应用于后处理阶段，不需要额外的数据(如表面描述、BVH树、KD树或阴影图等用于可见性的空间加速度结构)，并适用于许多类型的几何(如位移/法线贴图，顶点/几何着色器，表面光线投射)。

SSDO支持场景几何和光照的实时动态，比起单纯的遮蔽效果以外，还支持方向性的遮蔽（如阴影）以及间接光照效果（如色溢）。与经典的SSAO相比，其提出的泛化方法开销很小，可以容易地与其他模拟宏观结构光传输的方法相结合，在不引入artifact的情况下，在视觉上等同于SSAO的效果，如图1.1所示。

<div align=center>![SSDO渲染结果](https://renderwiki.github.io/ImageResources/SSDO/SSDO渲染结果.png)</div>

<center>图1.1 SSDO渲染结果不依赖于场景几何复杂度，在1600×1200 分辨率的图像下每秒运行20.4帧</center>

## 1.基本概念
SSDO使用位置和法线的帧缓冲区作为输入，并用2个渲染pass输出帧缓冲，一个输出直接光（direct light），另一个输出间接弹射（indirect bounce）。场景中使用一个环境光贴图和一个额外的带阴影贴图（shadow map）的点光源。

SSDO将直接光照和间接光照按照下列公式所示分别计算：

<div align=center>![公式1](https://renderwiki.github.io/ImageResources/SSDO/公式1.png)</div>

<math>L_o^{dir}(p,w_o)=\int_{\Omega^+,V=1}L_o^{dir}(p,w_i)fr(p,w_i,w_o)cos\theta_i {\rm d} w_i</math>

<math>L_o^{indir}(p,w_o)=\int_{\Omega^+,V=0}L_o^{indir}(p,w_i)fr(p,w_i,w_o)cos\theta_i {\rm d} w_i</math>


对于某个从![](http://latex.codecogs.com/svg.latex?\omega_{o})方向上看的shading point，往半球![](http://latex.codecogs.com/svg.latex?\\Omega^{+})各个方向随机发射一条光线，如果发射的光线方向![](http://latex.codecogs.com/svg.latex?\omega_{i})不被阻挡，即可见性![](http://latex.codecogs.com/svg.latex?V=1)，就按照直接光照来计算，否则，可见性![](http://latex.codecogs.com/svg.latex?V=0)，按照间接光照来计算。![](http://latex.codecogs.com/svg.latex?L_{i}^{\mathrm{dir}})(![](http://latex.codecogs.com/svg.latex?\mathrm{p},\omega_{i}))能够从点光源或环境光贴图高效地计算。Diffuse的![](http://latex.codecogs.com/svg.latex?f_{r})（BRDF函数）为ρ/π。

SSDO与SSAO对于直接光照和间接光照来源的计算方法刚好相反，如图1.2所示，对于SSAO，红圈有间接光照（全局光照），橙圈无间接光照（仅有直接光照）；对于SSDO，红圈无间接光照，橙圈有间接光照。SSDO的这一思想与路径追踪思想类似。

<div align=center>![光照分解计算](https://renderwiki.github.io/ImageResources/SSDO/光照分解计算.png)</div>

<center>图1.2 将shading point处的光照分解计算</center>


## 2.计算方法

可见性![](http://latex.codecogs.com/svg.latex?V)是在屏幕空间中近似计算的。对于每个法向为![](http://latex.codecogs.com/svg.latex?n)的shading point ![](http://latex.codecogs.com/svg.latex?P)，设置一组随机长度![](http://latex.codecogs.com/svg.latex?\lambda_{i} \in\left[0 \ldots r_{\max }\right])，![](http://latex.codecogs.com/svg.latex?r_{\max })是用户指定的半径，得到一组采样点![](http://latex.codecogs.com/svg.latex?P+\lambda_{i} \omega_{i})，分布在以![](http://latex.codecogs.com/svg.latex?P)为球心，![](http://latex.codecogs.com/svg.latex?n)为轴心的半球上。对于半球空间中的这些的采样点，有些会在物体表面外，而有些会在物体内部。在SSDO的可见性测试中，所有处于物体表面以下（即比第一个深度更远）的采样点都是遮挡物。

<div align=center>![SSDO直接光照和间接光照示意](https://renderwiki.github.io/ImageResources/SSDO/SSDO直接光照和间接光照示意.png)</div>

<center>图2.1 （左）计算带有方向性遮挡的直接光照，每个样本参与遮挡测试，（右）计算间接光照，每个表面patch存储了该点的直接光照</center>

如图2.1左所示，采样点（A、B、C、D）中，A、B和D位于物体内部，对于![](http://latex.codecogs.com/svg.latex?P)是遮挡物；而C处于物体外部，对于![](http://latex.codecogs.com/svg.latex?P)可见。逆向投影到屏幕空间即可测试采样点是否处于物体表面以下，可以从位置缓冲区（position buffer）中读取采样点在世界空间中的3D位置，接着按照红色箭头的位置投影到物体表面上。


<div align=center>![SSAO和SSDO对比](https://renderwiki.github.io/ImageResources/SSDO/SSAO和SSDO对比.png)</div>

<center>图2.2 左为SSAO直接光照和SSDO直接光照遮挡处对比，右为SSDO直接光照和一次额外bounce间接光照的遮挡处对比</center>

### 2.1直接光照
对于带有方向性遮挡的直接光照，此时只有C是不被遮挡的，因此只计算从C方向来的直接光照。这一做法在不同方向不同颜色的入射光的效果更显著，如图2.2所示，SSDO可以正确显示得到的彩色阴影（红色和蓝色），而SSAO只是在自遮挡位置显示灰色阴影。P点处的直接光照计算公式如下：

<div align=center>![公式2](https://renderwiki.github.io/ImageResources/SSDO/公式2.png)</div>

<math>L^{\mathrm{dir}}(\mathbf{P})=\sum_{i=1}^N \frac{\rho}{\pi}L_{in}(w_i)V(w_i)cos\theta_i \Delta w</math>

### 2.2间接光照
对于间接光照的计算，每个被分类为遮挡物的采样点（A、B、D）作为间接光照的发送者，P点作为接收者。发送者放置了一个小patch在表面上，在第一个pass中，直接光照被存储在帧缓存（frame buffer）里，对应的像素颜色为向P点发送的辐射亮度radiance。在这里注意到每个patch都有法线，这是为了防止背向面造成不应该存在的漏光现象而设置的。P点处接收其周围的patch传来的间接光照计算公式如下：

<div align=center>![公式3](https://renderwiki.github.io/ImageResources/SSDO/公式3.png)</div>

<math>L_{\mathrm{ind}}(\mathbf{P})=\sum_{i=1}^{N} \frac{\rho}{\pi} L_{\mathrm{pixel}}\left(1-V\left(\omega_{i}\right)\right) \frac{A_{s} \cos \theta_{s_{i}} \cos \theta_{\mathrm{r}_{i}}}{d_{i}^{2}}</math>

![](http://latex.codecogs.com/svg.latex?d_{i})是![](http://latex.codecogs.com/svg.latex?P)和遮挡物![](http://latex.codecogs.com/svg.latex?i)的距离（被clamp到1），![](http://latex.codecogs.com/svg.latex?\theta_{s_{i}})和![](http://latex.codecogs.com/svg.latex?\theta_{\mathrm{r}_{i}})是发送者/接收者的法线和传输方向的夹角。![](http://latex.codecogs.com/svg.latex?A_{s})是发送者patch的面积，![](http://latex.codecogs.com/svg.latex?A_{s}=\pi r_{\max }^{2} / N)（N为圆被分的区域个数），实际值可以根据处于半球的坡度来手动调节，以控制色溢的程度。A点由于法向朝向相对于![](http://latex.codecogs.com/svg.latex?P)处于背向，所以不贡献间接光照；C点（映射到物体表面后）处于半球外，也不参与贡献；此时B和D作为发送者向![](http://latex.codecogs.com/svg.latex?P)点贡献间接光照。

## 3.屏幕空间问题及解决方法
然而，基于屏幕空间的方法本身会有几个严重的问题：遮挡测试时将采样点错误分类，或者对于不可见的地方贡献的间接光照的计算缺失。如图3.1所示，从左到右是动画的四帧，光线从黄色物体上经过一次弹射到灰色平面上，产生箭头所指的色溢现象。随着视角的不断偏移，黄色墙面逐渐变得不可见直至背向屏幕，此时灰色平面的色溢现象逐渐淡去，直至完全消失。

<div align=center>![屏幕空间的色溢问题](https://renderwiki.github.io/ImageResources/SSDO/屏幕空间的色溢问题.png)</div>

<center>图3.1 不可见区域无法产生间接光照贡献</center>

有两种方法可以解决SSDO产生的偏差：深度剥离和额外相机。


### 3.1深度剥离
在之前的遮挡物测试中，由于基于屏幕空间的帧缓冲只存储距离相机最近的深度值，采样点可能会被错误分类（比如，将受阻碍的方向归类为可见的）。使用深度剥离后，前n个深度值经过n次pass被储存到帧缓冲。这样就获得了更多场景几何的信息以帮助更准确的遮挡检测。

之前仅仅判断采样点是否在第一个深度值以下（离相机更远处），在获得多个深度值以后，还测试采样点是否在第二个深度值的前面。当使用双流形几何时，第一个和第二个深度值对应于一个物体的正面和背面，所以两个面之间的采样点肯定在这个物体内部。对于深度复杂度较高的场景，要重建所有阴影，需要对所有连续的深度值(第三和第四个深度值等等)以相同的方式计算。

<div align=center>![遮挡测试中错误分类](https://renderwiki.github.io/ImageResources/SSDO/遮挡测试中错误分类.png)</div>

<center>图3.2 （左）为遮挡测试中两种分类错误的情况，（右）为各自的解决方法</center>

如图3.2所示，A的深度比z1更远，因此被错误地分类为遮挡，而B的深度尽管比最近深度更近，却被错误地分类为可见，此时P点的直接光照就被错误计算。在得到前n个深度后，能得知A并不处于z1和z2深度之间，因此被分类为可见；在B方向上多采样，就能找到遮挡。

### 3.2额外相机
另外，还可以使用不同的摄像机位置来代替不同的深度层查看隐藏区域。除了获取屏幕外遮挡物的信息外，不同的摄像机位置对于在掠角下观看的多边形尤其有用。当使用额外的相机时，之前消失的色溢现象就会变得可见。额外相机的视点最好与之前相机完全不同，例如围绕物体中心旋转90度，从原本相机的掠射角度（grazing angle）观察多边形。当然，在额外相机观察的多边形可能被其他物体遮挡，此时可以在额外相机的基础上使用深度剥离来解决。如图3.3所示，在使用多个相机后，可见性测试和色溢都能计算更加准确。

<div align=center>![更加准确的结果](https://renderwiki.github.io/ImageResources/SSDO/更加准确的结果.png)</div>

<center>图3.3 （上）为可见性遮挡测试，（下）为一次bounce的色溢，（左）为单个相机，（右）为多个相机</center>



参考文献：
[1] Tobias Ritschel, Thorsten Grosch, and Hans-Peter Seidel. 2009. Approximating dynamic global illumination in image space. In Proceedings of the 2009 symposium on Interactive 3D graphics and games (I3D '09). Association for Computing Machinery, New York, NY, USA, 75–82. DOI:https://doi.org/10.1145/1507149.1507161
...
