# 简介

阴影分为两种，硬阴影和软阴影。Shadow mapping是常见的硬阴影生成方法，但由于硬阴影有着明显的分界线，真实感较低，所以实际应用中更常使用有过渡、更真实的软阴影。PCSS(Percentage closer soft shadows)就是一种生成软阴影的方法。

# 1. Shadow mapping

## 1.1 主要思想

shadow mapping是一个2-pass的算法，其主要思想为：如果一个点既能从光源处看到，也能在相机处看到，那么这个点就不在阴影中。它的实现主要分为两部分：首先，从光源看向场景， 在渲染场景时输出一个在光源处看到的场景的最浅深度图shadow map；然后，从相机角度看向场景，对场景中的每一个点得到该点到光源的距离，并与shadow map中对应的该点的深度值作比较，如果距离比shadow map中的深度值大，说明该点被遮挡，处于阴影中，如果两者相等，那么该点就未被遮挡。

<div align=center>![图1.1 shadow mapping示意图](https://renderwiki.github.io/ImageResources/Percentage Closer soft shadows/1.png)</div>

<center></center>

# 2. PCF(Percentage Closer Filtering)

## 2.1 主要算法

PCF是一种抗锯齿的方法，后来人们发现可以将其运用在生成软阴影上，于是有了PCSS。shadow mapping是通过比较点到光源的实际距离和shadow map中对应的该点的深度值来判定点是否在阴影中，例如两值相等，则比较结果为1，点不在阴影中；反之比较结果为0，点被遮挡。而PCF的主要工作是将一系列的比较结果求平均：

* 将shadding point与shadow map上对应的像素的深度作比较时，不止比较这一个像素，还要和shadow map上这个像素周围的一个区域内的像素的深度值都做比较。例如这个区域大小为3 * 3，则得到9个比较结果，

  <center>1， 0， 1，</center>

  <center>1， 0， 1，</center>

  <center>1， 1， 0，</center>

* 将所有比较结果取平均值，得到该点的visibility，这个值不再是非0即1，而是一个[0,1]区间内的数，从而对遮挡的结果做了filtering，实现了软阴影。

## 2.2 filter size的选取

从PCF的算法实现中看到，该方法是对一个区域的结果进行filter，这个区域的大小选取十分重要。如果size过小，如1*1，那相当于没有做任何改变，依旧是shadow map的做法，得到硬阴影；如果size过大，阴影会变得非常模糊。不难看出，filter size决定了阴影的软硬程度。

另外需要考虑的一个问题是，每一个点所应用的filter size大小是不同的。如图2.1，在靠近笔尖的地方，阴影偏硬，而越往笔的上部，阴影越软。所以，filter size还与阴影的接收物（如纸）到阴影的投射物（如笔）的距离的远近有关。

<div align=center>![图2.1 阴影变化图](https://renderwiki.github.io/ImageResources/Percentage Closer soft shadows/2.png)</div>

<center> </center>

# 3 PCSS(Percentage closer soft shadows)

若想实现软阴影，应该对每一点有不同的filter size，且filter size与blocker distance（即遮挡物与阴影的接收物的距离）有关。

## 3.1 Filter size

<div align=center>![图3.1 距离关系图](https://renderwiki.github.io/ImageResources/Percentage Closer soft shadows/3.png)</div>

<center></center>

<math>d_b</math>是遮挡物blocker到光源的垂直距离，<math>d_r</math>是阴影接收物到光源的垂直距离，<math>W_{penumbra}</math>称为半影距离或filter size。从图中可以看到黄色虚线构造了两个相似三角形，根据相似三角形原理，Blocker离Receiver越近，<math>W_{penumbra}</math>越小；Blocker离Receiver越远，离光源越近，<math>W_{penumbra}</math>越大。结合图2.1，不难得出结论：<math>W_{penumbra}</math>一定程度上表示软阴影的程度。

<center><math>W_{penumbra}=(d_r-d_b)*W_{light}/d_b</math></center>

根据上式，filter size的大小取决于<math>d_b</math>和light size。其中，当光源为面光源时是不能生成shadow map的，为了模拟面光源的软阴影，通常将相机放在面光源中间，用点光源的方式生成shadow map。为了得到filter size，需要求出<math>d_b</math>。

## 3.2 主要算法

* step 1: 对于每一个shading point，在获得其到点光源的实际距离后，在shadow map所对应的像素周围，取一个区域，对区域中每个像素挨个判断点是否在阴影里，若在阴影里，说明该像素是blocker，记录该像素的深度值。记下区域中所有blocker的深度值后，求平均，得到<math>d_b</math>。
* step 2: 根据公式求filter size。
* step 3: PCF。

在步骤1中，需要取一个区域来做深度平均，可以固定区域大小为5*5，也可以根据light到shading point的距离，若二者离得远，取较小区域范围，二者离得近或light范围较大时，取较大区域范围。



## 3.3 实现效果

<div align=center>![图3.2 PCSS效果图](https://renderwiki.github.io/ImageResources/Percentage Closer soft shadows/4.png)</div>


树根离地面近，阴影偏硬，而树叶离地面远，离光源近，得到的阴影软。



参考文献：

[1]Randima Fernando (2005). Percentage-closer soft shadows International Conference on Computer Graphics and Interactive Techniques.