## 简介

次表面散射是在具有高散射系数的固体材质中发现的一种复杂现象，这种材质包括蜡、人的皮肤和牛奶。

整体参考自[1]

## 1.次表面散射（Subsurface Scattering）

### 1.1基本概念

对于光学深度较高的介质，比如人类皮肤，散射光会从表面接近其最初入射点的位置出射，这种位置上的偏移意味着次表面散射不能用BRDF建模。

图1显示了光在物体中的散射，这种散射会导致入射的光线形成许多不同的路径。单独模拟每个光子是不切实际的（即使是离线渲染），所以必须通过对可能的路径进行积分或通过近似这样的积分来解决这个问题。

<div align=center>![](https://renderwiki.github.io/ImageResources/Subsurface Scattering/光在物体中的散射.png)</div>

<center>图1 光在物体中的散射 </center>

### 1.2单次散射和多次散射

根据一条光路中散射事件的数量，可以分为单次散射（single scattering）和多次散射（multiple scattering）。单次散射指光被散射一次就离开材质，而多次散射则意味着光在射出材质前进行了两次或更多次散射。

### 1.3渲染方法

#### 1.3.1Wrap Lighting

Wrap Lighting是实现次表面散射的最简单的方式，其是一种近似面光源的方法，这种方法试图模拟弯曲表面的多次散射效果。对于Lambertian 着色模型表示漫反射的物体, 其引入一个系数让光柔和过渡到暗的地方。为了进一步近似次表面散射，Wrap Lighting根据系数加上一些随材质变化的颜色偏移效果，如图2，这就可以呈现穿过这种材质的光被部分吸收的现象。例如，在渲染皮肤时，可以应用红色偏移(shift)。

<div align=center>![](https://renderwiki.github.io/ImageResources/Subsurface Scattering/应用Wrap Lighting的球面.png)</div>

<center>图2 应用Wrap Lighting的球面</center>

#### 1.3.2Normal Blurring

Stam[2]指出，多次散射可以被建模为一个漫反射(diffusion)过程。其对出射辐射度有空间模糊效应。这种模糊只适用于漫反射，镜面反射发生在材质表面，不受次表面散射的影响。

Ma等人[3]根据测量数据，发现模糊程度在可见光谱上有所不同。所以他们提出了一种实时着色技术，为漫反射的RGB三个通道分别使用三张模糊的法线贴图。但对每个通道使用不同的法线贴图会导致颜色渗漏(color bleeding)。由于这些漫反射法线图通常类似于镜面反射图的模糊版本，因此也可以简化为使用单一的法线贴图，同时调整mipmap级别。其代价是失去颜色偏移，因为每个通道的法线都是一样的。

#### 1.3.3Pre-Integrated Skin Shading

Penner[4]结合了Warp Lighting和Normal Blurring的想法，提出了一种预积分的皮肤着色方法。皮肤表面的着色受两个系数的影响, 一个是![](https://latex.codecogs.com/svg.image?\mathbf{n} \cdot \mathbf{l}) 的值，另一个是皮肤表面的曲率![](https://latex.codecogs.com/svg.image?1 / r=\|\partial n / \partial p\|)。曲率越大，皮肤受散射的影响就越大，如鼻子处散射比面部会更加明显)。这样将这两个系数作为输入值, 对散射和透射进行预积分得到一张二维LUT表。然后再加上前一小节所讲的 Normal Blurring操作。

#### 1.3.4Texture-Space Diffusion

模糊漫反射的法线可以模拟次表面散射，但无法模拟软阴影边缘部分。而纹理空间的漫反射可以解决这个问题，其思想是使用模糊（Blur）来模拟多次散射：

1. 使用纹理坐标作为光栅化的位置，将表面irradiance(漫反射光照)渲染成一个纹理贴图。
2. 根据材质属性对得到的纹理贴图进行模糊。不同波长，模糊的程度不同。如，对于皮肤，R通道使用比G或B更宽的滤波器进行滤波，使得阴影边缘变红。
3. 在渲染时，该纹理贴图被映射到世界空间，根据其真实位置进行着色插值，从而完成漫反射着色。

该方法首先被用于离线渲染，但NVIDIA和ATI的研究人员很快就提出了实时GPU实现。d'Eon和Luebke[5]的报告代表了该技术最完整的处理方法之一，包括对多层次表面结构效果的支持。

#### 1.3.5Screen-Space Diffusion

Jimenez[6]提出一种屏幕空间的方法：

- 首先，将场景像往常一样渲染，并把需要进行次表面散射的网格（如人脸）记录在模板缓存区（stencil buffer）中。
- 然后，在存储的radiance上使用双pass屏幕空间过程模拟次表面散射，使用模板测试从而只在包含半透明材质的像素中应用这一过程。通过两次水平和垂直地应用两个一维和双边模糊核，来实现模糊的效果。

为了进一步优化这个过程，线性深度可以被存储在场景纹理的alpha通道中。

实现屏幕空间漫反射时，必须注意只模糊irradiance，而不是漫反射反照度(diffuse albedo)或镜面光照。

一种可行的方法是将irradiance和镜面光照渲染到单独的屏幕空间缓冲区中。 如果使用了延迟着色，那么已经有了漫反射的缓冲区。

为了减少存储带宽，Gallagher和Mittring[7]建议将irradiance和镜面光照使用棋盘格模式存储在一个缓冲区中。在对irradiance进行模糊处理后，通过将漫反射反照率与模糊的irradiance相乘并与镜面光照相加来合成最终的图像。

#### 1.3.6Depth-Map Techniques

对于表现出大规模散射的材质来说，必须考虑光照从背面穿过后的透射效果(比如光照在手上的效果)。为了模拟这种效果，需要测量光在材质中传播的距离。而估算这个距离可以使用深度贴图（Depth Maps）技术，此技术非常类似于阴影贴图(Shadow Mapping)，并且可用于实时渲染。

Barr´e-Brisebois和Bouchard[8]提出了一种廉价的对大范围次表面散射的近似方法。首先，他们为每个网格生成一个灰度纹理以存储平均局部厚度，该厚度为1减去从向内的法线![](https://latex.codecogs.com/svg.image?-n)计算的环境遮挡。这个纹理表示为为![](https://latex.codecogs.com/svg.image?t_{ss})，被认为是一个近似的透射率:

![img](file:///C:/Users/lxf11/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)

其中![](https://latex.codecogs.com/svg.image?\mathrm{l})和![](https://latex.codecogs.com/svg.image?\mathrm{v})分别是归一化的光和视图向量，![](https://latex.codecogs.com/svg.image?\mathrm{p})是一个近似相位函数的指数，![](https://latex.codecogs.com/svg.image?\mathrm{c}_{ss})是次表面反照率。然后将这个表达式与光的颜色、强度和距离衰减相乘。

这个模型不是基于物理的，也不是能量守恒的，但是它能够一次就迅速渲染出合理的次表面照明效果，如图3。

<div align=center>![](https://renderwiki.github.io/ImageResources/Subsurface Scattering/图3.png)</div>

<center>图3 左图是为Hebe雕像生成的局部厚度纹理。中间是用它实现的次表面光散射效果。右图是另一个使用相同技术渲染的半透明立方体的场景。(图片由Colin Barr´e-Brisebois和Marc Bouchard提供[8]）/center>
参考文献：

[1] Tomas Akenine-Mller, Eric Haines, and Naty Hoffman. 2018. Real-Time Rendering, Fourth Edition (4th. ed.). A. K. Peters, Ltd., USA.

[2] Stam, Jos, “Multiple Scattering as a Diffusion Process,” in Rendering Techniques ’95, Springer, pp. 41–50, June 1995.

[3] Ma, Wan-Chun, Tim Hawkins, Pieter Peers, Charles-F´elix Chabert, Malte Weiss, and Paul Debevec, “Rapid Acquisition of Specular and Diffuse Normal Maps from Polarized Spherical Gradient Illumination,” in Proceedings of the 18th Eurographics Symposium on Rendering Techniques, Eurographics Association, pp. 183–194, June 2007.

[4] Penner, E., “Pre-Integrated Skin Shading,” SIGGRAPH Advances in Real-Time Rendering in Games course, Aug. 2011.

[5] d’Eon, Eugene, and David Luebke, “Advanced Techniques for Realistic Real-Time Skin Rendering,” in Hubert Nguyen, ed., GPU Gems 3, Addison-Wesley, pp. 293–347, 2007.

[6] Jimenez, Jorge, “Next Generation Character Rendering,” Game Developers Conference, Mar. 2013.

[7] McGuire, Morgan, The Graphics Codex, Edition 2.14, Casual Effects Publishing, 2018.

[8] Barr´e-Brisebois, Colin, and Marc Bouchard, “Approximating Translucency for a Fast, Cheap and Convincing Subsurface Scattering Look,” Game Developers Conference, Feb.–Mar. 2011.

