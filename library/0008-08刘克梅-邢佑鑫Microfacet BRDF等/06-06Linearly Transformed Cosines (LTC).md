## 简介

Linearly Transformed Cosines (LTC)主要解决在多边形光源下使用微表面模型的BRDF来进行着色的问题，如果不用LTC，则需要在光源上进行采样。

## 1.Linearly Transformed Cosines (LTC)

### 1.1主要特点

LTC的主要特点有：

- 主要针对GGX模型，对于其他模型同样适用；
- 仅支持shading，不考虑阴影；
- 光源为均匀发光的多边形光源。

### 1.2主要思想

将二维BRDF的Lobe通过线性变换，转换成一个cosine函数，在BRDF变换的同时，light的形状也会做对应的变换，成为另一个多边形光源。最终原始计算将转换为固定的cosine项与任意的多边形光源进行积分的形式，而这一积分存在解析解。

### 1.3具体方法

原多边形光源的光照计算公式如下：

<div align=center>![原多边形光源光照计算公式](https://renderwiki.github.io/ImageResources/MicrofacetBRDFParts/原多边形光源光照计算公式.png)</div>

![](http://latex.codecogs.com/svg.latex?w_o)为出射方向，![](http://latex.codecogs.com/svg.latex?w_i)为入射方向，![](http://latex.codecogs.com/svg.latex?L_i)为光源强度，![](http://latex.codecogs.com/svg.latex?F)函数包含了BRDF和余弦项，![](http://latex.codecogs.com/svg.latex?P)为积分区域。

由于![](http://latex.codecogs.com/svg.latex?w_i)可以由![](http://latex.codecogs.com/svg.latex?w_i')通过线性变换矩阵![](http://latex.codecogs.com/svg.latex?M)计算并归一化后得到：

<div align=center>![wi应用变换矩阵M公式](https://renderwiki.github.io/ImageResources/MicrofacetBRDFParts/wi应用变换矩阵M公式.png)</div>

对原始光照公式进行一次变量替换使得原BRDF和余弦项![](http://latex.codecogs.com/svg.latex?F)变成cosine项，将上式带入原多边形光源的光照计算公式可变换得到新的计算公式：

<div align=center>![LTC换元中间式](https://renderwiki.github.io/ImageResources/MicrofacetBRDFParts/LTC换元中间式.png)</div>

最终整理得到：

<div align=center>![LTC最终式](https://renderwiki.github.io/ImageResources/MicrofacetBRDFParts/LTC最终式.png)</div>

其中J为雅各布项。引用上述公式便可以计算求解多边形光源下的着色问题。不过LTC仍然具有局限性，仅支持不考虑阴影的着色计算。



ac.wiki所需的公式：

<math>L(w_o)=L_i·\int_PF(w_i)dw_i. \tag{1}</math>

<math>w_i=\frac{Mw_i'}{||Mw_i'||}. \tag{2}</math>

<math>L(w_o)=L_i·\int_Pcos(w_i')d\frac{Mw_i'}{||Mw_i'||}. \tag{3}</math>

<math>L(w_o)=L_i·\int_Pcos(w_i')Jdw_i'. \tag{4}</math>



参考文献：

Games202 实时高质量着色2（表面模型：Disney principled BRDF；方法：LTC；杂项：NPR）