# Yolo V1
![YoloV1结构图](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9tbWJpei5xcGljLmNuL21tYml6X3BuZy9pYzRmYWpIQ1dpYkl6OXJmMXFuTjJNSkZnYTdYeWljdWliSnEwWWJCdmZ0clRvdmM4NTh1dXdNNmx3Q0V4V3UySHVTM3dnRWRpY0hKM0JZQUxxcFdRbDA1NU1BLzY0MA?x-oss-process=image/format,png)

## 模型概述

YOLO V1采用一个单独的CNN模型实现end-to-end的目标检测，采用卷积网络来提取特征，然后使用全连接层来得到预测值。网络类似GoogleNet，包含24个卷积层和2个全连接层，如左中图所示。

对于卷积层，主要使用1x1卷积来做通道降维(可以看作跨通道的池化，整合多通道信息，通过控制卷积核个数实现)，然后紧跟3x3卷积。对于卷积层和全连接层，采用Leaky ReLU激活函数。但是最后一层却采用线性激活函数。

## 通道降维与升维
![降维](https://img-blog.csdnimg.cn/20201216161039363.gif#pic_center)
可以理解为7x7x3的层通过和3x3x3的卷积核进行运算后会得到7x7x1的输出层，那么只要有多个卷积核就会有维度为7x7xn的输出层。

1x1卷积具有减少参数/增加模型深度的作用：1x1的卷积把256维通道降到64维，使用3x3卷积，然后在最后通过1x1卷积恢复，参数数目：1x1x256x64 + 3x3x64x64 + 1x1x64x256 = 69632，全连接层就是两个3x3x256的卷积，参数数目: 3x3x256x256x2 = 1179648，差了16.94倍。<sup>[1]<sup>

1x1卷积的主要作用有以下几点：<sup>[2]<sup>

1. 降维。比如，一张500x500且厚度depth为100 的图片在20个filter上做1x1卷积，那么结果的大小为500x500x20。
2. 加入非线性。卷积层之后经过激励层，1x1卷积在前一层的学习表示上添加了非线性激励，提升网络的表达能力；
3. 增加模型深度。可以减少网络模型参数，增加网络层深度，一定程度上提升模型的表征能力。

*参考文献：*

*[1] [为什么要分别使用1*1，3*3，1*1的卷积核进行降维和升维](https://blog.csdn.net/qq_37365385/article/details/111224471)*

*[2] [详细学习1*1卷积核](https://blog.csdn.net/qq_27871973/article/details/82970640)*

## 全连接层
全连接层在CNN中起到将隐含层中学习到的特征表示映射到样本标记空间。卷积在空间上部分连接，在通道上全连接，可以看作特征融合后从特征映射到标签的过程。

全连接层还可以当作是不进行特征共享的卷积层。

[全连接层的作用是什么？ - 胡孟的回答 - 知乎](https://www.zhihu.com/question/41037974/answer/150585634)

*参考文献：*

*[全连接层的作用是什么？](https://www.zhihu.com/question/41037974)*

# Yolo V3
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