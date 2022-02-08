## 微表面法线分布函数
微表面BRFDF最重要的部分就是微观⾯元的法线分布函数NDF，它决定了光泽部分的宽度，形状及其它特征。
## 1.定义
表面的微观几何结构中，所有微观面元的法线面向不同⽅向的概率
## 2.举例
⽐较流⾏的NDF包括Beckmann, GGX, GTR等分布函数。
### 2.1Beckmann 
#### 2.1.1简介
Beckmann分布是光学业界开发的第一批微平面模型中使用的法线分布[Beckmann 1963]，也是Cook-Torrance BRDF在提出时选择的NDF [Cook 1982]。
#### 2.1.2公式
$$D ( h ) = \frac { e ^ { - \frac { \tan ^ { 2 } \theta } { \alpha ^ { 2 } } } } { \pi a ^ { 2 } \cos ^ { 4 } \theta }$$  
*α*：法线的粗糙程度，粗糙程度值越小，表面就越光滑  
*θh*：微表面半向量与表面法线方向的夹角  
#### 2.1.3特点
类似于高斯函数，离法线越远，分布的越少。  
具备形状不变性（shape-invariant）。  
无限大的函数，都不会出现面朝下的面，但是不能保证法线的方向。  
### 2.2GGX
#### 2.2.1简介
GGX即Trowbridge-Reitz分布，最初由Trowbridge和Reitz [Trowbridge 1975]推导出，在Blinn 1977年的论文 [Blinn 1977]中也有推荐此分布函数，但一直没有受到图形学界的太多关注。30多年后，Trowbridge-Reitz分布被Walter等人独立重新发现[Walter 2007]，并将其命名为GGX分布。在重新发现并提出GGX分布之后，GGX分布采用风潮开始在电影和游戏行业中广泛传播，成为了如今游戏行业和电影行业中最常用的法线分布函数。
#### 2.2.2公式
D ( h ) = \frac { \alpha ^ { 2 } } { \pi ( ( n \cdot h ) ^ { 2 } ( \alpha ^ { 2 } - 1 ) + 1 ) ^ { 2 } }  
#### 2.2.3特点
GGX拥有长的尾部。
到了90°的时候还有值。

### 2.3GTR
#### 2.3.1简介
Burley [Burley 2012]根据对Berry，GGX等分布的观察，为了允许更多地控制法线分布函数的形状，提出了广义的Trowbridge-Reitz（Generalized-Trowbridge-Reitz，GTR）法线分布函数。
#### 2.3.2公式
D ( h ) = \frac { c } { ( 1 + ( n \cdot h ) ^ { 2 } ( \alpha ^ { 2 } - 1 ) ) ^ { \gamma } }  
其中，γ参数用于控制尾部形状,随着γ值的增加，分布的尾部变得更短。γ=1时，GTR即Berry分布；γ=2时，GTR即GGX；当γ超过10会接近Beckmann分布。  

图 各种γ值对应的GTR分布曲线与θh的关系

#### 2.3.3特点
GTR分布不具备形状不变性  

参考文献： 
Beckmann, Petr, and André Spizzichino, "The Scattering of Electromagnetic Waves from Rough Surfaces", Pergamon Press, 1963.  
Cook, Robert L., and Kenneth E. Torrance, "A Reflectance Model for Computer Graphics," Computer Graphics (SIGGRAPH '81 Proceedings),vol. 15, no. 3, pp. 307-316, Aug. 1981.  
Blinn, James F., "Models of Light Reflection for Computer Synthesized Pictures," ACM Computer Graphics (SIGGRAPH '77 Proceedings), vol. 11, no. 2, pp. 192-198, July 1977.  
Walter, Bruce, Stephen R. Marschner, Hongsong Li, and Kenneth E. Torrance, "Microfacet Models for Refraction through Rough Surfaces," Rendering Techniques 2007, Eurographics Association, pp. 195-206, June 2007.  
Burley, Brent, "Physically Based Shading at Disney," SIGGRAPH Practical Physically Based Shading in Film and Game Production course, Aug. 2012.  
