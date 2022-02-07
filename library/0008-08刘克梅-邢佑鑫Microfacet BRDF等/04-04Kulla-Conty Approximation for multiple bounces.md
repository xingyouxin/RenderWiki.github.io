## 简介

Kulla-Conty近似主要指在微表面模型中，通过一种经验性的方式，针对BRDF项来补全多次反射丢失的能量。

## 1.The Kulla-Conty Approximation

### 1.1问题引入

下图展示了**白炉测试**实验中，Roughness由小到大的渲染图，仔细观察发现图中结果从左到右越来越暗，这是由于着色过程中仅考虑了一次反射。部分本应该反射出来的光线被忽略，因此Roughness值越大，效果越暗。

<div align=center>![多次弹射引起能量丢失示意图](https://renderwiki.github.io/ImageResources/MicrofacetBRDFParts/多次弹射引起能量丢失示意图.png)</div>

<center>图1-1 多次弹射引起能量丢失示意图</center>



采用多次光线追踪的方法（Heitz et al. 2016）可以得到正确的结果，不过这种方法由于计算量偏大并不适合在实时光线追踪中应用。在实时渲染中通过补全多次反射引起的能量损失来解决该问题，即Kulla-Conty近似法。

## 1.2Kulla-Conty近似法

### 1.2.1BRDF无颜色时的处理方法

Kulla-Conty近似是一种经验方法，旨在补全多次反射损失的能量。下述公式把BRDF、Lighting和cosine项在半球上进行积分，计算出射能量：

$$E(μ_o) = \int^{2\pi}_0\int^{1}_0f(μ_0, μ_i, \phi)μ_idμ_id\phi.\tag{1}$$

该公式描述了在光照uniform的情况下（任何方向的入射Radiance是1）发生一次弹射时反射出的能量。其中μ=sinθ，θ和Φ维空间中的两个角度，$μ_o$代表观察方向，$μ_i$为入射方向。因此损失的能量为$1-E(μ_0)$。考虑BRDF的对称性，待补充的能量表示为：

$$f_{ms}(μ_o, μ_i) = \frac{(1-E(μ_o))(1-E(μ_i))}{\pi(1-E_{avg})}, E_{avg}=2\int^1_0E(μ)μdμ. \tag{2}$$



通常会对$E(μ)$和$E_{avg}$做预计算，实际使用时通过查表的方式得到对应数据。

<div align=center>![kulla-conty预存二维表](https://renderwiki.github.io/ImageResources/MicrofacetBRDFParts/kulla-conty预存二维表.png)</div>

<center>图1-2 E(μ)的预存二维表（关于观察方向和粗糙度）</center>

添加Kulla-Conty近似后的结果如下：

<div align=center>![Kulla-Conty近似示意图（无颜色）](https://renderwiki.github.io/ImageResources/MicrofacetBRDFParts/Kulla-Conty近似示意图（无颜色）.png)</div>

<center>图1-3 Kulla-Conty近似示意图（无颜色）</center>

### 1.2.2BRDF有颜色时的处理方法

有颜色意味着物体发生了能量吸收的情况，即存在能量的损失，单次反射的积分结果不是1。Kulla-Conty近似方法的思路是先计算不含颜色的能量损失，然后再乘上能量损失比例（即颜色项）：

$$\frac{F_{avg}E_{avg}}{1-F_{avg}(1-E_{avg})}. \tag{3}$$

其中$E_{avg}$表示直接看到的能量，将不会参与后续的弹射；$F_{avg}$为平均Fresnel项，表示无论入射角多大平均每次会反射多少能量：

$$F_{avg}=\frac{\int^1_0F(μ)μdμ}{\int^1_0μdμ}=2\int^1_0F(μ)dμ \tag{4}$$

因此有：

- 直接观察到的能量：$F_{avg}E_{avg}$
- 经过一次弹射的能量：$F_{avg}(1-E_{avg})·F_{avg}E_{avg}$

  ...

- 经过k次弹射的能量：$F^k_{avg}(1-E_{avg})^k·F_{avg}E_{avg}$

将上述式子加起来可以得到一个级数，即Kulla-Conty近似的颜色项，如公式(3)。该颜色项乘上不含颜色的能量损失便得到了考虑颜色时应该补加的能量损失总量。

<div align=center>![Kulla-Conty近似示意图（无颜色）](https://renderwiki.github.io/ImageResources/MicrofacetBRDFParts/Kulla-Conty近似示意图（有颜色）.png)</div>

<center>图1-4 上：不考虑不加能量损失；下：Kulla-Conty近似示意图（有颜色）</center>



参考文献：

Games202 实时高质量着色1（表面模型：Microfacet Model, GGX, Multiple Bounce Approximation）

Kulla and Conty, [Revisiting Physically Based Shading at Imageworks](https://fpsunflower.github.io/ckulla/data/s2017_pbs_imageworks_slides_v2.pdf), SIGGRAPH 2017 course