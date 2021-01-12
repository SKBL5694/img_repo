# Fusion

## Contribute

**1** 在卷积层而不是sofatmax层融合不影响性能，并且能节省参数

**2** 在最后一层卷积上融合好于在之前的层融合，另外在分类层再加一次融合有助于提高准确性

**3** 融合层附近的卷积层池化有利于提高精度

## 之前two-stream的两个主要缺陷

**1 无法学习时空特征像素级别的关系，因为只在分类score部分融合的（像素没对齐）**

**2 在时间尺度上过于有限： 空间尺度只选了一帧，时间尺度围绕改帧附近的L帧（如L=10）**

**不过第二个问题在某种程度上被规则排列样本在时间上的池化解决了** （不知道啥意思）

## 融合

### **1空间融合**

空间融合分为两个问题：像素融合和channel融合

#### **像素融合**：easy，将两个空间分辨率相同的卷积层堆叠在一起就可以

#### **channel融合**（对应关系）：

假设**f**为融合函数，如图所示：

![](https://raw.githubusercontent.com/SKBL5694/img_repo/master/Fusion/img1.PNG)

HWD为高，宽，通道数，简单起见H，W一直不变，输入的a，b通道数相等。**略去下标t（也就是暂时只考虑单帧）**。

#### **几种融合方式**

##### **1.Sum fusion**(对应位置的数值相加)

![](https://raw.githubusercontent.com/SKBL5694/img_repo/master/Fusion/img2.PNG)

因为吧，这个channel的编号是随意的（确实很随意，也不知道特定编号的channel代表啥，或者说两个不同的卷积网络输出的通道也没啥对应关系），所以呢对应channel 的值相加也就建立了一种随意的关系，不过我们可以通过**后续的优化**让这种**随意建立的关系**有用。

ps：这就很**强行**

##### **2.Max fusion**(对应位置取最大值)

![](https://raw.githubusercontent.com/SKBL5694/img_repo/master/Fusion/img3.PNG)

ps: 和上面那个差不多离谱吧

##### **3.Concatenation fusion**(在通道对应位置处堆叠两个feature map，即通道数变为原来的二倍)

![](https://raw.githubusercontent.com/SKBL5694/img_repo/master/Fusion/img4.PNG)

这个也挺逗的，相当于把两个不同网络的输出就那么排在一块了（交叉排的），也没啥联系，感觉和在classification阶段的融合（挨着排，I guess）差不多一回事儿。

##### **4.Conv fusion**(延续上面3的思路继续做，感觉是一个fusion中的两步)

![](https://raw.githubusercontent.com/SKBL5694/img_repo/master/Fusion/img5.PNG)

在**3**交叉排列完的基础上，采用1×1卷积（D个1×1×2D卷积），加上偏置项b。这样一来使得维度不变(本来拼成了2D,现在又搞回了D)，并且还融合了特征(每个像素点的结构都是2D个1×1卷积核共同作用后的结果)，原文是：

**reduce the dimensionality by a factor of two and is able to model weighted combinations of the two feature maps x a , x b at the same spatial (pixel) location**

这样，训练时候，f就能以最小化损失函数为前提，自主地学习2个feature map的关系(偷鸡，不过确实合理)

#### **这里原文举了个什么置换相等矩阵的例子，没看懂**

**5.Bilinear fusion**(外积，两个向量乘完变矩阵)

![](https://raw.githubusercontent.com/SKBL5694/img_repo/master/Fusion/img6.PNG)

xa，xb（i,j固定时）均为 1×D向量，转置相乘之后变为D×D的矩阵。

<img src="https://raw.githubusercontent.com/SKBL5694/img_repo/master/Fusion/img7.jpg" style="zoom:20%;" />

feature太大了(D^2)，为了应用这个融合方式，通常在Relu5使用，并移除全连接层，blalba

**这个方法，用一个解析文章里的话说就是：不分主次**



### **2效果对比**

![](https://raw.githubusercontent.com/SKBL5694/img_repo/master/Fusion/img8.PNG)

看下面5个，实在卷积层融合的。参数变少了，Conv的精度也够，**所以阶段性结论是，哪融合都可以**

卷积层融合的唯一限制是两个feature map的尺寸要一致，文章说可以通过“upconvolutional”实现

该方法在经典可视化文章**Visualizing and understanding convolutional networks**中有涉及

或者如果尺寸近似的话，可在较小的feature上充0

#### 另外一种融合是conv5后融合一次且不截断spacial网络，最后在fc出再融合一次（不减少参数，如果仅在conv5融合，则减少一半参数），如图右，该方式的效果后面再评估

![](https://raw.githubusercontent.com/SKBL5694/img_repo/master/Fusion/img9.PNG)

### 3时间融合

之前的融合都是省略下标t的，也就是说一直都是**单帧**的融合，接下来讨论如何**跨越时间t（多帧）**组合feature map.

#### 三种方案

①的输入是R(H,W)二维，正常的池化就是要在每个通道上都做重复操作的，所以要重复D次（一般来说D都略去不谈），这次又堆叠了时间T个帧，又重复T次操作。

②的输入为R(H,W,T)三维，在三维上池化，结果还是三维，重复D次（通常略去不谈）。

③的输入为R(H,W,T,D)四维，按通道都排好了，用D‘个（H,W,T,D）的卷积核卷积（不是很确定，文章说的有点乱）

**① **一种方式是如传统双流的做法，在时间上对网络预测取平均。也就是2D池化（单帧池化）结果是多个2D的帧，多帧预测取平均

**②**3D池化，在堆叠的时间帧上池化(不跨越通道)，结果还是3D的堆叠帧

**③**在concat之后，3D卷积再池化(跨越通道)，卷积的好处在于可训练http://dgschwend.github.io/netscope/#/gist/9b544da58def3b9611c4d333418f4e80看conv3D5的位置

##### *图不放了，放了也对理解没帮助，气死我了*



# 有个大佬写了blog

https://blog.csdn.net/u013588351/article/details/102074562



## 总结： 在融合之前，T这个维度都是独立出来的，有啥操作就在这个维度上做一下就可以了。到了融合开始：空间融合concat通道，也就是说将RGB流的一堆（通道数）feature map和flow的一堆（通道数）feature map交叉叠在一起，对于每个时间采样点Ti都这么做，之后把Ti堆在一起，用D’个文章里叫做3DConv,其实我觉得是4DConv的卷积核（H,W,T,D）来做卷积，只不过T这个维度上是充满的(类似于2DConv其实是把通道充满的3DConv一样)，所以叫3DConv是合理的，因为输出结果是D‘个3Dfeature。

## 我好像忽然理解了卷积取名的意思了，不是指卷积核的大小，而是指卷积核在几个维度上能“滑动”，所以哪怕是100维卷积核，只要它只能在3个维度上滑动，就叫3DConv。文章里就是D’个4维卷积核在H,W,T上滑动！！！！







