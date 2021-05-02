# YoloV1
待完成
# YoloV3
![YoloV3结构图](https://pic2.zhimg.com/v2-af7f12ef17655870f1c65b17878525f1_r.jpg)


## 基本元件
## CBL = Conv(卷积层)->BN(batch normalization, 批标准化)->Leaky Relu(带泄露修正线性单元)
#### Batch Normalization
一个神经网络在训练过程中，如果参数发生了变化，就会导致输出的数据分布发生改变(ICS, Internal Covariate Shift)，那么后一层的网络就需要适应学习新的数据分布，而不断改变的数据分布会影响网络的训练速度。经过预处理的数据会比原始数据在训练过程中得到更加理想的结果。标准化满足数据零均值，单位方差和弱相关性，能够达到快速收敛加速训练速度的效果。那么在网络的每一层输入的时候，增加一个归一化层（在这里也就是BN层），就能达到我们所要的效果了。

*参考文献:*

*[深度学习（二十九）Batch Normalization 学习笔记](https://blog.csdn.net/hjimce/article/details/50866313)*

*[什么是批标准化 (Batch Normalization)](https://zhuanlan.zhihu.com/p/24810318)*

#### Leaky Relu
相对Relu而言可以避免梯度消失
## Res Unit 残差块
浅层输出和深层输出结果求和，使得权重从原本需要学习x->f(x)的表达变为学习x->f(x)-x的表达，残差连接可以被看作特殊的跳跃连接。能够帮助上千层的模型收敛。

*参考文献:*

*[残差网络学习心得](https://blog.csdn.net/litt1e/article/details/87644012)*

*[视觉主干网络之ResNet系列 简单而又高效的残差连接 (视频)](https://www.bilibili.com/video/BV1UK4y1U7aG)*