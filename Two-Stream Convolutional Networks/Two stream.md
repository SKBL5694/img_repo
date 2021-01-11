# Two stream

#### directory

sec2 指定Spatial ConvNet

**sec3 指定Temporal ConvNet**

sec4 多任务框架

**sec5 实现细节**



##### **光流文献**

光流计算方法来自于**[2]High accuracy optical flow estimation based on a theory for warping, 2004**

可能更好的光流计算**[30]DeepFlow: Large displacement optical flow with deep matching, 2013**



**两种Iτ计算方式**

**1.Optocal flow stacking**(基于同一位置(u,v)采样的光流)

![](https://raw.githubusercontent.com/SKBL5694/img_repo/master/Two-Stream%20Convolutional%20Networks/img1.PNG)

τ帧对应的光流为，从τ开始往后数L帧，即到**(τ+L)**帧，这样便有了L个间隔，结合xy两个方向，就有了2L个Channel的光流作为输入。

**Iτ(u,v,c)存的是每帧固定位置(u,v)的位移矢量**

**2.Trajectory stacking**(基于运动轨迹采样的光流)

![](https://raw.githubusercontent.com/SKBL5694/img_repo/master/Two-Stream%20Convolutional%20Networks/img2.PNG)

pk是τ后第k帧上的一点，有τ帧上的初始点(u,v)经过递归计算得到(相当于追踪运动轨迹)

p2 = p1+dτ(p1)

**Iτ(u,v,c)存的是起始点(u,v)的沿运动轨迹位移矢量（逐帧首尾相连）**



## 从上面公式还能看出，x光流与y光流在页这个维度上交替排布

**两种Iτ形式区别见下图**

![](https://raw.githubusercontent.com/SKBL5694/img_repo/master/Two-Stream%20Convolutional%20Networks/img3.PNG)

**Tricks**

**BI-direction** 之前描述的是从τ帧开始往后数L帧，现在改为从τ-L/2到τ+L/2.但具体是(τ-L/2, τ)还是(τ,τ-L/2)还不好说

**Mean flow subtraction** 防止因为相机移动导致的光流出现，文章采取了简单办法，每个点的位移场减去平均位移场（推测为所有点的平均值）。更复杂的方法在[10],[26]有提到。、

**Architecture** 为配适常见ConvNet，将w×h×2L的输入采样为224×224×2L，并采取同空间域一样的网络构架。



# ?

3.3中的可视化没看懂，96列可以理解，代表96个卷积核，但20行(对应于L=10的情况，所以在页这个维度上有20个通道？不懂)，这部分相当于图片卷积中的RGB3通道，不知道怎么能被拆开展开成20行。类比于RGB3通道，难道是把**卷积核的每个通道**可视化了？确实是这样，alexnet中的96个可视化应该是把三层叠加在一起了，所以会有颜色。

所以3.3中的可视化结果是把**feature-map**展示了出来(没有拍在一起)，但实际上应该是xy交叉的featuremap为了展示应该是被分堆儿了。

![](https://raw.githubusercontent.com/SKBL5694/img_repo/master/Two-Stream%20Convolutional%20Networks/img4.PNG)

## Question：空间流的预训练可在imagenet，但时间流只能在ucf101和HMDB51，数据量不够，咋办？

## Answer: 多任务学习。FC之后跟两个softmax，一个去HMDB一个去ucf101，各自损失只对各自视频起作用，总损失为两项相加。



## 网络结构

对应于[3]的CNN-M-2048结构，fig1中也有，与[31]的网络相似

LRN(已淘汰)与[15]的设置一样

temperol 的第二个正则化层被删了

### 训练

每轮取256个样本，各自随机采其中一帧

**空间上** 缩放到256×256 随机crop224×224，随机flip和RGB jitter

**时间上** 计算选中帧光流  随机crop和flip 224×224×2L 

**lr设置** 略

### 测试

给定video，等间距取25帧，每帧crop四角和中心并flip，如此每帧得到10个input，得分取frames和crops的平均值。

### 光流处理

open-cv 预先计算（0.06s一对frames，即一张光流），缩放到[0,225]后压缩存入jpeg，用的时候再转回来。

能将1.5T光流数据变为27G





