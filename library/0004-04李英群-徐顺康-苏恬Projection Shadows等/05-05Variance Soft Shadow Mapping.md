# 简介

PCSS的算法步骤中，需要shading point和shadow map区域中每个像素进行比较，这大大增加了算法开销，为了解决这个问题，Yang等人[1]提出了VSSM算法。

# 1 VSMM(Variance Soft Shadow Mapping)

PCSS的实现步骤如下：

* step 1: 对于每一个shading point，在获得其到点光源的实际距离后，在shadow map所对应的像素周围，取一个区域，对区域中每个像素挨个判断点是否在阴影里，若在阴影里，说明该像素是blocker，记录该像素的深度值。记下区域中所有blocker的深度值后，求平均，得到![](http://latex.codecogs.com/svg.latex?d_b)。
* step 2: 根据公式求filter size。
* step 3: PCF。

​	在PCF中，需要shading point与shadow map中的某个区域范围内的像素深度值依次进行比较，记录比较结果再对比较结果进行平均，这一过程等价于求该区域中深度值比实际距离小的像素所占的百分比，也等价于求该区域中有多少像素的深度值比实际距离小。

​		PCSS的求解方法是对每个像素都比较一次，因此导致开销大。而VSMM则是将区域内的深度值分布当作正态分布来近似估计。

## 1.1 算法思想

* step 1: 定义一个正态分布需要均值和方差，因此必须要能够迅速求出区域内深度值的均值和方差。

    * 求均值：
      * MIPMAP：MIPMAP是一种快速的、近似的范围查询方法，但是由于其还有一个特点是范围必须是方形，因此在这里并不算很好的方法。
      * SAT(Summed Area Table)

    * 求方差：

      ​									![](http://latex.codecogs.com/svg.latex?Var(x)=E(x^2)-E^2(x))

​		其中，![](http://latex.codecogs.com/svg.latex?E^2(x))可以在求出均值后获得，而![](http://latex.codecogs.com/svg.latex?E(x^2))则是在生成shadow map的同时，生成一张“square-depth map”，其每个texel的值为深度的平方，然后对其求均值。

* step 2: 得到均值和方差后，获得该区域内深度值的正态分布，求该区域内有多少像素的深度值比实际距离小。

  * 方法一：求面积。

    <div align=center>![图1.1 面积法求值](https://renderwiki.github.io/ImageResources/Variance Soft Shadow Mapping/1.png)</div>

    <center></center>

  * 方法二：利用切比雪夫不等式：对于任意分布，在已知其均值和方差后，可以求出随机变量的值超过某个值t的概率。

    <div align=center>![图1.2 切比雪夫公式](https://renderwiki.github.io/ImageResources/Variance Soft Shadow Mapping/2.png)</div>

    <center></center>

    在这里，将不等式作为约等式来近似概率，从而得到P(x<t)=1-P(x>t)。但切比雪夫不等式在使用时有一个条件：t必须在均值的右边。

* step 3: 简化了第三步PCF后，还需简化第一步，求区域内遮挡物的![](http://latex.codecogs.com/svg.latex?d_b)的平均值。

  已知shading point到光源的实际距离为t，则shadow map对应区域范围内，所有深度值z小于t的像素都属于blocker，记它们的平均深度为![](http://latex.codecogs.com/svg.latex?Zocc),不是blocker的像素记其平均深度为![](http://latex.codecogs.com/svg.latex?Zunocc).

  <div align=center>![图1.3 shadow map例图](https://renderwiki.github.io/ImageResources/Variance Soft Shadow Mapping/3.png)</div>

​		例如图1.3，假设t=7，则所有蓝色像素为blocker，蓝色数字的平均和为![](http://latex.codecogs.com/svg.latex?Zooc).

​		令N1为非blocker的像素个数，N2为blocker的像素个数，N为区域内像素总个数，Zavg为区域内深度平均值(通过一次查询即可得到)，利用下式求![](http://latex.codecogs.com/svg.latex?Zocc)

<div align=center>![图1.4 求解Zocc公式](https://renderwiki.github.io/ImageResources/Variance Soft Shadow Mapping/4.png)</div>



​		其中，N1/N是非blocker像素所占的比例，即比较结果为1的像素所占的比例，也即P(x>t)的值；N2/N是blocker像素所占的比例，即1-P(x>t)。

​		上式中，还有一个未知数Zunocc，VSMM中做了一个大胆的假设：Zunocc=t。

​		至此，Zocc的快速求解方法也得到了。

## 1.2 SAT(Summed Area Table)

SAT是一种利用前缀和的范围查询方式，范围查询即在区域内求平均。

* 在一维数组中：

  <div align=center>![图1.5 一维数组SAT构造图](https://renderwiki.github.io/ImageResources/Variance Soft Shadow Mapping/5.png)</div>

SAT的每个元素都是原本的一维数组从最左加到该元素下标所对应的元素的累加和，如SAT[2]=Input[0]+Input[1]+Input[2]。

* 在二维数组中：给一个任意矩形，则矩形内所有值的和可以通过矩形的加减得到。（所有矩形都以左上角为起点）

  <div align=center>![图1.6 二维数组SAT构造图](https://renderwiki.github.io/ImageResources/Variance Soft Shadow Mapping/6.png)</div>

  

  因此，二维SAT中的每个元素值=以元素的行列值为范围，在该范围中从左上角各个元素依次相加直到加到该元素后所得的累计和。所以通过构建SAT可以得到区域累计和，从而得到平均值。
  
  构建二维SAT的方式是先每一行做一次SAT，再每一列做一次SAT。



参考文献：

[1]Baoguang Yang, Zhao Dong, Jieqing Feng, Hans-Peter Seidel, & Jan Kautz (2010). Variance Soft Shadow Mapping Computer Graphics Forum, 29, 2127-2134.