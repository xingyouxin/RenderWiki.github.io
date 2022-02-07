## 简介

简介

## 1. 超采样

### 1.1 定义

超采样(**Supersampling**)在实时渲染领域一般指在着色处理阶段每个像素计算一个以上的子像素样本，再通过滤波器生成滤波后的像素颜色，从而总体上使得画面更加平滑，细节更加丰富，因此超采样经常与实时渲染中的反走样技术相关。

### 1.2 SSAA

**Supersampling Antialiasing**简称**SSAA**，又被称为**FSAA**(**Full-Scene Antialiasing**)。它是一种概念上最简单的反走样算法。通常先渲染一幅高分辨率下的场景，然后对高分辨下渲染结果进行滤波处理得到低分辨率的图像。如**图1**所示，每个像素内采用4个子像素样本，分别进行着色计算再将每个子像素均值滤波得到最终像素颜色。

<div align=center>![alt文本](https://renderwiki.github.io/ImageResources/Super sampling/SSAA原理示意图.png)</div>

**<center>图1.2 SSAA原理示意图</center>**

### 1.3 RGSS

研究者发现人眼对于近似水平和近似竖直方向的锯齿现象较为敏感<sup>[1](#f1)</sup>，而正常**SSAA**采用**OGSS**(**Ordered Grid Supersampling**)如**图1**所示的采样模式。考虑到接近45°的直线锯齿现象最严重，因此提出了**RGSS**(**Rotated Grid Supersampling**)如**图2**所示的采样模式，该采样模式采用旋转后的采样点，更适合捕获水平和竖直边缘。

<div align=center>![alt文本](https://renderwiki.github.io/ImageResources/Super sampling/RGSS原理示意图.png)</div>

**<center>图1.3 RGSS原理示意图</center>**

### 1.4 MSAA

**Multisampling Antialiasing**简称**MSAA**，是一种为了减轻**SSAA**带来成倍性能开销的反走样算法。不同于**SSAA**对于每一个子像素样本都做一次实际的着色计算，**MSAA**只对深度缓存(**Z-Buffer**)和模板缓存(**Stencil Buffer**)中的数据进行超采样，再根据几何覆盖的结果复用着色结果，从而每次对于一个像素只执行了一次着色计算，大大减少了性能开销。如**图3**所示，每次片段着色器只进行一个采样点的着色计算，根据深度缓存和模板缓存来更新其他子样本点的着色结果。

<div align=center>![alt文本](https://renderwiki.github.io/ImageResources/Super sampling/MSAA原理示意图.png)</div>

**<center>图1.4 MSAA原理示意图</center>**

### 1.5 CSAA和EQAA

**Custom Filter AntiAliasing**简称**CSAA**<sup>[2](#f2)</sup>是**NVIDIA**对于**MSAA**的改进，**Enhanced Quality AntiAliasing**简称**EQAA**<sup>[3](#f3)</sup>是**AMD**对于**MSAA**的改进，两者原理较为相似，都是对**MSAA**的数据存储和覆盖方法进一步解耦。例如**图4**中，**EQAA**不再直接向每个子像素样本中存储对应颜色以及深度缓存(模板缓存)值，而是另外设置一张表格存储最高k个覆盖率的颜色以及深度缓存(模板缓存)值，每个子像素都过映射关系访问或更新相应的值。

<div align=center>![alt文本](https://renderwiki.github.io/ImageResources/Super sampling/EQAA原理示意图.png)</div>

**<center>图1.5 EQAA原理示意图</center>**

### 1.6 CFAA

**Custom Filter AntiAliasing**简称**CFAA**<sup>[4](#f4)</sup>，该方法起源于**AMD-ATI**的**R600**家庭。**CFAA**支持编程自定义采样模式，与**MSAA**的每个像素都执行一样采样模式不同。**CFAA**通过可编程的超采样模式来加入检测走样的相关程序，使得走样严重的地方采用更多的样本数，其他地方采用更少的样本，从而以较少的性能牺牲换取更平滑的效果，降低显卡资源的占用。

## 2. DLSS

## 简介
**DLSS** 全称 **Deep Learning Super Sampling**，是 Nvidia 于2019年提出的一种全新的 RTX 技术，使用深度学习和 AI 的强大功能来训练 GPU 以渲染清晰的应用程序或游戏图像。  

### 2.1 定义
**NVIDIA** **DLSS**（深度学习超级采样）是一项开创性 AI 渲染技术，它利用GeForce RTX GPU上的专用 AI 处理单元 - Tensor Core 将视觉保真度提升至全新高度。DLSS 利用深度学习神经网络的强大功能提高帧率，为游戏生成精美清晰的图像。

### 2.2 功能
__DLSS__ 利用 NVIDIA 超级计算机来训练和改进其 AI 模型。更新后的模型将通过游戏就绪驱动程序（Game Ready Drivers）传输给用户的 GeForce RTX PC。然后，Tensor Core 将利用其 Teraflops 级的专用 AI 性能来实时运行 DLSS AI 网络，用户可以利用 DLSS 超级计算机网络的强大性能来帮助自己提升性能和分辨率。  
__DLSS__ 历经三代发展，功能愈加强大，目前最新版本为DLSS 2.1。  
**DLSS 2.0**可以通过仅渲染1/4到一半的像素就可以提供与原始分辨率相当的图像质量。并使用非特定的游戏内容进行训练，提供能在多个游戏工作的通用网络，能够更快集成到更多游戏中。  
**DLSS 2.0**提供了3种图像质量模式：**Quality**，**Balanced**，**Performance**。**DLSS 2.1** 新增 **Ultra Performance** 模式。不同模式可控制游戏的内部渲染分辨率，Quality模式可实现高达4倍的超分辨率（即1080p→4K）。  
<div align=center>![游戏Deliver Us the Moon – DLSS 2.0 Quality模式开启对比](https://renderwiki.github.io/ImageResources/Super sampling/DLSS 2.0 Quality模式开启图片对比.png)</div>
<center>图2.2 游戏Deliver Us the Moon – DLSS 2.0 Quality模式开启对比<sup>[5](#s1)</sup></center>  


### 2.3 基本技术实现
**DLSS**深度神经网络在 NVIDIA DGX 驱动的超级计算机上进行训练。**DLSS 2.0**在 AI 网络中有两个主要输入：
1. 由游戏引擎渲染的低分辨率、带锯齿的图像；
2. 由游戏引擎生成的低分辨率、来自相同图像的运动矢量。  

运动矢量告诉我们场景中的对象在帧与帧之间的移动方向。我们可以将这些运动矢量应用于先前高分辨率的输出帧，以估计下一帧图像。我们将这一过程称为“时域反馈”，因为它使用历史来告知未来。
<div align=center>![NVIDIA DLSS 2.0 架构](https://renderwiki.github.io/ImageResources/Super sampling/NVIDIA DLSS 2.0 架构.png)</div>
<center>图2.3 NVIDIA DLSS 2.0 架构<sup>[5](#s1)</sup></center>  

**DLSS 2.0**使用的 AI 神经网络叫做**卷积自动编码器（convolutional autoencoder）**<sup>[5](#s1)</sup>，采用低分辨率的当前帧和高分辨率的上一帧，逐个像素地确定如何生成更高质量的当前帧。  
在训练过程中，将输出图像与离线渲染生成的超高质量16K参考图像进行比较，并将差异传达回网络，以便它可以继续学习和改进其结果。这个过程在超级计算机上重复数万次，直到网络能够可靠地输出高质量、高分辨率的图像。  
一旦网络训练完成，NGX 就会通过 Nvidia 显卡的游戏就绪驱动程序（Game Ready Drivers）和 OTA updates 将 AI 模型传送到用户的 GeForce RTX PC上。图灵的 Tensor Cores 提供高达110 teraflops的专用 AI 算力，使得**DLSS**网络可以与大型的3D游戏同时实时运行。

### 2.4 支持DLSS的游戏与应用
目前已有超过150个游戏和应用程序支持**DLSS**，例如支持的游戏有《赛博朋克2077》（Cyberpunk 2077）、《堡垒之夜》（Fortnite）、《战地2042》（Battlefield 2042）、《死亡搁浅》（Death Stranding）、《永劫无间》等等<sup>[6](#s2)</sup>；支持的应用程序有Autodesk VRED、Dabanjia BIM、Dimension 5 Techs D5 Render、Enscape等<sup>[7](#s3)</sup>。  
以游戏Deliver Us The Moon为例，通过支持**DLSS 2.0**，游戏性能可以提升**60%**。
<div align=center>![游戏Deliver Us the Moon – DLSS 2.0 Quality模式性能对比](https://renderwiki.github.io/ImageResources/Super sampling/DLSS 2.0 Quality模式性能对比.png)</div>

<center>图2.4.1 游戏Deliver Us the Moon – DLSS 2.0 Quality模式性能对比<sup>[5](#s5)</sup></center>  

此外，**DLSS 2.0**能够提供更好的图像质量和更出色的时间稳定性，并能够以更高的清晰度显示更多细节。

<div align=center>![游戏Deliver Us the Moon –DLSS 2.0 画面对比图1](https://renderwiki.github.io/ImageResources/Super sampling/游戏Deliver Us the Moon –DLSS 2.0 画面对比图1.png)</div>  

<div align=center>![游戏Deliver Us the Moon –DLSS 2.0 画面对比图2](https://renderwiki.github.io/ImageResources/Super sampling/游戏Deliver Us the Moon –DLSS 2.0 画面对比图2.png)</div>

<center>图2.4.2 游戏Deliver Us the Moon –DLSS 2.0 画面对比<sup>[5](#s1)</sup></center>  


参考文献：

<a name="f1">[1]</a>Naiman, Avi C., "Jagged Edges: When Is Filtering Needed?," ACM Transactions on Graphics, vol. 14, no. 4, pp. 238-258, 1998.  
<a name="f2">[2]</a>[White Paper Coverage-Sampled Antialiasing](https://developer.download.nvidia.cn/SDK/10/direct3d/Source/CSAATutorial/doc/CSAATutorial.pdf)  
<a name="f3">[3]</a>[EQAA Modes for AMD 6900 Series Graphics Cards](https://developer.amd.com/wordpress/media/2012/10/EQAA%2520Modes%2520for%2520AMD%2520HD%25206900%2520Series%2520Cards.pdf)  
<a name="f4">[4]</a>[Shilov, Anton, Yaroslav Lyssenko, and Alexey Stepin, “Highly Defined: ATI Radeon HD 2000Architecture Review,” Xbit Laboratories website, Aug. 2007. Cited on p. 142](https://web.archive.org/web/20110213072435/http:/www.xbitlabs.com:80/articles/video/display/r600-architecture_8.html)  
<a name='s1'>[5]</a> https://www.nvidia.com/en-us/geforce/news/nvidia-dlss-2-0-a-big-leap-in-ai-rendering/  
<a name='s2'>[6]</a> https://www.nvidia.cn/geforce/technologies/dlss/  
<a name='s1'>[7]</a> https://www.nvidia.com/en-us/geforce/news/nvidia-rtx-games-engines-apps/
