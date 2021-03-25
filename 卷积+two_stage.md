## 卷积神经网络(CNN)

### 是什么？

1. 输入层(Input layer)

   > 图片的向量形式 [batch_size,m,n,3]

2. 卷积层(Conv layer)

   【作用】用来提取特征

   【计算方式】互相关运算，(W-F+2P)/S+1

   ![img](https://pic4.zhimg.com/v2-68dde94137ea012bf5de36219d71fb7f_b.webp)

   【名词解释】核(kernel)、感受野、步幅(stride)、填充(padding)、特征图(feature map)

3. 池化层(pooling layer)

   【作用】降维、防止过拟合（下采样）

   【计算方式】最大池化、平均池化

   ![image-20210316214245554](.\image\polling.png)

4. 激励层(ReLU layer)

   【作用】将卷积层输出结果做非线性映射

   【计算方式】ReLU: f(x)=max(0,x)

   【理解】表达的是一种相应变量（目标变量、类标签、分数）等与其解释变量的一种非线性变化。[参考链接](https://blog.goodaudience.com/artificial-neural-networks-explained-436fcf36e75)

5. 全连接层(FC layer)

   【作用】对结果进行输出，根据不同任务输出想要的结果

   【计算方式】softmax等

6. example(手写体识别)

   <img src=".\image\minist.png" alt="image-20210322201546018" style="zoom:70%;" />

   ![image-20210322203832746](./image\minist_all.png)

### 为什么？

1. 为什么不直接使用全连接的网络，而要先用卷积提取特征？

   > 1. 数据量太大，计算成本高
   > 2. 可以有效避免过拟合

2. 为什么CNN有用？

   > 1. 较少参数量
   > 2. 比较有效提取图片特征 [参考链接](https://www.zhihu.com/question/32260451/answer/284404502) （一些认为有效果的，但是应用于整个网络的想法可能就没那么有效，eg：Cascade r-cnn）

## 目标检测

### one_stage

> 没有中间的区域检出过程，直接从图片获得预测结果

+ SSD、yolo系列

### two_stage

> 先生成proposals建议框，然后进行预测

![image-20210323200144467](./\image\two_stage.png)

#### Faster R-CNN

1. resize到指定大小，经过backbone得到feature maps共享特征层（features map实际为一个抽取的最后输出为batch_size*38\*38\*1024的卷积网络）

2. 通过两个1*1卷积改变通道数到 18以及36

   + 9\*4的卷积用来预测公用特征层上每个采样点每个先验框的**变化情况**
   + 9\*2的卷积用来预测每个采样点上每个预测狂是否包含物体

   利用score以及建议框调整值调整建议框，通过nms从12000多个先验框中筛选出比较准确的建议框（eg：600）

3. 利用建议框rois在共享特征层feature map上进行截取并分区域池化从而得到同等shape的局部特征层

4. 对每个建议框进行Resnet原有的第五次压缩。压缩完后进行一个平均池化，再进行一个Flatten，最后分别进行一个num_classes的全连接和(num_classes)x4全连接

结构图：

<img src="./image\faster r-cnn_modify.png" alt="image-20210323162900464" style="zoom:80%;" />

#### ResNet 50

> 已有的神经网络很难做到恒等映射（特征随着层层前向传播得到完整保留，增加一个什么都不做的网络效果并不会变差）
>
> ResNet直接将恒等映射作为网络的一部分，学习目标为F(x)=H(x)-x

结构图：

<img src=".\image\resnet.jpg"  />

具体的：

<img src="./image\resnet50.png" style="zoom:0%;" />

####　FPN

1. 提取到的P2、P3、P4、P5、P6可以作为RPN网络的有效特征层（实验表面用P2效果提升最明显）
2. 提取到的P2、P3、P4、P5可以作为Classifier和Mask网络的有效特征层

结构图：

![](./\image\FPN.jpg)

理解：

1. 最后又进行一次3\*3卷积是为了消除上采样带来的
2. 将高层特征传下来，补充底层的语义，这样可以获得高分辨率、强语义的特征，有利于小目标的检测

#### Mask R-CNN

1. ROI Align代替ROI pooling 利用**双线性插值法**解决其在按比例池化需要取整时出现的候选框位置偏差问题(misalignment)
2. 增加mask卷积

#### Cascade R-CNN

存在问题：实际IOU情况于其阈值设置之间的mismatch问题

<img src=".\image\mismatch.png" alt="image-20210323205139653" style="zoom:80%;" />

解决方法：

![](./\image\cascade r-cnn.png)

#### DCN

V1: 对于普通卷积来说，卷积操作的位置都是固定的，而可变形卷积引入对x、y的offset，卷积操作的位置会在监督信息指导下进行选择，可以较好的适应目标的各种尺寸、形状，因此提取的特征更加丰富并能更集中到目标本身；

<img src="./\image\DCN_view.png" style="zoom:80%;" />

具体操作:

<img src="./\image\DCN.png" style="zoom:80%;" />

## 数据增广

+ 基本数据增强操作：RandomFilp、Pad、裁剪、填鸭式等

+ mixup

  + 具体实现：

    <img src="./\image\mixup.png" alt="image-20210323211950648" style="zoom: 50%;" />

  + 具体效果

  + 原作者回答 [参考链接](https://www.zhihu.com/question/67472285/answer/256651581)