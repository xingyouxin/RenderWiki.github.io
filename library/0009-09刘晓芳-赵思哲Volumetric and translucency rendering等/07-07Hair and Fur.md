## 简介

整体参考自[1]

## 1.毛发（Hair）和皮毛（Fur）

### 1.1定义

毛发和皮毛是生长在哺乳动物的真皮层中的蛋白质丝。毛发一般形容在人体不同部位生长的蛋白质丝，如头发、胡须、眉毛和睫毛等。而其他哺乳动物身上覆盖的，通常称为皮毛。在视觉上看，皮毛可以看作是密集的、长度有限的毛发。它们因其生长部位的不同而具有不同的特性。

### 1.2基本结构

<div align=center>![](https://renderwiki.github.io/ImageResources/Hair and Fur/一根毛发的纵向切面.png)</div>

<center>图1 一根毛发的纵向切面 </center>

毛发和皮毛的结构基本相同, 由三层组成，如图1所示:

- 外面是角质层（cuticle），表示纤维的表面。这个表面是不光滑的，由重叠的鳞片组成，鳞片相对于头发倾斜约![](https://latex.codecogs.com/svg.image?\alpha=3^{\circ})，使即法线向根部（root)方向倾斜。（<math>\alpha=3^{\circ}</math>)
- 中间是皮质层（cortex），含有黑色素从而使纤维具有颜色。其中有一种真黑素，可以呈现棕色，![](https://latex.codecogs.com/svg.image?\alpha_{a,e}=(0.419,0.697,1.37))；还有一种伪黑色素，可以呈现红色![](https://latex.codecogs.com/svg.image?\alpha_{a,p}=(0.187,0.4,1.05))。（<math>\alpha_{a,e}=(0.419,0.697,1.37)</math>)、（<math>\alpha_{a,p}=(0.187,0.4,1.05)</math>)
- 内部是髓质层（medulla），它很小，在建立人体毛发模型时经常被忽略。但在动物皮毛中，它占据了较大体积，具有更大的意义。

### 1.3几何表示

一般类似胡须或者静态的短的毛发可以先由艺术家画出头发方向的引导曲线，然后在顶点着色器中，通过挤压四边形形成丝带形状来渲染毛发线条。这是有效的，因为大的四边形可以覆盖更大的面积，因此如头发所需的四边形就越少，从而提高了性能。

但如果需要更多细节，通常需要成千上万更薄的四边形丝进行渲染。此时，这些四边形最好也使用沿着头发曲线的切线的圆柱体约束朝向视图。

### 1.4基于alpha的混合

上述所有元素都可以被渲染为alpha混合（alpha-blended）的几何体，需要保证其渲染顺序以避免产生透明度伪影（transparency artifacts）。

可以使用预先排序的索引缓冲区，将里面的毛发排在前面，外层的毛发排在最后。这对于短头发非常实用，但对于长的相互交错的头发就不适用了。

依靠深度测试来解决排序问题，可以使用alpha检测（alpha testing），但会出现高频几何与纹理的走样问题。可以使用MSAA对每个样本进行alpha检测，但要进行额外的采样，以及付出额外的内存带宽。

此外，渲染毛发时也可以使用与顺序无关的半透明方法。

### 1.5毛发

Marschner等人[2]测量了人类毛发纤维中的光散射，并根据这些观察提出了一个模型，如图1所示。

 R表示光在角质层的空气与纤维表面的反射。TT表示光射入毛发纤维，然后出射的过程，其中整个路径的事件只包括两个：一次从空气射入毛发纤维发生折射，一次从毛发纤维折射返回空气。TRT表示光从空气射入毛发纤维，之后在纤维另一侧的内部表面发生反射，然后又在纤维的入射表面射出。其中R表示纤维内部的反射。

在视觉上，R被认为时头发上无色的镜面反射。当一卷头发从后面被照亮时，TT被认为时一个明亮的高光。而TRT对于渲染真实毛发至关重要，它将导致头发上的闪烁（glints）。

由此可以推出该散射模型中更多路径的表示，如TRRT、TRRRT和更长的路径。

如果想对毛发渲染进行更深的了解，可以参考Yuksel和Tariq[3]在网上提供的一个全面的实时头发渲染课程。

### 1.6皮毛

#### 1.6.1Volumetric Textures

体积纹理贴图是由二维半透明纹理层表示的体积描述。Lengyel等人[4]使用八种纹理来表示表面上的皮毛。

其中，每个纹理都代表了在从表面一定距离通过一组毛发的切片(slice)。该模型被渲染了八次，每次都由顶点着色器程序沿着顶点法线将每个三角形稍微向外移动。从而使得每一个连续的模型都描绘了表皮以上不同的高度。以这种方式创建的嵌套模型被称为shells。这种渲染技术在物体轮廓的边缘会出现问题，因为毛发随着层的展开会逐渐分解成点。

#### 1.6.2基于几何着色器的渲染

几何着色器使得用皮毛挤压表面的多段线（polyline）皮毛成为可能，《失落的星球》使用了这种技术。表面渲染时，每个像素的值都被保存:皮毛颜色、长度和角度。 然后在几何着色器处理这幅图像，把每个像素变成半透明的polyline。渲染分两步进行。 在屏幕空间中指向下方的皮毛首先被渲染，以从屏幕底部到顶部的顺序。 这种方式可以从后往前正确地执行混合操作。最后，剩下的皮毛从上到下依次渲染。

参考文献：

[1] Tomas Akenine-Mller, Eric Haines, and Naty Hoffman. 2018. Real-Time Rendering, Fourth Edition (4th. ed.). A. K. Peters, Ltd., USA.

[2] Marschner, Stephen R., Henrik Wann Jensen, Mike Cammarano, Steve Worley, and Pat Hanrahan, “Light Scattering from Human Hair Fibers,” ACM Transactions on Graphics (SIGGRAPH 2003), vol. 22, no. 3, pp. 780–791, 2000.

[3] Yuksel, Cem, and Sara Tariq, SIGGRAPHAdvanced Techniques in Real-Time Hair Rendering and Simulation course, July 2010.

[4] Lengyel, Jerome, Emil Praun, Adam Finkelstein, and Hugues Hoppe, “Real-Time Fur over Arbitrary Surfaces,” in Proceedings of the 2001 Symposium on Interactive 3D Graphics, ACM, pp. 227–232, Mar. 2001.

