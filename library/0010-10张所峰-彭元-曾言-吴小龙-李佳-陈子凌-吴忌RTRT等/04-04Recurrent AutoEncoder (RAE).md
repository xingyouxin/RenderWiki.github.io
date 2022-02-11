## RAE（Recurrent AutoEncoder）

随着神经网络的兴起，基于学习的方法也逐渐应用于实时渲染领域，其中一个交互级别的稀疏蒙特卡洛图像序列（1spp）重建方法就是RAE[3]。同样使用TAA的思想，在网络迭代训练的过程中，RAE将过去若干帧的信息也保留用于当前帧的去噪和重建。该方法在 NVIDIA (Pascal) Titan X上运行，对1280×720分辨率的图像上的重建时间是每帧 54.9毫秒。

<div align=center>![RAE结果](https://renderwiki.github.io/ImageResources/RAE/RAE结果.png)</div>

<center>图1 1spp噪声输入（左）、RAE输出（中）和4096spp收敛图（右）对比</center>

## 1.网络结构

<div align=center>![RAE网络结构](https://renderwiki.github.io/ImageResources/RAE/RAE网络结构.png)</div>

<center>图2 RAE整体网络结构（左）以及循环卷积块细节（右）</center>


RAE网络结构如图2所示。整体网络前半部分为编码器（Encoder），后半部分为解码器（Decoder）。网络的输入为逐像素级的大小为7的标量（噪声输入RGB、法向量、深度以及粗糙度）。每个编码器层级都有一个卷积层和2×2大小的最大池化层。每个解码器层级都有一个2×2的最邻近向上采样，并与从跳跃连接（skip）来的空间分辨率大小一致的特征合并后，再应用两套卷积层和池化层。所有的卷积层都是3×3大小的。

RAE是基于循环神经网络（RNN）的结构，具有反馈回路（图19右展示了编码器中的循环卷积结构），将先前帧的隐含层的输出与当前帧的输入连接起来，从而保留了连续帧之间的重要时间信息。

## 2.损失函数

RAE使用的损失函数包含一个空间的L1损失、一个梯度域的L1损失和一个时间的L1损失：

<div align=center>![损失函数](https://renderwiki.github.io/ImageResources/RAE/损失函数.png)</div>

<math>\begin{array}{c}
\mathcal{L}=w_{\mathrm{s}} \mathcal{L}_{\mathrm{s}}+w_{\mathrm{g}} \mathcal{L}_{\mathrm{g}}+w_{\mathrm{t}} \mathcal{L}_{\mathrm{t}} \\
\mathcal{L}_{\mathrm{s}}=\frac{1}{N} \sum_{i}^{N}\left|P_{i}-T_{i}\right| \\
\mathcal{L}_{\mathrm{g}}=\frac{1}{N} \sum_{i}^{N}\left|\nabla P_{i}-\nabla T_{i}\right| \\
\mathcal{L}_{\mathrm{t}}=\frac{1}{N} \sum_{i}^{N}\left(\left|\frac{\partial P_{i}}{\partial t}-\frac{\partial T_{i}}{\partial t}\right|\right)
\end{array}</math>

其中，![](http://latex.codecogs.com/svg.latex?P_{i})和![](http://latex.codecogs.com/svg.latex?T_{i})分别代表第![](http://latex.codecogs.com/svg.latex?i)帧的预测值和目标值，![](http://latex.codecogs.com/svg.latex?\nabla)为使用HFEN计算的梯度,![](http://latex.codecogs.com/svg.latex?w)为每个损失对应的权重，RAE中分别设置为![](http://latex.codecogs.com/svg.latex?w_{\mathrm{s} / \mathrm{g} / \mathrm{t}}=0.8 / 0.1 / 0.1)。
空间损失用来单帧去噪，梯度域损失用来维持边缘细节，而时间损失用来维持时间稳定性。

参考文献：

[1] C. R. A. Chaitanya, A. S. Kaplanyan, C. Schied, M. Salvi, A. Lefohn, D. Nowrouzezahrai, and T. Aila, “Interactive reconstruction of monte carlo image sequences using a recurrent denoising autoencoder,” ACM Trans. Graph., vol. 36, no. 4, pp. 1–12, 2017.
