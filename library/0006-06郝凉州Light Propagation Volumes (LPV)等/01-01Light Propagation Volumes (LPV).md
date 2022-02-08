## 简介

Light Propagation Volumes（LPV）是一种实时的、无需任何预计算的全局光照技术，其在RSM和SH的基础上创造性地提出了使用Volumes来存储、传播间接光照的方法。

## 1.基本思想

假设Radiance沿直线传播并且传播过程中保持不变，将整个场景划分为一个三维网格，让Radiance从被直接光照照亮的表面开始，在网格中传播，从而计算最终到达着色点的间接光照。

## 2.步骤
### 2.1生成次级光源
可以应用RSM等方法找到被光源直接照亮的表面，并定义一系列虚拟点光源，可以通过采样等方式适量减少虚拟点光源的数量。
<div align=center>![图2.1](https://renderwiki.github.io/ImageResources/LPV/LPV2.1.png)</div>
### 2.2注入
将整个场景划分为一个三维网格，某些网格单元中可能包含一个或多个虚拟点光源，每个网格单元中的虚拟点光源的光照在各个方向上的Radiance并不相同，假设这些虚拟点光源都在网格单元中心，将它们相加得到一个球面空间的二元函数，可以使用球谐函数来拟合，工业界一般认为两阶SH即可拟合。将拟合的结果作为每个网格单元Radiance的初始值。
<div align=center>![图2.2](https://renderwiki.github.io/ImageResources/LPV/LPV2.2.png)</div>

### 2.3传播
对于每个网格单元，在传播时可以传播上下左右前后共六个格子。传播过程中收集从其6个面中的每个面接收的辐射，并相加起来，再次使用球谐函数来拟合。重复此传播过程若干次，直到网格趋于稳定。
<div align=center>![图2.3](https://renderwiki.github.io/ImageResources/LPV/LPV2.3.png)</div>
### 2.4渲染
对于任意着色点，找到其所在的网格并获得所在网格中所有方向radiance后进行渲染即可。



## 3.优缺点


这种方法的重要优点是它为每个网格单元生成一个完整的辐射场。这意味着即使使用二阶球面谐波时光滑BRDF的反射质量相当低，我们仍可以使用任意的BRDF进行着色。
该方法与其他基于体积渲染的方法有相同的问题，其中最大的问题是漏光现象。
<div align=center>![图3.1](https://renderwiki.github.io/ImageResources/LPV/LPV3.1.png)</div>
对于如图3.1中的p点，其在一面薄墙的左侧，但是由于在一点上的出射光在注入时全部认为是网格单元中心发出的，所以p点被移动到了墙右侧，产生了漏光现象。

<div align=center>![图3.2](https://renderwiki.github.io/ImageResources/LPV/LPV3.2.png)</div>

不幸的是，增加网格分辨率来修复它会导致其他问题。当使用更小的单元网格尺寸时，需要更多的迭代来在相同的空间距离上传播，这使得该方法的成本大大增加。在网格的分辨率和性能之间找到一个平衡点并非易事。这种方法还存在走样问题。有限的网格分辨率，加之模糊的Radiance方向，在传播过程中可能导致信号损坏。多次迭代后可能出现空间伪影，其中一些问题可以通过在传播后执行空间滤波来消除。
## 4.拓展
为了允许光在更长的距离上传播，并增加体积所覆盖的面积，同时保持合理的内存使用，kapplanyan和Dachsbacher开发了该方法的级联变体。他们使用的不是具有统一大小的单元格的单一卷，而是一组单元格逐渐增大的卷，并相互嵌套。照明被注入到所有的层和传播独立。在查找期间，它们为给定的职位选择最详细的可用级别。最初的实现没有考虑任何遮挡的间接照明。修改后的方法使用了来自反射阴影地图的深度信息，以及相机位置的深度缓冲，以添加关于遮光器的信息到体量中。这个信息是不完整的，但场景也可以在预处理过程中体素化，因此使用更精确的表示。


参考文献：
Kaplanyan, Anton, “Light Propagation Volumes in CryEngine 3,” SIGGRAPH Advances in Real-Time Rendering in Games course, Aug. 2009.
《real-time rendering4》