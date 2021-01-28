# Slow-Fast

### lateral connections

###### ![](https://raw.githubusercontent.com/SKBL5694/img_repo/master/slow_fast/img1.PNG)

###### 侧连接通过sum或者concatenate的方式实现，具体来讲，T2C有两个版本，其余的均是concatenate。

###### 其中1×1卷积的实现：stride为α，则时间维度由αT变为T。通道数为2βC，代表fast路的通道数2倍于自己之前(本来是βC)。但要注意的是，该数字仍为slow路(C)的1/4(因为β等于1/8)

### 一些参数的解释

###### τ就是stride，即步长/采样间隔，T为输入slow网络的帧数，即slow网络需要T帧，则rawframe需要提供τT帧。fast网络输入αT帧，在rawframe不变的情况下，更密集的提取帧。β为我们给fast网络通道数的系数(default 1/8)。一般来讲 αβ = 1

### 与mmaction2 configs文件名“不一致”

###### slowfast实验中最好的参数T×τ=8×8，与config文件名字相符合，但文件内容中的resample_rate #τ, speed_ratio=4,  # α, channel_ratio=8,  # β却看着不是很对应。

###### 原因在于configs中，对于数据本来就有一个SampleFrames的操作，所以要结合这个操作来达到文章中的参数，不过configs的文件名是正确的。

### why slowonly？

###### 因为slowonly用的是Res的backbone，所以哪怕单单是slowonly就已经比之前的在Kinetics400上效果好了。这部分在Table5(a)row1 与Table2中对比可见

### 延伸阅读的文献

######  [19]Accurate, large minibatch SGD: training ImageNet in 1 hour 2017        1小时训练imagenet 

###### [56]**Non-local Neural Networks 2018**          slowfast之前的nonlocal， 弄清啥叫a shorter side randomly sampled in [256,320] 

###### [20] AVA: A video dataset of spatiotemporally localized atomic visual actions 2018

### slowfast关于AVA数据集

![](C:\Users\Admin\AppData\Roaming\Typora\typora-user-images\image-20210128145553512.png)

###### 用了faster RCNN框架做小修改，将2D-ROI扩展为3D-ROI 像[20] AVA: A video dataset of spatiotemporally localized atomic visual actions 2018





