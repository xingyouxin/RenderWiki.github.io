## 简介

把没有受到光照的表面渲染为全黑通常是不可取的。为了避免这种黑影，通常用一种粗糙但很有效的方法，即在着色模型中添加一个常量部分，其对像素颜色的贡献仅仅取决于被光线击中的对象，与物体表几何形状无关。

这种方法被称为ambient shading（环境光着色）——物体表面就像是被来自任何地方的环境光照亮一样。

## 1.Ambient Shading

### 1.1公式表示

为了方便调整参数，ambient shading通常表示为物体表面颜色与环境光颜色的乘积，所以ambient shading可以对某个表面单独调整，或者对所有表面统一调整。Ambient shading与Blinn-Phong模型的其余部分共同组成了简易着色模型的完整版本：

​                                 $$L=k_aI_a+k_dImax(0, \pmb{n·l})+k_sImax(0, \pmb{n·h})^n  \tag{1}$$

其中$k_a$是表面的环境系数或“环境色”，$I_a$是环境光强度。



参考文献：

[1] Marschner S ,  Shirley P . Fundamentals of computer graphics. 4th edition.[J]. World Scientific Publishers Singapore, 2009, 9(1):83-84.