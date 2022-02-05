## 简介

当艺术家对线条图进行着色时，他们通常使用低强度的着色方式来赋予物体某些颜色，从而突出表现物体表面上的某些曲线特征（Gooch, Gooch, Shirley, & Cohen, 1998）。一般朝着曲面的一个方向用冷色（如蓝色）着色，朝着相反方向用暖色（如橙色）着色。通常来说这些颜色饱和度并不高，同时也不会很暗淡。按照这种方式可以很好的呈现出黑色轮廓线，整体上表现出一种卡通效果。

## 1.Cool-to-Warm Shading

### 1.1着色方法

可以使用一个指向”暖光源”的方向$l$以及法线$n$与$l$的余弦来调制颜色值，其中暖光常数$k_w$的取值在$[0,\ 1]$之间：

$k_w=\frac{1+\pmb{n·l}}{2}. \tag{1}$

颜色$c$通过冷色$c_b$和暖色$c_w$之间的线性混合得到：

$c=k_wc_w+(1-k_w)c_b. \tag{2}$

有许多组可能的冷色$c_b$和暖色$c_w$搭配来产生不同的外观效果。下面给出一个例子：

$$c_c=(0.4,\ 0.4,\ 0.7), \tag{3}$$

$$c_c=(0.8,\ 0.6,\ 0.6). \tag{4}$$

### 1.2效果展示

图xxx展示了传统的Phong光照着色和这种类型的艺术着色之间的比较。

![传统的Phong光照着色和这种类型的艺术着色之间的比较](https://renderwiki.github.io/ImageResources/shading model/传统的Phong光照着色和这种类型的艺术着色之间的比较.png)

图xxx 左：Phong光照模型的效果；中：冷色到暖色的艺术着色效果，未绘制轮廓线；右：冷色到暖色的艺术着色效果，绘制轮廓线。*Image courtesy Amy Gooch.*



参考文献：

[1] Marschner S ,  Shirley P . Fundamentals of computer graphics. 4th edition.[J]. World Scientific Publishers Singapore, 2009, 9(1):239-240.
