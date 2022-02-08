## 简介

屏幕空间环境光遮蔽(SSAO)是一种实时有效地逼近环境遮挡效果的计算机图形学技术。它是Vladimir Kajalin在Crytek工作时开发的，2007年，同样由Crytek开发的电子游戏《孤岛危机》(Crysis)首次使用了它。

该算法以像素着色器的形式实现，对存储在纹理中的场景深度缓冲区进行分析。对于屏幕上的每个像素，像素着色器采样当前像素周围的深度值，并试图从每个采样点计算遮挡量。在其最简单的实现中，遮蔽因子只取决于采样点和当前点之间的深度差。

## 1.SSAO (Screen Space Ambient Occlusion)

### 1.1环境光遮蔽理论

环境光遮蔽(Ambient Occlusion, AO)本质上是对全局光照的近似，用于计算场景中每个点对环境光照的暴露程度。例如，一个管子的内部通常比暴露在外的表面更封闭(因此颜色更深)，而且管子越往里越黑。尽管AO对该效应的物理基础进行了相当程度的简化，但结果看起来可信的令人惊讶。当光线缺乏方向变化和不能显示物体细节时，这种方法可以廉价地提供关于形状的细节。

AO的顶层想法可以总结为三点：

1. 对于一个着色点来说，假设其接收到来自四面八方的间接光照是常数。

2. 显示的考虑visibility function，即，一个着色点因为其周围的几何关系，有些方向的间接光照被遮挡而无法被接收到。

3. 限定材质为diffuse。

AO的理论背景可以从渲染方程推导得出。首先，可以对渲染方程进行近似，把它拆成两项：

<div align=center>![](https://renderwiki.github.io/ImageResources/SSAO/SSAO-渲染方程近似.svg)</div>

<!-- $$
\begin{aligned}
L_o(p,\omega_o) &= \int_{\Omega^+} L_i(p,\omega_i) f_r(p,\omega_i,\omega_o) V(p,\omega_i) cos\theta_i \, \mathrm{d} \omega_i \\&
\approx \frac{\int_{\Omega^+} V(p, \omega_i) cos\theta_i \, \mathrm{d} \omega_i}{\int_{\Omega^+} cos\theta_i \, \mathrm{d} \omega_i} \cdot \int_{\Omega^+} L_i(p,\omega_i) f_r(p,\omega_i,\omega_o) V(p,\omega_i) cos\theta_i \, \mathrm{d} \omega_i \\&
= \frac{\int_{\Omega^+} V(p, \omega_i) cos\theta_i \, \mathrm{d} \omega_i}{\pi} \cdot L_i(p) \cdot \frac{\rho}{\pi} \cdot \pi \\&
= k_A \cdot L_i(p) \cdot \rho
\end{aligned}
$$ -->

其中，<div>![](https://renderwiki.github.io/ImageResources/SSAO/SSAO-常数项.svg)</div>是AO的常数项，![](http://latex.codecogs.com/svg.latex?k_A)表示着色点各个方向上visibility的加权平均值。之所以可以如此近似拆分，是AO假设了着色点接收的间接光照为常数，在这种情况下，近似拆分可认为是准确的。

### 1.2屏幕空间

可以在对象空间(object space)内计算AO中的![](http://latex.codecogs.com/svg.latex?k_A)项，但是其计算开销与场景复杂度成正比。通常，一些关于遮挡的信息可以纯粹从屏幕空间(screen space)数据中推导出来，这些数据已经可用，比如深度和法线。这种方法有一个固定的成本，与场景的详细程度无关，而只与渲染所用的分辨率有关。

### 1.3屏幕空间环境光遮蔽算法

通常，AO将计算![](http://latex.codecogs.com/svg.latex?k_A)的范围被限定在着色点法线方向上半径为R的半球内。然而由于早期的渲染管线中比较难拿到屏幕空间的物体法线信息，因此SSAO是这样进行的：

1. 任何着色点都在以它为中心半径为R的球内采样若干个点。

2. 将这些采样点投影到相机空间，得到它们的深度，与屏幕空间中的深度图比较。若采样点的深度大于屏幕空间中对应位置的深度，则认为改采样点被遮挡。

3. 只有当超过半数的采样点都被认为是遮挡时，才开始考虑AO。计算被遮挡的点占所有采样点的比率，作为![](http://latex.codecogs.com/svg.latex?k_A)。

### 1.4屏幕空间环境光遮蔽的优缺点

与其他AO解决方案相比，SSAO具有以下优点：

- 独立于场景复杂性。

- 无需数据预处理，无需加载时间，系统内存中无需分配内存。

- 适用于动态场景。

- 对屏幕上的每个像素都以相同的一致方式工作。

- 没有CPU使用，它可以完全在GPU上执行。

- 可以很容易地集成到任何现代图形管线上。

SSAO也有以下缺点:

- 很难在深度不连续的情况下得到正确的结果，例如物体边缘。

- 采样点个数的选择，少则存在噪声，多则计算时间过长。可以采取先计算少量的采样点，然后对得到的结果做去噪。

- 没有真正考虑着色点法线方向的半球，计算不够准确。现代图形管线可以很容易得到屏幕空间的法线信息，因此出现了HBAO来解决SSAO存在的问题。

参考文献：

[1] [Finding Next Gen – CryEngine 2](https://web.archive.org/web/20090219082501/http://delivery.acm.org/10.1145/1290000/1281671/p97-mittring.pdf?key1=1281671&key2=9942678811&coll=ACM&dl=ACM&CFID=15151515&CFTOKEN=6184618)

[2] REAL-TIME RENDERING, FOURTH EDITION - Chapter 11.3

[3] [GAMES202 - Lecture 08](https://sites.cs.ucsb.edu/~lingqi/teaching/resources/GAMES202_Lecture_08.pdf)

[4] [Wikipedia - Screen space ambient occlusion](https://en.wikipedia.org/wiki/Screen_space_ambient_occlusion)

[5] [Wikipedia - Ambient occlusion](https://en.wikipedia.org/wiki/Ambient_occlusion)

[6] [Image-Space Horizon-Based Ambient Occlusion](http://artis.inrialpes.fr/Membres/Olivier.Hoel/ssao/nVidiaHSAO/2317-abstract.pdf)
