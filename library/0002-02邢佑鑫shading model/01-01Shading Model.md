## 简介
Shading Model不止一种，主要用于处理光线反射的过程，保证物体表面被光线合理的照亮。
## 1.Shading Model
### 1.1重要变量
简单的着色模型根据点光源的照明进行定义。

在光线反射过程中，**重要的变量**有：

- **光线方向**![](http://latex.codecogs.com/svg.latex?l)，它是指向光源的单位矢量；
- **视线方向**![](http://latex.codecogs.com/svg.latex?v)，它是指向眼睛或者相机的单位矢量；
- **表面法线**![](http://latex.codecogs.com/svg.latex?n)，它是在发生反射的点处，垂直于表面的单位矢量；
- 以及**颜色**![](http://latex.codecogs.com/svg.latex?color)、**光泽**![](http://latex.codecogs.com/svg.latex?shininess)**等**其他特定于模型的属性。



参考文献：

[1] Marschner S ,  Shirley P . Fundamentals of computer graphics. 4th edition.[J]. World Scientific Publishers Singapore, 2009, 9(1):81.