## 简介

PBR材质的参数与物理量相关，对于艺术家来说并不友好。例如现代物理中金属的折射率是以复数形式表示的，如n-ik。因此诞生了一些对艺术家友好的材质模型，Disney principled BRDF就是其中的代表。

## 1.Disney principled BRDF

### 1.1基本原则

Disney principled BRDF在物理上并不一定完全是正确的。但是一般来说实时渲染并不完全符合物理，因此可以认为Disney principled BRDF是近似符合PBR要求的。

为了方便艺术家使用，Disney principled BRDF具有以下原则：

- 使用艺术家更熟悉的变量而不是物理量；
- 参数数量越少越好；
- 参数范围应该位于[0, 1]之间；
- 参数的组合具有鲁棒性，允许自由搭配不同属性。

### 1.2如何应用

下图展示了一组相互独立的参数的应用效果

<div align=center>![Disney principled BRDF材质效果展示](https://renderwiki.github.io/ImageResources/MicrofacetBRDFParts/Disney principled BRDF材质效果展示.png)</div>

<cneter>图1 Disney principled BRDF材质效果展示 </center>

例如第一行中展示了添加次表面散射的效果，从左到右次表面散射程度越来越强，球体整体表现为越来越扁平；

第二行展示了金属性的效果，从左到右物体的金属度越来越强；

第三行展示了specular效果，用于控制镜面反射内容的多少，从左到右镜面反射的内容越来越多，高光效果越来越明显；

第四行展示了specularTint，即镜面反射的颜色，用于对镜面反射进行颜色控制；

第五行展示了粗糙程度，从左到右物体表面越来越粗糙；

第六行展示了各向异性程度，从各向同性到完全各向异性；

第七行展示了grazing angle位置处雾化的程度；

第八行展示了grazing angle位置处雾化的颜色；

第九行展示了添加一层透明层的程度，从左到右越来越透明，如汽车清漆的效果；

第十行展示了clearcoat的光滑程度，从左到右clearcoat越来越光滑，左侧最粗糙，类似磨砂效果。

上述不同参数可以共同起作用从而混合出不同的效果。同样，参数的范围越大越容易出现冗余、重合现象。

### 1.3优缺点 

Disney principled BRDF具有以下优缺点：

- 方便易懂，容易控制；
- 材质类型丰富；
- 开源，方便获取；
- 并不是基于物理的；
- 有巨大的参数空间，存在冗余信息。



参考文献：

Games202 实时高质量着色2（表面模型：Disney principled BRDF；方法：LTC；杂项：NPR）