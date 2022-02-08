## 简介

反射对于全局光照的计算来说十分重要，而反射属于间接光照的范畴，在实时渲染中计算间接光照下的反射是十分困难的，因此需要使用各种技术来获得近似的反射效果，一些常使用的技术包括环境贴图反射、IBL反射和屏幕空间反射。

屏幕空间反射（SSR，全称Screen Space Reflection），是一种基于屏幕空间技术处理反射的方法，该技术最初由Crytek在2011年的siggraph的course《Secrets of CryEngine 3 Graphics Technology》中提出。该算法基本流程如下：

（1） 对于屏幕空间的物体的每个像素，根据该像素所对应的法线和视线信息，求得反射光线的方向；

（2） 根据当前像素的反射方向进行屏幕空间的光线步进（RayMarching），判断步进后的像素深度和深度缓存中存储的物体的深度是否相交；

（3） 若判断为相交，则取该点的颜色作为起始点的反射颜色。

该技术比较适合具有延迟渲染的管线，因为延迟渲染可以直接提供法线和深度信息，而前向渲染的管线需要单独设置一个pass来获取这些信息。

以下是每个步骤的实现方法。

## 1.RayMarching

SSR采用在屏幕空间的光线步进，通过深度来判断是否相交，若相交，则该点的颜色作为最终的反射颜色，若不相交，则直接步进到超出屏幕空间为止。

上述的光线步进方法是二维空间的方法，该方法于2014年由Morgan McGuire等人提出[1]，该方法避免了三维空间下光线步进一些缺点，首先解释一些三维空间下光线步进存在的一些问题。

RayMarching的过程如下：

（1） 对于着色点x，根据该点法线、坐标和摄像机坐标可计算出光线的反射方向；

（2） 以着色点x为起点，沿反射方向每次步进一段距离，然后得到一个新的着色点位置，记作xi，由![](http://latex.codecogs.com/svg.latex?xi = x + i*delta(l))计算得到；

（3） 将得到的新着色点xi投影到屏幕空间，获得对应的UV坐标；

（4） 根据获得的UV采样深度缓冲区，将得到的深度与xi点本身的深度做对比，判断是否该采用该点作为反射的颜色。

但是问题在于，将获取的新着色点投影到屏幕空间时，会出现以下两种情况：

<div align=center>![Figure1.1](https://renderwiki.github.io/ImageResources/SSR/Figure1.1.png)</div>

<center>图1.1 过采样</center>

当每次步进获得的着色点都对应于一个像素时，会出现过采样的问题。

<div align=center>![Figure1.2](https://renderwiki.github.io/ImageResources/SSR/Figure1.2.png)</div>

<center>图1.2 欠采样</center>

而每次步进得到的着色点在投影后横跨多个像素时，会出现欠采样的问题。

因此采用在二维空间下进行光线步进的方法来解决上述所提出的问题。

二维空间下的光线步进是对连续的像素点采样，可以有效地解决过采样和欠采样的问题，如图1.3所示：

<div align=center>![Figure1.3](https://renderwiki.github.io/ImageResources/SSR/Figure1.3.png)</div>

<center>图1.3 连续的像素点采样</center>

并具有以下特性：

（1） 每个采样的像素将与上一个相邻或在对角方向；

（2） 对每个像素最多采样一次；

（3） 光线会限制在视锥之内；

（4） 该方法通过最小化寄存器消耗、减少分支判断等高效地利用GPU资源。

对像素的采样使用DDA算法，该算法是在x方向每次步进一个像素，y反向根据反射方向的斜率确定，具体是由以下公式确定：

<div align=center>![公式2](https://renderwiki.github.io/ImageResources/SSR/公式2.png)</div>

这样的像素采样方法将不会出现过采样和欠采样的问题。

## 2.相交测试

在判断相交测试的时候，需要考虑到像素的厚度，并且对采样的深度进行适当的偏移，这里两个参数可以根据具体场景进行调节，添加这两个参数可以避免出现不连续的artifacts，shader的伪代码实现如下：

```c++
for (int i = 0; i < MaxLinearStep; ++i)

{

  // 光线步进

  Ray += Step;

  // 到达边界，没有相交

  if (Ray.z < 0 || Ray.z > 1)

​    return false;

  // 采样深度

  Depth = SceneDepthZ.SampleLevel(PointSampler, Ray.xy, 0).x;

  // 相交测试

  if (Depth + PerPixelCompareBias < Ray.z && Ray.z < Depth + PerPixelThickness)

  {

​    // 返回相交的UV和深度

​    OutHitUVz = Ray;

​    return true;

  }

}
```

参考文献：

[1] Mara M . Efficient GPU Screen-Space Ray Tracing. 