## 简介

RSM是目前主要的全局光照算法之一，它是Shadow Maps方法的延伸，它将直接光源照亮的区域作为次级光源，产生间接光照，将直接光照与间接光照融合后形成全局光照的效果。

## 1.Reflective Shadow Maps

### 1.1 RSM基本思想
   &emsp;RSM的核心思想是将接收了直接光源照亮的区域作为发光物（次级光源）再照亮别的区域。   
   &emsp;使用RSM对场景进行渲染分为以下几步：   
   &emsp;1.单独计算直接光照。   
   &emsp;2.以光源的位置作为相机的位置进行渲染，在阴影贴图中存储深度值、位置信息、法线信息、反射光强度等。   
   &emsp;3.用第二步中光源所直接照亮的位置作为虚拟点光源，利用阴影贴图中存储的信息计算间接光照。   
   &emsp;4.将直接光照与间接光照相加得到最终效果。

### 1.2 计算公式
   &emsp;令着色点处和虚拟点光源表面各自法线为 n 和 np 后，着色点 x 收到的虚拟点光源 xp 的辐射强度如下所示：   
<div align=center>![辐射强度计算公式](https://renderwiki.github.io/ImageResources/Reflective Shadow Maps/Figure1.1.png)
</div><center>图1.1 辐射强度计算公式</center>   

### 1.3 优化
   &emsp;由于不是所有的虚拟点光源都对着色点有贡献，还需要考虑可见性、方向与距离的问题，因此可以通过有侧重地选取一些虚拟点光源进行计算来对RSM加速。   
   &emsp;将着色点投影到shadow map空间中，距离着色点越近的虚拟点光源采样多，但权重小；距离着色点远的虚拟点光源采样少，权重占比大。
<div align=center>![采样示意图](https://renderwiki.github.io/ImageResources/Reflective Shadow Maps/Figure1.2.png)
</div><center>图1.2 采样示意图</center> 

### 1.4 优缺点
优点：   
   &emsp;实现简单，使用shadow map的生成流程即可。   
缺点：   
   &emsp;1.随光源数量的增多，RSM的性能会逐渐下降。   
   &emsp;2.由于在计算过程中对间接光照没有可见性计算，并且假设多，因此在一些情况下会有不真实的现象。   
   &emsp;3.如果要进行采样，采样的速度会比较慢，要在效率和质量上进行平衡.   

参考文献：   
[1]  Dachsbacher, Carsten, and Marc Stamminger, “Reflective Shadow Maps,” in Proceedings of
the 2005 Symposium on Interactive 3D Graphics and Games, ACM, pp. 203–231, 2005. Cited
on p. 491   
[2] Carsten Dachsbacher, Marc Stamminger.Reflective Shadow Maps
