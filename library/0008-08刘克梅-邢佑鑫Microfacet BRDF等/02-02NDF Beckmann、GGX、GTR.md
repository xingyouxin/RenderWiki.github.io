## 微表面法线分布函数
微表面BRFDF最重要的部分就是微观面元的法线分布函数NDF，它决定了光泽部分的宽度，形状及其它特征。如图1所示 当微观面元法线方向比较集中时，宏观表面呈现出光泽，而当微观面元法线方向特别集中时，宏观表面呈现出高光特征；当微观面元法线方向杂乱时，宏观表面呈现漫射特征。  
![image](https://github.com/RenderWiki/RenderWiki.github.io/blob/main/ImageResources/MicrofacetBRDFParts/%E5%AE%8F%E8%A7%82%E8%A1%A8%E9%9D%A2%E4%B8%8E%E5%BE%AE%E8%A7%82%E9%9D%A2%E5%85%83%E6%B3%95%E7%BA%BF%E6%96%B9%E5%90%91%E5%88%86%E5%B8%83%E7%9A%84%E5%85%B3%E7%B3%BB.png)    
图1 宏观表面与微观面元法线方向分布的关系。（图来自GAMES 202 实时高质量着色 1）    
## 1.定义
微表面法线分布函数描述了表面的微观几何结构中所有微观面元的法线面向不同方向的概率，是微观表面上的表面法线的统计分布。

## 2.性质
1. 微平面法线密度始终为非负值:
2. 微表面的总面积始终不小于宏观表面总面积:
3. 任何方向上微观表面投影面积始终与宏观表面投影面积相同：
4. 若观察方向为法线方向，则其积分可以归一化。

## 3.举例
⽐较流⾏的NDF包括Beckmann, GGX, GTR等分布函数。
### 3.1Beckmann 
#### 3.1.1简介
Beckmann分布是光学业界开发的第一批微平面模型中使用的法线分布[Beckmann 1963]，也是Cook-Torrance BRDF在提出时选择的NDF [Cook 1982]。
#### 3.1.2公式
$$D ( h ) = \frac { e ^ { - \frac { \tan ^ { 2 } \theta } { \alpha ^ { 2 } } } } { \pi a ^ { 2 } \cos ^ { 4 } \theta }$$  
α：法线的粗糙程度，粗糙程度值越小，表面就越光滑  
θ：微表面半向量法线与宏观表面法线的夹角  
#### 3.1.3特点
类似于高斯函数，离法线越远，分布的越少。  
具备形状不变性（shape-invariant）。  
仅定义在坡度空间。  
### 3.2GGX
#### 3.2.1简介
GGX即Trowbridge-Reitz分布，最初由Trowbridge和Reitz [Trowbridge 1975]推导出，在Blinn 1977年的论文 [Blinn 1977]中也有推荐此分布函数，但一直没有受到图形学界的太多关注。30多年后，Trowbridge-Reitz分布被Walter等人独立重新发现[Walter 2007]，并将其命名为GGX分布。在重新发现并提出GGX分布之后，GGX分布采用风潮开始在电影和游戏行业中广泛传播，成为了如今游戏行业和电影行业中最常用的法线分布函数。
#### 3.2.2公式
D ( h ) = \frac { \alpha ^ { 2 } } { \pi ( ( n \cdot h ) ^ { 2 } ( \alpha ^ { 2 } - 1 ) + 1 ) ^ { 2 } } 
α：法线的粗糙程度
θ：微表面半向量法线与宏观表面法线的夹角
#### 3.2.3特点
GGX具有长尾性质，随着θ的增加，D ( h )会快速衰减，但是衰减到一定程度的时候衰减速度会变慢,即使到了90°时值仍不为0。从图2可以观察到，Beckmann的高光会逐渐消失,而GGX的高光会减少但不会消失。长尾性质使高光到非高光有一个柔和的过渡状态,而非Beckmann的高光到达90°后戛然而止  

![image](https://github.com/RenderWiki/RenderWiki.github.io/blob/main/ImageResources/MicrofacetBRDFParts/GGX%E4%B8%8EBeckmann%E5%AF%B9%E6%AF%94%E5%9B%BE.png)
图2 GGX与Beckmann对比图（图来自https://planetside.co.uk/news/terragen-4-5-release/）     

### 3.3GTR
#### 3.3.1简介
Burley [Burley 2012]根据对Berry，GGX等分布的观察，为了允许更多地控制法线分布函数的形状，提出了广义的Trowbridge-Reitz（Generalized-Trowbridge-Reitz，GTR）法线分布函数。
#### 3.3.2公式
D ( h ) = \frac { c } { ( 1 + ( n \cdot h ) ^ { 2 } ( \alpha ^ { 2 } - 1 ) ) ^ { \gamma } }  
α：法线的粗糙程度  
θ：微表面半向量法线与宏观表面法线的夹角  
γ参数用于控制尾部形状,随着γ值的增加，分布的尾部变得更短。γ=1时，GTR即Berry分布；γ=2时，GTR即GGX；当γ超过10会接近Beckmann分布。    
![image]（https://github.com/RenderWiki/RenderWiki.github.io/blob/main/ImageResources/MicrofacetBRDFParts/%E4%B8%8D%E5%90%8C%CE%B3%E5%80%BC%E5%AF%B9%E5%BA%94%E7%9A%84GTR%E5%88%86%E5%B8%83%E6%9B%B2%E7%BA%BF%E4%B8%8E%CE%B8%E7%9A%84%E5%85%B3%E7%B3%BB.png）  
图3 不同γ值对应的GTR分布曲线与θ的关系（图来自GAMES 202 实时高质量着色 1）  

#### 3.3.3特点
GTR分布不具备形状不变性  
具有更长的拖尾  

参考文献：   
Beckmann, Petr, and André Spizzichino, "The Scattering of Electromagnetic Waves from Rough Surfaces", Pergamon Press, 1963.  
Cook, Robert L., and Kenneth E. Torrance, "A Reflectance Model for Computer Graphics," Computer Graphics (SIGGRAPH '81 Proceedings),vol. 15, no. 3, pp. 307-316, Aug. 1981.  
Blinn, James F., "Models of Light Reflection for Computer Synthesized Pictures," ACM Computer Graphics (SIGGRAPH '77 Proceedings), vol. 11, no. 2, pp. 192-198, July 1977.  
Walter, Bruce, Stephen R. Marschner, Hongsong Li, and Kenneth E. Torrance, "Microfacet Models for Refraction through Rough Surfaces," Rendering Techniques 2007, Eurographics Association, pp. 195-206, June 2007.  
Burley, Brent, "Physically Based Shading at Disney," SIGGRAPH Practical Physically Based Shading in Film and Game Production course, Aug. 2012.  
GAMES 202 实时高质量着色 1（表面模型：Microfacet Model, GGX, Multiple Bounce Approximation）
