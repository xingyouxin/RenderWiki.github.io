# 简介

​		VSSM中将区域内的深度值看作正态分布，但实际分布与正态分布相差较大时会造成一定的影响。为此，Moment Shadow mapping在VSSM的基础上，将分布描述的更加准确。

# 1 算法内容

# 1.1 Moments

​		为了更准确的描述分布，需要使用更高阶的Moments（矩），矩的阶数越高，分布越准确。矩的最简单的定义就是数的次方，例如VSSM中，用到了深度和深度的平方，因此VSSM就是用了前两阶矩。

# 1.2 MSM(Moment Shadow Mapping)

Peters等人提出，若保留前m阶矩，则可以描述一个有m/2个台阶的函数，如下图：

<div align=center>![图1.1 ](https://renderwiki.github.io/ImageResources/Moment Shadow Mapping/1.png)</div>



如图绿色线，保留了4阶矩，描述出了一个有两个台阶的函数。通常，4阶就足够描述较为准确的分布了。

* MSM和VSSM非常相似，唯一的区别在于分布估计上。
* 在MSM中，生成shadow map的同时，记录下Z,Z<sup>2</sup>,Z<sup>3</sup>,Z<sup>4</sup>



参考文献：

[1]Christoph Peters, & Reinhard Klein (2015). Moment shadow mapping Interactive 3D Graphics and Games.