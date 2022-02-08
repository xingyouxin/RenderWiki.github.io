## 简介

   &emsp;Shadow Maps是一项在生成阴影，形成高真实感渲染效果的重要内容，它的主要思想是将相机的位置作为光源的位置，那么光无法“看”到的部分就是阴影部分。通过从相机的位置对场景进行渲染时渲染点与shadow map中的点之间内容的比较，得出渲染点的可见性，从而形成阴影的效果。

## 1.Shadow Maps

### 1.1基本定义

**shadow map（阴影贴图）**一般指z缓冲区中的内容，有时也称为阴影深度贴图或阴影缓冲区。其中，z缓冲区中的每个像素都是最接近光源的对象的深度。   
**omnidirectional shadow maps（全向阴影贴图）**是指当光源为点光源时生成的阴影贴图。一般点光源会分别向六个方向生成阴影贴图，然后构成cubemap。   
**cascaded shadow maps**：见1.2.2   
**Percentage-Closer Filtering(PCF)**：见1.3   
**VSM、CSM、ESM**：见1.4   
**Opacity shadow maps、Adaptive volumetric shadow maps**：见1.5   
**Irregular Z-Buffer**：见1.6

### 1.2 Shadow Mapping

#### 1.2.1 基本步骤
使用shadow map对场景进行渲染由以下两步构成：
  
1.将光源的位置作为相机的位置对场景进行渲染，生成shadow map   
   &emsp; 从光源的位置渲染场景，生成并存储shadow map，记录此时的投影变换矩阵M。其中，点光源对应透视投影，定向光对应正交投影。
   
2.将“人眼”的位置作为相机的位置对场景进行渲染   
   &emsp; 用矩阵M将场景中的三维点变换为二维坐标(x,y)和深度z，将z与shadow map中对应点的深度c(x,y)进行比对，判断此点是否在阴影中。
<div align=center>![shadow mapping示意图](https://renderwiki.github.io/ImageResources/shadow maps/Figure1.1.png)</div>   
<center>图1.1 shadow mapping示意图</center>   

#### 1.2.2 优化   
1.针对阴影痤疮问题的优化   
   &emsp;由于深度的数值精度和阴影贴图分辨率都有限，所以在进行深度值比较的时候会产生阴影痤疮(shadow acne)的问题，也可以称为自阴影混叠(self-shadow aliasing）   
   &emsp;解决方案：在比较的时候添加偏差——depth bias   
   &emsp;但在添加偏差时，由于偏差需要人为设定，因此在偏差设定过小时，会出现自阴影现象；而偏差设定过大时，会出现光泄漏问题(Peter Pan现象）
<div align=center>![depth bias图](https://renderwiki.github.io/ImageResources/shadow maps/Figure1.2.png)
</div><center>图1.2 引入偏差后的图像</center>  
   &emsp;解决自阴影的一种方法：second-depth shadow mapping。这种方法的思想是在shadow map中不仅存储最小深度，还额外存储次小深度,在进行后续的阴影比较时使用最小深度与次小深度之间的中间深度进行比较。
<div align=center>![second-depth shadow mapping](https://renderwiki.github.io/ImageResources/shadow maps/Figure1.3.png)
</div><center>图1.3 second-depth shadow mapping示意图（此时假设光源位于正上方）</center>

2.针对走样问题的优化  
 
   &emsp;解决方案1：**cascaded shadow maps (CSM)**  
   &emsp;CSM的基本思想是将相机的视锥体分割成若干部分，为分割的每一部分生成独立的阴影贴图，在使用时，近处的场景使用较高分辨率的阴影贴图，远处的场景使用较低分辨率的阴影贴图，在过渡的地方选择二者之一来使用   
<div align=center>![CSM](https://renderwiki.github.io/ImageResources/shadow maps/Figure1.4.png)
</div><center>图1.4 cascaded shadow maps示意图</center>

   &emsp;解决方案2：perspective warping   
   &emsp;在基础的shadow mapping中，由于近处的精度需求高于远处，所以会出现走样现象，perspective warping方法是在生成阴影贴图时，对矩阵M进行一定的扭曲，使阴影贴图更加匹配视锥体的形状，并且在近处分配了更多的分辨率，使走样问题得到一定的解决。
<div align=center>![透视扭曲对比图](https://renderwiki.github.io/ImageResources/shadow maps/Figure1.5.png)
</div><center>图1.5 perspective warping前后对比图，左图为基础的shadow mapping，右图使用了perspective warping方法</center>

   &emsp;解决方案3：shadow casters   
场景中的物体没必要都渲染到光源的视锥中，例如地面确定只会接受投影而不会产生投影，那么就不需添加到shadow map中。以光源为相机所产生的视锥可以被扩大或者缩小来安全地丢弃一些投影物体，调整视锥不仅可以节省渲染的时间，并且可以提高shadow map的分辨率。一般来说，以光源为相机所产生的视锥中近平面离光源越远越好，远平面离光源越近越好。
<div align=center>![shadow casters示意图](https://renderwiki.github.io/ImageResources/shadow maps/Figure1.6.png)
</div><center>图1.6 调整视锥大小选择投影物体</center>

### 1.3 Percentage-Closer Filtering   
#### 1.3.1 简介
   &emsp;**Percentage-Closer Filtering(PCF)**是在进行阴影判断时，从shadow map中混合多个像素的判断结果得到最终判断结果的想法。PCF最初用于反走样，后PCF的原理也常用于生成软阴影。   
#### 1.3.2 基本思想   
   &emsp;在shadow mapping的第二步中，进行阴影判断时，点(x,y)的判断结果不仅仅依靠z与c(x,y)的比较结果，而是对(x,y)周围一定范围内的像素都进行比较，再将所有的比较结果求均值（滤波操作）得到最终的可见性。   
   &emsp;当进行求均值操作的范围大的时候，阴影的效果就会趋向于软阴影；当求均值的范围小的时候，阴影的效果就会趋向于硬阴影。   
   &emsp;PCF在进行滤波操作时，可以选择双线性滤波、泊松轮盘滤波、自适应滤波等多种滤波核进行滤波。
### 1.4  Filtered Shadow Maps
#### 1.4.1 variance shadow map (VSM)
   &emsp;基本思想：在基础的shadow mapping中，深度比较的结果非1即0，而VSM利用了单边切比雪夫不等式近似得到可见性，使得在渲染时只需要知道shadow map中深度的均值和方差，就可以求出对应像素的可见性。在具体操作时，需要存储两张图：一张为存储深度的shadow map，另一张存储的是深度的平方，便于计算均值（也称为一阶矩 M1）和方差（也称为二阶矩 M2）。
<div align=center>![可见性计算公式](https://renderwiki.github.io/ImageResources/shadow maps/Figure1.7.png)
</div><center>图1.7 可见性计算公式。其中，t是着色点到光源的距离</center>
   &emsp;优缺点：优点是VSM的计算效率高，成本低；缺点是会出现漏光(light bleeding)的问题。   
   &emsp;优化：moment shadow mapping是对VSM的优化，它的基本思想是使用更高阶的矩来更精确地描述遮挡物的分布，但在得到较好的优化效果的同时，也耗费了巨大的存储成本和时间成本等。
#### 1.4.2 convolution shadow map (CSM)
   &emsp;基本思想：与VSM类似，CSM采用傅里叶展开来代替深度比较的结果。   
   &emsp;优缺点：优点是漏光现象有一定改善；缺点是由于傅里叶展开的截断，会出现振铃现象,并且存储成本和执行成本高。 
#### 1.4.3 exponential shadow map (ESM)
   &emsp;基本思想：将CSM中的傅里叶展开替换为指数函数即可。   
   &emsp;优缺点：ESM可以解决CSM的振铃问题，并且有高于VSM的质量，存储空间也比较低，故ESM在这三种方法中性能最好。  
### 1.5 Volumetric Shadow Techniques
   &emsp;Self-shadowing对于实现很小的或半透明的对象的真实感渲染十分重要，单深度阴影贴图不再适用于这种情况。Deep shadow maps中的阴影贴图中的每个像素存储了光如何随深度下降的函数，此函数通常由GPU通过一系列不同深度的样本进行计算。   
   &emsp;**Opacity shadow maps**是一种基于GPU的方法，阴影贴图中只存储了一组固定深度的不透明度。   
   &emsp;**Adaptive volumetric shadow maps**是为了避免依赖固定的截面设置，使用自适应技术提出的想法，其中每个阴影贴图像素都存储了不透明度和层深度。   
### 1.6  Irregular Z-Buffer Shadows
&emsp;在shadow mapping中，由于离散化的采样方式，会导致存在偏差问题，即表面采样的位置与看到的位置会稍有不同，目前有两种分析阴影的方法对此问题有所改进：   
  &emsp;1.Ray Tracing 光线追踪：射线从receiver的位置发射到光源，如果中间有物体挡住了光线，那么这个receiver就在阴影中。   
  &emsp;2.使用GPU的栅格化硬件来查看场景：除了深度值，在每个阴影贴图中还需要存储几何信息（三角形与其他数据），以使在采样的时候可以更精准地得到遮挡体的信息。   
&emsp;**Irregular Z-Buffer**不直接生成一个阴影纹理，而是直接把遮挡的几何信息传递给相机。可见性测试将在像素着色器中进行，从图像点的位置到光线处生成一条射线，如果射线与三角形所在平面的交点在三角形内，并且比三角形的平面更远，那么此点位于阴影内。


参考文献：   
[1] Kasyan, Nikolas, “Playing with Real-Time Shadows,” SIGGRAPH Efficient Real-Time Shadows course, July 2013. Cited on p. 54, 234, 245, 251, 264, 585   
[2] Wang, Yulan, and Steven Molnar, “Second-Depth Shadow Mapping,” Technical Report TR94-
019, Department of Computer Science, University of North Carolina at Chapel Hill, 1994.
Cited on p. 238   
[3] Engel, Wolfgang, “Cascaded Shadow Maps,” in Wolfgang Engel, ed., ShaderX5
, Charles River
Media, pp. 197–206, 2006. Cited on p. 242, 243   
[4] Tuft, David, “Common Techniques to Improve Shadow Depth Maps,” Windows Dev Center:
DirectX Graphics and Gaming Technical Articles, 2011. Cited on p. 236, 239, 240, 265   
[5] Reeves, William T., David H. Salesin, and Robert L. Cook, “Rendering Antialiased Shadows
with Depth Maps,” Computer Graphics (SIGGRAPH ’87 Proceedings), vol. 21, no. 4, pp. 283–
291, July 1987. Cited on p. 247   
[6] Donnelly, William, and Andrew Lauritzen, “Variance Shadow Maps,” in Proceedings of the
2006 Symposium on Interactive 3D Graphics, ACM, pp. 161–165, 2006. Cited on p. 252   
[7] Annen, Thomas, Tom Mertens, Philippe Bekaert, Hans-Peter Seidel, and Jan Kautz, “Convolution Shadow Maps,” in Proceedings of the 18th Eurographics Conference on Rendering
Techniques, Eurographics Association, pp. 51–60, June 2007. Cited on p. 255   
[8] Annen, Thomas, Tom Mertens, Hans-Peter Seidel, Eddy Flerackers, and Jan Kautz, “Exponential Shadow Maps,” in Graphics Interface 2008, Canadian Human-Computer Communications Society, pp. 155–161, May 2008. Cited on p. 256   
[9] Bavoil, Louis, “Advanced Soft Shadow Mapping Techniques,” Game Developers Conference,
Feb. 2008. Cited on p. 256   
[10]  Lokovic, Tom, and Eric Veach, “Deep Shadow Maps,” in SIGGRAPH ’00: Proceedings
of the 27th Annual Conference on Computer Graphics and Interactive Techniques, ACM
Press/Addison-Wesley Publishing Co., pp. 385–392, July 2000. Cited on p. 257, 258, 570, 638   
[11] Lecocq, Pascal, Pascal Gautron, Jean-Eudes Marvie, and Gael Sourimant, “Sub-Pixel Shadow
Mapping,” in Proceedings of the 18th Meeting of the ACM SIGGRAPH Symposium on Interactive 3D Graphics and Games, ACM, pp. 103–110, 2014. Cited on p. 259   
[12]  Sen, Pradeep, Mike Cammarano, and Pat Hanrahan, “Shadow Silhouette Maps,” ACM Transactions on Graphics (SIGGRAPH 2003), vol. 22, no. 3, pp. 521–526, 2003. Cited on p. 259
