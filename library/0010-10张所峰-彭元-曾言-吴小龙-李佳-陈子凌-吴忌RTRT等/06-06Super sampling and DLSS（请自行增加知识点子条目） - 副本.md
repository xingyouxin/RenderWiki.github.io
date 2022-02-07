## 简介

简介

## 1. 超采样

### 1.1 定义

超采样(**Supersampling**)在实时渲染领域一般指在着色处理阶段每个像素计算一个以上的子像素样本，再通过滤波器生成滤波后的像素颜色，从而总体上使得画面更加平滑，细节更加丰富，因此超采样经常与实时渲染中的反走样技术相关。

### 1.2 SSAA

**Supersampling Antialiasing**简称**SSAA**，又被称为**FSAA**(**Full-Scene Antialiasing**)。它是一种概念上最简单的反走样算法。通常先渲染一幅高分辨率下的场景，然后对高分辨下渲染结果进行滤波处理得到低分辨率的图像。如**图1**所示，每个像素内采用4个子像素样本，分别进行着色计算再将每个子像素均值滤波得到最终像素颜色。

![alt文本](https://renderwiki.github.io/ImageResources/Super sampling/SSAA原理示意图.png)

**<center>图1 SSAA原理示意图</center>**

### 1.3 RGSS

研究者发现人眼对于近似水平和近似竖直方向的锯齿现象较为敏感<sup>[1](#f1)</sup>，而正常**SSAA**采用**OGSS**(**Ordered Grid Supersampling**)如**图1**所示的采样模式。考虑到接近45°的直线锯齿现象最严重，因此提出了**RGSS**(**Rotated Grid Supersampling**)如**图2**所示的采样模式，该采样模式采用旋转后的采样点，更适合捕获水平和竖直边缘。

![alt文本](https://renderwiki.github.io/ImageResources/Super sampling/RGSS原理示意图.png)

**<center>图2 RGSS原理示意图</center>**

### 1.4 MSAA

**Multisampling Antialiasing**简称**MSAA**，是一种为了减轻**SSAA**带来成倍性能开销的反走样算法。不同于**SSAA**对于每一个子像素样本都做一次实际的着色计算，**MSAA**只对深度缓存(**Z-Buffer**)和模板缓存(**Stencil Buffer**)中的数据进行超采样，再根据几何覆盖的结果复用着色结果，从而每次对于一个像素只执行了一次着色计算，大大减少了性能开销。如**图3**所示，每次片段着色器只进行一个采样点的着色计算，根据深度缓存和模板缓存来更新其他子样本点的着色结果。

![alt文本](https://renderwiki.github.io/ImageResources/Super sampling/MSAA原理示意图.png)

**<center>图3 MSAA原理示意图</center>**

### 1.5 CSAA和EQAA

**Custom Filter AntiAliasing**简称**CSAA**<sup>[2](#f2)</sup>是**NVIDIA**对于**MSAA**的改进，**Enhanced Quality AntiAliasing**简称**EQAA**<sup>[3](#f3)</sup>是**AMD**对于**MSAA**的改进，两者原理较为相似，都是对**MSAA**的数据存储和覆盖方法进一步解耦。例如**图4**中，**EQAA**不再直接向每个子像素样本中存储对应颜色以及深度缓存(模板缓存)值，而是另外设置一张表格存储最高k个覆盖率的颜色以及深度缓存(模板缓存)值，每个子像素都过映射关系访问或更新相应的值。

![alt文本](https://renderwiki.github.io/ImageResources/Super sampling/EQAA原理示意图.png)

**<center>图4 EQAA原理示意图</center>**

### 1.6 CFAA

**Custom Filter AntiAliasing**简称**CFAA**<sup>[4](#f4)</sup>，该方法起源于**AMD-ATI**的**R600**家庭。**CFAA**支持编程自定义采样模式，与**MSAA**的每个像素都执行一样采样模式不同。**CFAA**通过可编程的超采样模式来加入检测走样的相关程序，使得走样严重的地方采用更多的样本数，其他地方采用更少的样本，从而以较少的性能牺牲换取更平滑的效果，降低显卡资源的占用。

参考文献：

<a name="f1">[1]</a>Naiman, Avi C., "Jagged Edges: When Is Filtering Needed?," ACM Transactions on Graphics, vol. 14, no. 4, pp. 238-258, 1998.  
<a name="f2">[2]</a>[White Paper Coverage-Sampled Antialiasing](https://developer.download.nvidia.cn/SDK/10/direct3d/Source/CSAATutorial/doc/CSAATutorial.pdf)  
<a name="f3">[3]</a>[EQAA Modes for AMD 6900 Series Graphics Cards](https://developer.amd.com/wordpress/media/2012/10/EQAA%2520Modes%2520for%2520AMD%2520HD%25206900%2520Series%2520Cards.pdf)  
<a name="f4">[4]</a>[Shilov, Anton, Yaroslav Lyssenko, and Alexey Stepin, “Highly Defined: ATI Radeon HD 2000Architecture Review,” Xbit Laboratories website, Aug. 2007. Cited on p. 142](https://web.archive.org/web/20110213072435/http:/www.xbitlabs.com:80/articles/video/display/r600-architecture_8.html)  
...
