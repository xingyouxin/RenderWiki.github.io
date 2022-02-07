# 简介
通过巧妙地运用stencil-buffer，可以将阴影投射到任意对象上。该算法可以在任何GPU上使用，因为只需要stencil-buffer，但由于其不可预测的开销，很少得到应用。
# 1.Shadow Volumes
## 1.1算法概述
<div align=center>![图1.1](https://renderwiki.github.io/ImageResources/Shadow Volumes/Figure2.1.png)</div>
<center>图1.1 左：点光源的线条通过三角形的顶点延伸，形成一个无限的三棱锥。右：上半部分是一个三棱锥，下半部分是一个截断的无限棱锥，也称为阴影体积。阴影体积内的所有几何体都在阴影中。</center>
如图1.1所示，将直线从一个点通过三角形的顶点延伸到无穷远处，就得到了一个无限延伸的三棱锥，三角形下方是一个截断的无限棱锥，上方是一个棱锥。假设这个点是一个点光源，位于截断棱锥体体积内（三角形下方）的物体都处于阴影当中，该体积被称为阴影体积。

当我们看向某个场景时，假设有一条光线从眼睛穿过像素，直至该光线击中某个要在屏幕上显示的物体。在这个过程中，每次在光线穿过阴影体积的正面（即朝向观察者）时使计数器加一，即每次光线进入阴影时，计数器增加。同样，每次光线穿过截断棱锥的背面时使计数器减一，此时光线穿过了该阴影。如上所述继续递增和递减计数器，直至光线击中要在该像素处显示的对象。如果此时计数器大于零，则该像素处于阴影中，否则不是。当存在多个三角形投射阴影时，该原理也适用，如图1.2所示。
<div align=center>![图1.2](https://renderwiki.github.io/ImageResources/Shadow Volumes/Figure2.2.png)</div>
<center>图1.2 </center>
使用光线进行这种操作会非常的耗时。有一个更聪明的解决方案：使用一个stencil-buffer进行计数。第一，清空stencil-buffer。第二，将整个场景绘制到帧缓冲区中，此时只使用未照明材质的颜色，将这些着色组件绘制到颜色缓冲区中，并将深度信息绘制到z-buffer中。第三，在关闭z-buffer的情况下（z-buffer测试仍在进行）更新颜色缓冲区，然后绘制阴影体积的前向三角形。在此过程中，stencil-buffer的操作设置为在绘制三角形的位置增加缓冲区中的值。第四，重复上一步，这次只绘制阴影体积的背面三角形，在这一过程中操作设置为在绘制三角形的位置减少缓冲区中的值。仅当阴影体积面的像素可见（即未被任何物体遮挡）时，才会执行递增和递减操作。最后，再次渲染整个场景，这一次仅使用受光源影响的活动材质的组件，并且仅在stencil-buffer中的值为0时显示。值为0表示光线离开阴影的次数与进入阴影体积的次数相同，即该位置被光源照亮。
## 1.2缺陷
为每个三角形创建四边形会产生大量的重复。每个三角形将会创建三个必须渲染的四边形，假如一个球由一千个三角形组成，就会形成三千个四边形，每个四边形都可能会跨越屏幕。一种解决方案是只沿着对象的轮廓边绘制这些四边形，例如，球体可能只存在50条轮廓边，因此只需要50条四边形。几何体着色器可用于自动生成此类轮廓边缘
<div align=center>![图1.3](https://renderwiki.github.io/ImageResources/Shadow Volumes/Figure2.3.png)</div>
<center>图1.3 左：角色投射了一个阴影。右：显示该模型的拉伸三角形。 
</center>
 然而，阴影体积算法存在一个巨大缺陷：极端的变化性。假设存在一个单一的小三角形，如果相机和光源处于完全相同的位置，阴影体积算法的开销最小，形成的四边形不会覆盖任何像素。但如果观察者围绕三角形进行旋转，并始终将其保持在视野内，随着相机远离光源，阴影体积四边形会覆盖更多屏幕，从而导致更多计算。如果观察者进入阴影体积中，阴影体积将会填满屏幕，需要花费大量时间进行计算。这种变化性会导致算法的开销出现巨大的、不可预测的跳跃，使得阴影体积算法无法应用于具有固定帧率的交互式应用程序中。 
 
 由于上述原因，阴影体积算法被大多数应用程序放弃。然而，随着技术的不断发展，以及研究人员的不断探究，该算法有一天可能会被重新广泛使用。
# 参考文献：
[1] Sintorn, Erik, Viktor K¨ ampe, Ola Olsson, and Ulf Assarsson, “Per-Triangle Shadow Volumes Using a View-Sample Cluster Hierarchy,” in Proceedings of the 18th Meeting of the ACM SIGGRAPH Symposium on Interactive 3D Graphics and Games, ACM, pp. 111–118, Mar. 2014. Cited on p. 233, 259
Heidmann, Tim, “Real Shadows, Real Time,” Iris Universe, no. 18, pp. 23–31, Nov. 1991. Cited on p. 230, 231
[2] Crow, Franklin C., “Shadow Algorithms for Computer Graphics,” Computer Graphics (SIGGRAPH ’77 Proceedings), vol. 11, no. 2, pp. 242–248, July 1977. Cited on p. 230
[3] Seiler, L. D. Carmean, E. Sprangle, T. Forsyth, M. Abrash, P. Dubey, S. Junkins, A. Lake, J. Sugerman, R. Cavin, R. Espasa, E. Grochowski, T. Juan, and P. Hanrahan, “Larrabee: A Many-Core x86 Architecture for Visual Computing,” ACM Transactions on Graphics, vol. 27, no. 3, pp. 18:1–18:15, 2008. Cited on p. 230, 996, 1017
[4] Rottger, Stefan, Alexander Irion, and Thomas Ertl, “Shadow Volumes Revisited,” Journal of WSCG (10th International Conference in Central Europe on Computer Graphics, Visualization and Computer Vision), vol. 10, no. 1–3, pp. 373–379, Feb. 2002. Cited on p. 232
[5] Everitt, Cass, and Mark Kilgard, “Practical and Robust Stenciled Shadow Volumes for Hardware-Accelerated Rendering,” NVIDIA White Paper, Mar. 2002. Cited on p. 232
[6] Hornus, Samuel, Jared Hoberock, Sylvain Lefebvre, and John Hart, “ZP+: Correct Z-Pass Stencil Shadows,” in Proceedings of the 2005 Symposium on Interactive 3D Graphics and Games, ACM, pp. 195–202, Apr. 2005. Cited on p. 232
[7] Stich, Martin, Carsten Wachter, and Alexander Keller, “Efficient and Robust Shadow Volumes Using Hierarchical Occlusion Culling and Geometry Shaders,” in Hubert Nguyen, ed., GPU Gems 3, Addison-Wesley, pp. 239–256, 2007. Cited on p. 233
[8] Lloyd, Brandon, Jeremy Wendt, Naga Govindaraju, and Dinesh Manocha, “CC Shadow Volumes,” in Proceedings of the 15th Eurographics Workshop on Rendering Techniques, Eurographics Association, pp. 197–206, June 2004. Cited on p. 233
[9] Sintorn, Erik, Viktor Kampe, Ola Olsson, and Ulf Assarsson, “Per-Triangle Shadow Volumes Using a View-Sample Cluster Hierarchy,” in Proceedings of the 18th Meeting of the ACM SIGGRAPH Symposium on Interactive 3D Graphics and Games, ACM, pp. 111–118, Mar. 2014. Cited on p. 233, 259
