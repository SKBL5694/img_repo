# C3D

1. 3×3×3的3D卷积核效果最好(stride 1×1×1)，池化层为2×2×2(stride 2×2×2)，poo1除外，第一层为1×2×2(stride 1×2×2)。理由是过早时间融合不好。

### 2. 网络结构如图

![](https://raw.githubusercontent.com/SKBL5694/img_repo/master/C3D%C3%97I3D/img1.PNG)

#### C3D中经常会提到feature这个词，含义是经过一次fc的，也就是fc6的输出结果作为feature，而不是我之前以为的pool5的结果

### 3. 由该文章扩展的参考文献

**1** [46]卷积神经网络经典可视化文章

**2** [43]t-SNE 将特征映射到2个dimension的方法，效果图如图。

![](https://raw.githubusercontent.com/SKBL5694/img_repo/master/C3D%C3%97I3D/img2.PNG)

**3** 此外，有个ASLAN数据集，目标是给定两个视频，判断是否为同一类动作，难点在于test上有never-seen-before数据

## 总的来讲，这个文章给我的感觉就是可视化特别多，用各种可视化的方法描述了自己的方法，另外看这篇文章的原因是slow-fast文章中提到了 non-degenerate temporal conv，以及看看3D卷积网络最开始时候的结构。



# I3D

### 感觉意思不是很大吧。总结一下，I3D是two-stream结构的，一路光流，一路RGB帧堆叠。将Imagenet上的预训练参数沿着时间轴复制N次，之后系数都除以N，保证响应值不变。网络结构如下图：

![](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20210128114307552.png)

**其中REC.Field指感受野，可以画个图计算一下，公式我没看，算到11，27，27没啥问题，右边的Inception没咋看**

**对于3D来说，时间维度不能缩减地过快或过慢。如果时间维度的感受野尺寸比空间维度的大，将会合并不同物体的边缘信息。反之，将捕捉不到动态场景。因此改进了BN-Inception的网络结构。在前两个池化层上将时间维度的步长设为了1，空间还是2*2。最后的池化层是2*7*7。训练的时候将每一条视频采样64帧作为一个样本，测试时将全部的视频帧放进去最后average_score。除最后一个卷积层之外，在每一个卷积后面都加上BN和relu**


