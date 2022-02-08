## 简介

预计算辐射传输（Precomputed Radiance Transfer）是一种计算机图形学技术，通过预先计算复杂的光路交互，以此节约时间来达到实时渲染场景的目的。

本质上，PRT 用入射辐照度的线性组合来表示一点的光照。必须使用一种有效的方法来编码这些数据，通常使用各类基函数，例如球谐函数（Spherical Harmonics），哈尔小波（Haar Wavelet），球面高斯（Spherical Gaussian）等。

## 1.Precomputed Radiance Transfer（PRT）

### 1.1发展历程

* PRT的起源论文<sup>[1]</sup>由Sloan等人于2002年撰写，通过球谐函数表示一个无穷远的照明环境。此论文分析了两种情况，一是标量辐照度与漫反射表面交互，二是需要渲染非兰伯特材质或需要使用法线贴图。在第二种情况下，由于需要保存渲染点在球体下的完整入射光分布，让它与BRDF进行卷积来得到正确的光照效果，因此其内存开销是庞大的。

* 在上一论文发布的一年后，Sloan等人撰写的新论文<sup>[2]</sup>着手解决了内存开销问题。通过对整体集合进行主成分分析（PCA）来压缩存储，以此替代直接存储传输向量或矩阵。

* 2005年，Sloan等人撰写了新论文<sup>[3]</sup>，提出了允许物体有限形变的拓展方法，称为local deformable precomputed radiance transfer（LDPRT）

* 同年，由于最初的PRT方法假设周围的照明距离无限远，对室内环境的限制太大。 Kristensen等人提出了一种方法<sup>[4]</sup>，对散布于场景中的一组光源计算PRT。接下来，灯光被组合成簇，接收物体被划分到不同区域，每个区域都受到不同灯光簇的影响。这个过程会显著压缩数据。在运行时，灯光产生的照明通过插值预先计算簇中最近的数据来近似。

* 之后，为了节省内存，Sugden和Iwanicki于2011年提出将量化后的球谐传输系数存储为索引<sup>[5]</sup>。Jendersie等人于2016年提出构建层次结构，并在孩子节点对应立体角太小时存储对高层节点的引用<sup>[6]</sup>。同样在2016年，Stefanov等人引入了中间步骤，即将来自表面元素的辐射先传播到场景的体素化表达中<sup>[7]</sup>。

* 表面的理想分割取决于接收者的位置。对于遥远的元素，将它们视为独立的实体会产生不必要的存储成本，但是当近距离查看时，它们应该被单独处理。层次结构在某种程度上减轻了这个问题，但并不能完全解决它。因此， Silvennoinen和Lehtinen于2017年提出一种新式的方法<sup>[8]</sup>，为每个接收位置生成一个不同的源补丁集。

* 除了上述的PRT方法之外，其他一些方法也被提出。Lehtinen等人于2008年提出了源元素和接收元素都存于体积上，可以在三维空间中的任何位置进行查询的方法<sup>[9]</sup>。在2011年，Loos等人使用有侧壁的模块化单元进行PRT并近似场景几何形状<sup>[10]</sup>。

### 1.2原理

<div align=center>![RenderEquation](https://renderwiki.github.io/ImageResources/PRT/RenderEquation.png)</div>

<center>图1.2.1 PRT基础思想（来自GAMES202）</center>

PRT把渲染方程分成lighting和light transport（包括brdf和visibility）两项，将lighting用基函数表示，在预计算时可以把lighting算好。同时，PRT假设在一个环境中只有光照变化，那么对于任意的shading point，light transport就是不变的，因此可以投影到基函数上。在实时运行时，通过lighting和light transport的卷积就可以得到照明结果。



#### 1.2.1渲染Diffuse物体

<div align=center>![DiffuseCase](https://renderwiki.github.io/ImageResources/PRT/DiffuseCase.png)</div>

<center>图1.2.2 Diffuse情况（来自GAMES202）</center>

在diffuse的情况下，brdf是一个常数，可以从积分里提出来。把lighting项用基函数来表示，这样就能把它从积分里拿出来，那么积分中剩下来的就是light transport乘以一个基函数，这就相当于light transport投影到这个基函数，对这个系数进行预计算，实时运行时只计算一个点乘就能还原光照。

#### 1.2.2渲染glossy物体

<div align=center>![GlossyCase](https://renderwiki.github.io/ImageResources/PRT/GlossyCase.png)</div>

<center>图1.2.3 Glossy情况（来自GAMES202）</center>

Glossy材质的brdf就是完整的4维brdf，给定不同的入射方向会有不同的结果，因此light transport需要用矩阵来表示，而不能像之前一样使用向量。由于矩阵的原因，最终还原光照时就要做light向量和transport矩阵的相乘。

参考文献：

[1] Peter-Pike Sloan, Jan Kautz, and John Snyder. Precomputed radiance transfer for real-time rendering in dynamic, low-frequency lighting environments[J]. ACM Transactions on
Graphics (SIGGRAPH 2002), vol. 21, no. 3, pp. 527–536, July 2002.

[2] Peter-Pike Sloan, Jesse Hall, John Hart, and John Snyder. Clustered principal components for precomputed radiance transfer[J]. ACM Transactions on Graphics (SIGGRAPH 2003),
vol. 22, no. 3, pp. 382–391, 2003.

[3] Peter-Pike Sloan, Ben Luna, and John Snyder. Local, deformable precomputed radiance
transfer[J]. ACM Transactions on Graphics (SIGGRAPH 2005), vol. 24, no. 3, pp. 1216–1224,
Aug. 2005.

[4] Kristensen, Anders Wang, Tomas Akenine-Mller, and Henrik Wann Jensen. Precomputed local radiance transfer for real-time lighting design[J]. ACM Transactions on Graphics (SIGGRAPH 2005), vol. 24, no. 3, pp. 1208–1215, Aug. 2005.

[5] Sugden, B., and M. Iwanicki. Mega meshes: modelling, rendering and lighting a World
made of 100 billion polygons[R]. Game Developers Conference, Mar. 2011.

[6] Jendersie, Johannes, David Kuri, and Thorsten Grosch. Precomputed illuminance composition for real-time global illumination[J]. Proceedings of the 20th ACM SIGGRAPH
Symposium on Interactive 3D Graphics and Games, ACM, pp. 129–137, 2016.

[7] Stefanov, Nikolay. Global illumination in Tom Clancy’s The Division[R]. Game Developers
Conference, Mar. 2016.

[8] Silvennoinen, Ari, and Jaakko Lehtinen. Real-time global illumination by precomputed
local reconstruction from sparse radiance probes[J]. ACM Transactions on Graphics (SIGGRAPH Asia 2017), vol. 36, no. 6, pp. 230:1–230:13, Nov. 2017.

[9] Lehtinen, Jaakko, Matthias Zwicker, Emmanuel Turquin, Janne Kontkanen, Fr´edo Durand, Fran¸cois Sillion, and Timo Aila. A meshless hierarchical representation for light transport[J]. ACM Transactions on Graphics, vol. 27, no. 3, pp. 37:1–37:9, 2008.

[10] Loos, Bradford J., Lakulish Antani, Kenny Mitchell, Derek Nowrouzezahrai, Wojciech Jarosz, and Peter-Pike Sloan. Modular radiance transfer[J]. ACM Transactions on Graphics, vol. 30, no. 6, pp. 178:1–178:10, 2011.

[11] Tomas Akenine-M¨oller, Eric Haines, Naty, Hoffman, Angelo Pesce, Micha l Iwanicki, S´ebastien Hillaire. Real-Time Rendering Fourth Edition[M]. A K Peters/CRC Press, 2018:478-484

[12] 闫令琪. GAMES202: 高质量实时渲染[Z]. 2021:Lecture6-Lecture7