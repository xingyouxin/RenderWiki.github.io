## 1.多级渐进纹理（Mipmapping）
### 1.1使用Mipmapping进行纹理过滤
纹理映射的一个基本的问题是抗锯齿。渲染纹理映射的图像是一个采样过程：将纹理映射到几何对象的表面上，然后将表面投影到图像中，将在整个图像空间上产生二维函数，渲染过程中通过以像素为单位进行采样，当图像包含细节或者锐利边缘信息时，这种点采样的策略会产生锯齿失真。

抗锯齿的解决方案有：将之前基于点的采样变为基于图像范围的采样。如果我们有足够的样本，像素区域内的许多样本落在纹理中的不同位置，通过这些样本区域的平均值，从而估算得到纹理区域的平均颜色值，但这种方式需要大量的采样点，为了得到很好的结果需要较长的时间，因此，得到纹理中对应区域颜色值的平均是十分重要。

渲染图像和纹理图像的关系是不断变化的。通常情况下，每个像素对应单个平面，相当于对表面上的区域求平均，如果这个表面的颜色来自纹理，那么就需要对纹理对应区域取平均，纹理上对应的区域被称为纹理覆盖区(Texture Footprint)。而抗锯齿问题的关键就变为了纹理覆盖区的颜色值的平均。

多级渐进纹理技术，也就是Mipmapping被提出用来解决或缓解锯齿问题，这种技术的基本思想是：在不同大小和位置的各个区域上预先计算并存储纹理的平均值，因此Mipmap纹理是一系列纹理，这些纹理描述的是同一个图像的不同程度的近似，分辨率越来越低。初始Mipmap纹理是层级0级，层级1级的Mipmap纹理是在0级纹理的基础上在每个维度上降采样两倍得到的，从而具有0级图像像素的1/4，继续这个降采样过程，得到不同层级的Mipmap。

### 1.2Mipmapping的误差

使用Mipmap可以更加高效的完成纹理过滤。当我们需要一个大面积平均纹理值时，我们使用更高层级的Mipmap，因为这些值已经是预计算得到的图像在大面积上的平均值。因此最简单的方法是：从Mipmap查找单个值，然后选择层级，从而使得该纹素对应的纹理空间面积和纹理覆盖区的大小大致相同，当然，纹理覆盖区的形状可能和纹素表示的形状（正方形）不完全相同，这也是Mipmap会导致部分失真的原因。

### 1.3Mipmapping层级选择

当纹理覆盖区的宽度为 ![](http://latex.codecogs.com/svg.latex?D)（通常取纹理覆盖区较长轴最为宽度，从而近似成一个正方形）时，需要决定哪个层级的Mipmap适合采样，由于第 ![](http://latex.codecogs.com/svg.latex?k) 级的纹素覆盖了宽度为 ![](http://latex.codecogs.com/svg.latex?2^{k})的正方形，因此存在着以下关系：

![](http://latex.codecogs.com/svg.latex?D=max({|u_{x}|,|u_{y}|}),)

<math>D=max({|u_{x}|,|u_{y}|}), \tag{1}</math>

![](http://latex.codecogs.com/svg.latex?D \approx 2^{k}.)

<math>D \approx 2^{k}. \tag{2}</math>

所以可以得到 ![](http://latex.codecogs.com/svg.latex?k=log2(D)),当然在大多数情况下 ![](http://latex.codecogs.com/svg.latex?k) 并不是一个整数，但是我们只存储了整数级别的Mipmap图像，有两种解决方案，其一是找到最接近 ![](http://latex.codecogs.com/svg.latex?k) 的整数层级Mipmap图像，但是这种突然的过渡会带来明显的不连续或接效果；其二是针对最近的 ![](http://latex.codecogs.com/svg.latex?k) 的上下两个层级，进行线性插值。

基本的Mipmapping能够很好的消除锯齿，但是由于它无法处理和正方形形状差异较大的纹理覆盖区，对于细长或各向异性纹理覆盖区，在掠射角度观察时，效果并不理想。


参考文献：

[1] Marschner S ,  Shirley P . Fundamentals of computer graphics. 4th edition.[J]. World Scientific Publishers Singapore, 2009, 9(1):.
[2] Tomas Akenine-Moller, Eric Haines, Naty Hoffman .Real-Time Rendering 4rd.
