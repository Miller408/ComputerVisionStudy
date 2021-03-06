2014年，GoogLeNet和VGG是当年ImageNet挑战赛(ILSVRC14)的双雄，GoogLeNet获得了第一名、VGG获得了第二名，这两类模型结构的共同特点是层次更深了。VGG继承了LeNet以及AlexNet的一些框架结构，而GoogLeNet则做了更加大胆的网络结构尝试，虽然深度只有22层，但大小却比AlexNet和VGG小很多，GoogleNet参数为500万个，AlexNet参数个数是GoogleNet的12倍，VGGNet参数又是AlexNet的3倍，因此在内存或计算资源有限时，GoogleNet是比较好的选择；从模型结果来看，GoogLeNet的性能却更加优越。

> 小知识：GoogLeNet是谷歌（Google）研究出来的深度网络结构，为什么不叫“GoogleNet”，而叫“GoogLeNet”，据说是为了向“LeNet”致敬，因此取名为“GoogLeNet”

那么，GoogLeNet是如何进一步提升性能的呢？
一般来说，提升网络性能最直接的办法就是增加网络深度和宽度，深度指网络层次数量、宽度指神经元数量。但这种方式存在以下问题：
（1）参数太多，如果训练数据集有限，很容易产生过拟合；
（2）网络越大、参数越多，计算复杂度越大，难以应用；
（3）网络越深，容易出现梯度弥散问题（梯度越往后穿越容易消失），难以优化模型。
所以，有人调侃“深度学习”其实是“深度调参”。
解决这些问题的方法当然就是在增加网络深度和宽度的同时减少参数，为了减少参数，自然就想到将全连接变成稀疏连接。但是在实现上，全连接变成稀疏连接后实际计算量并不会有质的提升，因为大部分硬件是针对密集矩阵计算优化的，稀疏矩阵虽然数据量少，但是计算所消耗的时间却很难减少。

那么，有没有一种方法既能保持网络结构的稀疏性，又能利用密集矩阵的高计算性能。大量的文献表明可以将稀疏矩阵聚类为较为密集的子矩阵来提高计算性能，就如人类的大脑是可以看做是神经元的重复堆积，因此，GoogLeNet团队提出了Inception网络结构，就是构造一种“基础神经元”结构，来搭建一个稀疏性、高计算性能的网络结构。

**【问题来了】什么是Inception呢？**
Inception历经了V1、V2、V3、V4等多个版本的发展，不断趋于完善，下面一一进行介绍

## 一、**Inception V1**

论文地址：[Going Deeper with Convolutions](http://arxiv.org/abs/1409.4842)

主要贡献：提出了Inception的卷积网络结构。

### 1.1 原理

论文中提到，想要更高的识别率，很自然的想到增加网络的层数（网络更深）和每层卷积数量（网络更宽），但是这样会导致两个问题：第一个就是参数太大了，容易导致over-fitting（过拟合），特别是在数据集数量有限的情况下，而有效的数据集获取非常困难，需要大量人的人力进行标定；另外一个缺点就是会导致很多重复计算。想要解决这个问题就是使用更加稀疏的网络结构，但是现在的计算设备对于分布不均匀并且稀疏的计算比较困难。
Inception结构就是为了近似稀疏分布提出来的，主要思想就是考虑如何近似一个局部最优的稀疏卷积视觉网络结构。

通过设计一个稀疏网络结构，但是能够产生稠密的数据，既能增加神经网络表现，又能保证计算资源的使用效率。谷歌提出了最原始Inception的基本结构。

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419175920.png)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419183714.png)

该结构将CNN中常用的卷积（1x1，3x3，5x5）、池化操作（3x3）堆叠在一起（卷积、池化后的尺寸相同，将通道相加），一方面增加了网络的宽度，另一方面也增加了网络对尺度的适应性。

首先看第一个结构，有四个通道，有1×1、3×3、5×5卷积核，该结构有几个特点：

① 采用不同大小的卷积核意味着不同大小的感受野，最后拼接意味着不同尺度特征的融合；
② 之所以卷积核大小采用1×1、3×3、5×5主要是为了方便对齐。设定卷积步长stride=1之后，采用大小不同的卷积核，只要分别设定padding =0、1、2，采用same卷积可以得到相同维度的特征，然后这些特征直接拼接在一起；
③ 文章说很多地方都表明pooling挺有效，所以Inception里面也嵌入了pooling。
④ 网络越到后面特征越抽象，且每个特征涉及的感受野也更大，随着层数的增加，3x3和5x5卷积的比例也要增加。

网络卷积层中的网络能够提取输入的每一个细节信息，同时5x5的滤波器也能够覆盖大部分接受层的的输入。还可以进行一个池化操作，以减少空间大小，降低过度拟合。在这些层之上，在每一个卷积层后都要做一个ReLU操作，以增加网络的非线性特征。
然而这个Inception原始版本，所有的卷积核都在上一层的所有输出上来做，而那个5x5的卷积核所需的计算量就太大了，造成了特征图的厚度很大，为了避免这种情况，在3x3前、5x5前、max pooling后分别加上了1x1的卷积核，以起到了降低特征图厚度的作用，这也就形成了Inception v1的网络结构，如下图所示：

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419180059.png)

**1x1的卷积核有什么用呢？**
1x1卷积的主要目的是为了减少维度，还用于修正线性激活（ReLU）。比如，上一层的输出为100x100x128，经过具有256个通道的5x5卷积层之后(stride=1，pad=2)，输出数据为100x100x256，其中，卷积层的参数为128x5x5x256= 819200。而假如上一层输出先经过具有32个通道的1x1卷积层，再经过具有256个输出的5x5卷积层，那么输出数据仍为为100x100x256，但卷积参数量已经减少为128x1x1x32 + 32x5x5x256= 204800，大约减少了4倍。

### 1.2 网络结构

基于Inception构建了GoogLeNet的网络结构如下（共22层）：

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419180244.png)

对上图说明如下：
（1）GoogLeNet采用了Inception模块化（9个）的结构，共22层，方便增添和修改;
（2）网络最后采用了average pooling（平均池化）来代替全连接层，该想法来自NIN（Network in Network），事实证明这样可以将准确率提高0.6%。但是，实际在最后还是加了一个全连接层，主要是为了方便对输出进行灵活调整；
（3）虽然移除了全连接，但是网络中依然使用了Dropout ; 
（4）为了避免梯度消失，网络额外增加了2个辅助的softmax用于向前传导梯度（辅助分类器）。辅助分类器是将中间某一层的输出用作分类，并按一个较小的权重（0.3）加到最终分类结果中，这样相当于做了模型融合，同时给网络增加了反向传播的梯度信号，也提供了额外的正则化，对于整个网络的训练很有裨益。而在实际测试的时候，这两个额外的softmax会被去掉。

GoogLeNet的网络结构图细节如下：

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419180455.png)

注：上表中的“#3x3 reduce”，“#5x5 reduce”表示在3x3，5x5卷积操作之前使用了1x1卷积的数量。

GoogLeNet网络结构明细表解析如下：

**0、输入**
原始输入图像为224x224x3，且都进行了零均值化的预处理操作（图像每个像素减去均值）。

**1、第一层（卷积层）**
使用7x7的卷积核（滑动步长2，padding为3），64通道，输出为112x112x64，卷积后进行ReLU操作
经过3x3的max pooling（步长为2），输出为((112 - 3+1)/2)+1=56，即56x56x64，再进行ReLU操作。

**2、第二层（卷积层）**
使用3x3的卷积核（滑动步长为1，padding为1），192通道，输出为56x56x192，卷积后进行ReLU操作
经过3x3的max pooling（步长为2），输出为((56 - 3+1)/2)+1=28，即28x28x192，再进行ReLU操作。

**3a、第三层（Inception 3a层）**
分为四个分支，采用不同尺度的卷积核来进行处理
（1）64个1x1的卷积核，然后RuLU，输出28x28x64
（2）96个1x1的卷积核，作为3x3卷积核之前的降维，变成28x28x96，然后进行ReLU计算，再进行128个3x3的卷积（padding为1），输出28x28x128
（3）16个1x1的卷积核，作为5x5卷积核之前的降维，变成28x28x16，进行ReLU计算后，再进行32个5x5的卷积（padding为2），输出28x28x32
（4）pool层，使用3x3的核（padding为1），输出28x28x192，然后进行32个1x1的卷积，输出28x28x32。
将四个结果进行连接，对这四部分输出结果的第三维并联，即64+128+32+32=256，最终输出28x28x256

**3b、第三层（Inception 3b层）**
（1）128个1x1的卷积核，然后RuLU，输出28x28x128
（2）128个1x1的卷积核，作为3x3卷积核之前的降维，变成28x28x128，进行ReLU，再进行192个3x3的卷积（padding为1），输出28x28x192
（3）32个1x1的卷积核，作为5x5卷积核之前的降维，变成28x28x32，进行ReLU计算后，再进行96个5x5的卷积（padding为2），输出28x28x96
（4）pool层，使用3x3的核（padding为1），输出28x28x256，然后进行64个1x1的卷积，输出28x28x64。
将四个结果进行连接，对这四部分输出结果的第三维并联，即128+192+96+64=480，最终输出输出为28x28x480

第四层（4a,4b,4c,4d,4e）、第五层（5a,5b）……，与3a、3b类似，在此就不再重复。

从GoogLeNet的实验结果来看，效果很明显，差错率比MSRA、VGG等模型都要低，对比结果如下表所示：

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419181221.png)

### 1.3 Inception作用

作者指出了Inception的优点：

- 显著增加了每一步的单元数目，计算复杂度不会不受限制，尺度较大的块卷积之前先降维。

- 视觉信息在不同尺度上进行处理聚合，这样下一步可以从不同尺度提取特征。

但是具体，为什么Inception会起作用，一直想不明白，作者后面实验也证明了GoogLeNet的有效性，但为什么也没有具体介绍。深度学习也是一个实践先行的学科，实践领先于理论，实践证明了它的有效性。后来看到一个博客，解开了我的谜团。在此贴出他的回答就是：

> Inception的作用：代替人工确定卷积层中的过滤器类型或者确定是否需要创建卷积层和池化层，即：不需要人为的决定使用哪个过滤器，是否需要池化层等，由网络自行决定这些参数，可以给网络添加所有可能值，将输出连接起来，网络自己学习它需要什么样的参数。

## 二、Inception V2

论文地址：[Rethinking the Inception Architecture for Computer Vision](http://arxiv.org/abs/1512.00567)

**Inception V2相比Inception V1进行了如下改进:**

- 加入**Batch Normalization**(批归一化)层，标准结构为:卷积-BN-relu，减少了Internal Covariate Shift（内部neuron的数据分布发生变化），使每一层的输出都规范化到一个N(0, 1)的高斯；

- 分解为更小的卷积，借鉴VGG的使用，使用两个3*3的卷积串联来代替Inception 模块中的5*5卷积模块。因为两个3*3的卷积与一个5*5的卷积具体相同的感受野，但是参数量却少于5*5的卷积。并且因为增加了一层卷积操作，则对应多了一次Relu，即增加一层非线性映射，使特征信息更加具有判别性；
- 使用非对称卷积对，3×3的卷积进一步分解为3×1和1×3的卷积；

### 2.1 Batch Normalization

这个算法太牛了，使得训练深度神经网络成为了可能。

#### 2.1.1 为了解决什么问题提出的BN

训练深度神经网络时，作者提出一个问题，叫做“Internal Covariate Shift”。
这个问题是由于在训练过程中，网络参数变化所引起的。具体来说，对于一个神经网络，第n层的输入就是第n-1层的输出，在训练过程中，每训练一轮参数就会发生变化，对于一个网络相同的输入，但n-1层的输出却不一样，这就导致第n层的输入也不一样，这个问题就叫做“Internal Covariate Shift”。

#### 2.1.2 BN的来源

**白化操作**–在传统机器学习中，对图像提取特征之前，都会对图像做白化操作，即对输入数据变换成0均值、单位方差的正态分布。 卷积神经网络的输入就是图像，白化操作可以加快收敛，对于深度网络，每个隐层的输出都是下一个隐层的输入，即每个隐层的输入都可以做白化操作。
在训练中的每个mini-batch上做正则化：
![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/bn01_meitu_1.png)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/bn_meitu_2.png)

#### 2.1.3 BN的本质

我的理解BN的主要作用就是：

- 加速网络训练
-  防止梯度消失

如果激活函数是sigmoid，对于每个神经元，可以把逐渐向非线性映射的两端饱和区靠拢的输入分布，强行拉回到0均值单位方差的标准正态分布，即激活函数的兴奋区，在sigmoid兴奋区梯度大，即加速网络训练，还防止了梯度消失。基于此，BN对于sigmoid函数作用大。sigmoid函数在区间[-1, 1]中，近似于线性函数。如果没有这个公式：

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419210458.png)

就会降低了模型的表达能力，使得网络近似于一个线性映射，因此加入了**scale 和shift。** 它们的主要作用就是找到一个线性和非线性的平衡点，既能享受非线性较强的表达能力，有可以避免非线性饱和导致网络收敛变慢问题。

### 2.2 **卷积分解（Factorizing Convolutions）**

#### 2.2.1 分解为更小的卷积

大尺寸的卷积核可以带来更大的感受野，但也意味着会产生更多的参数，比如5x5卷积核的参数有25个，3x3卷积核的参数有9个，前者是后者的25/9=2.78倍。因此，GoogLeNet团队**借鉴VGG的使用**可以用2个连续的3x3卷积层组成的小网络来代替单个的5x5卷积层，即在保持感受野范围的同时又减少了参数量，如下图：

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419211357.png)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/3.png) 

#### 2.2.2 分解为不对称卷积

可以看出，大卷积核完全可以由一系列的3x3卷积核来替代，那能不能再分解得更小一点呢？GoogLeNet团队考虑了nx1的卷积核。

使用**非对称卷积**。将nxn的卷积分解成1xn和nx1卷积的串联。如下图，3x3卷积可以分解为3x1和1x3卷积，节省33%的计算量。

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419214302.png)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419215454.png)

> 因此，任意nxn的卷积都可以通过1xn卷积后接nx1卷积来替代。GoogLeNet团队发现在网络的前期使用这种分解效果并不好，在中度大小的特征图（feature map）上使用效果才会更好（特征图大小建议在12到20之间）。

关于非对称卷积，论文中提到：

- 先进行 n×1 卷积再进行 1×n 卷积，与直接进行 n×n 卷积的结果是等价的。
- 非对称卷积能够降低运算量，原来是 n×n 次乘法，分解之后，变成了 2×n 次乘法了，n越大，运算量减少的越多。
- 虽然可以降低运算量，但这种方法不是哪儿都适用的，这样的非对称卷积不要用在靠近输入的层，会影响精度，要用在较高的层，**非对称卷积在图片大小介于12×12到20×20大小之间的时候，效果比较好。**

### 2.3 **降低特征图大小**

一般情况下，如果想让图像缩小，可以有如下两种方式：

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419215846.png)

先池化再作Inception卷积，或者先作Inception卷积再作池化。但是方法一（左图）先作pooling（池化）会导致特征表示遇到瓶颈（特征缺失），方法二（右图）是正常的缩小，但计算量很大。为了同时保持特征表示且降低计算量，将网络结构改为下图，使用两个并行化的模块来降低计算量（卷积、池化并行执行，再进行合并）

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419215940.png)

Inception v2中有以下三种不同的Inception 模块：

- 第一种模块（figure 5）用在35x35的feature map上，主要是将一个5x5的卷积替换成两个3x3的卷积

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419221509.png)

- 第二种模块（figure 6），进一步将3*3的卷积核分解为nx1 和 1xn 的卷积，用在17x17的feature map上，具体模块结构，如下图：

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419221543.png)

- 第三种模块（figure 7）主要用在高维特征上，在网络中用于8x8的feature map上，具体结构如下图：

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419221607.png)



使用Inception V2作改进版的GoogLeNet，网络结构图如下：

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419220014.png)

> 注：上表中的Figure 5指没有进化的Inception，Figure 6是指小卷积版的Inception（用3x3卷积核代替5x5卷积核），Figure 7是指非对称版的Inception（用1xn、nx1卷积核代替nxn卷积核）。

经实验，模型结果与旧的GoogleNet相比有较大提升，如下表所示：

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200419220158.png)

## 三、Inception V3

论文地址：[Rethinking the Inception Architecture for Computer Vision](http://arxiv.org/abs/1512.00567)

 Inception v3整体上采用了Inception v2的网络结构，并在优化算法、正则化等方面做了改进,具体如下：

1. 优化算法使用RMSProp替代SGD

2. 辅助分类器中也加入BN操作

3. 使用Label Smoothing Regularization（LSR）方法 ，来避免过拟合，防止网络对某一个类别的预测过于自信，对小概率类别增加更多的关注。（关于LSR优化方法，可以参考下这篇博客https://blog.csdn.net/lqfarmer/article/details/74276680）

4. 最重要的改进是卷积分解（Factorization），将7x7分解成两个一维的卷积（1x7,7x1），3x3也是一样（1x3,3x1）
5. 网络输入从224x224变为了299x299。

## 四、Inception V4

论文地址：[Inception-v4, Inception-ResNet and the Impact of Residual Connections on Learning](http://arxiv.org/abs/1602.07261)

中心思想：这篇论文一口气提出了三个网络，Inception v4、Inception-ResNet v1和Inception ResNet v2，主要就是看到ResNet觉得挺好，就拿来和Inception模块结合一下，加shortcut。

引入了residual connection直连，把Inception和ResNet结合起来，让网络又宽又深，提除了两个版本：

- Inception-ResNet v1：Inception加ResNet，计算量和Inception v3相当，较小的模型
- Inception-ResNet v2：Inception加ResNet，计算量和Inception v4相当，较大的模型，当然准确率也更高

![image-20200421085423627](assets/image-20200421085423627.png)

Inception-ResNet-v1的Inception模块与原始Inception模块对比，增加shortcut结构，而且在add之前使用了线性的1x1卷积对齐维度。对于Inception-ResNet-v2模型，与v1比较类似，只是参数设置不同。

Inception-ResNet-v1的Inception模块：

1. Inception module都是简化版，没有使用那么多的分支，因为identity部分（直接相连的线）本身包含丰富的特征信息；
2. Inception module每个分支都没有使用pooling；
3. 每个Inception module最后都使用了一个1x1的卷积（linear activation），作用是保证identity部分和Inception部分输出特征维度相同，这样才能保证两部分特征能够相加。

### 4.1 Inception v4

作者称，由于历史的原因，Inception v3继承了太多的历史包袱，看起来过于复杂（是的= =），所以设计并非最优的，技术上的限制主要是为了模型能在DistBelief进行分布式训练，做了让步，而现在迁移到Tensorflow了（论文顺便给Tensorflow打了一波广告），所以去掉不必要的历史包袱，做一个简单一致的网络设计，所以有了Inception v4：

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200420224728.png)

Stem是Inception-v4结构的主干，起到基本特征提取的作用。Inception module适用于CNN的中间层，处理高维特征。若直接将Inception module用于低维特征的提取，模型的性能会降低。

Inception-v4使用了4个Inception-A module、7个Inception-B module、3个Inception-C module，起到高级特征提取的作用。并且**各个Inception module的输入输出维度是相同的**，Inception-A、Inception-B、Inception-C分别处理输入维度为35x35、17x17、8x8的feature map，这种设计是懒人式的，即直接告知哪个module对哪种size的feature map是最合适的，你只需要根据size选择对应的module即可。

Inception-v4使用了两种Reduction模块，Reduction-A和Reduction-B，作用是在避免瓶颈的情况下减小feature map的size，并增加feature map的depth。Reduction module避免瓶颈的原理在论文[Rethinking the Inception Architecture for Computer Vision](https://arxiv.org/pdf/1512.00567.pdf)中有详细阐述，简单地说就是多个分支同时减小size，最终将多个分支的输出在depth维上合并。

Inception-v4使用了[Network in Network](https://arxiv.org/pdf/1312.4400.pdf)中的average-pooling方法避免full-connect产生大量的网络参数，使网络参数量减少了8x8x1536x1536=144M。最后一层使用softmax得到各个类别的类后验概率，并加入Dropout正则化防止过拟合。

> **注意：**后文中，若无特殊说明，图中没有标注stride=2和符号V的操作默认为是stride=1，padding=SAME；
>
> 若标注符号V默认padding=VALID。

#### Stem (299x299x3 → 35x35x384)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421081528215.png)

> 71x71x192的filter concat后接的3x3 Conv应该是(192 stride=2 V)。

假设网络输入图像尺寸是299x299x3，经过Stem后输出为35x35x384。为了便于理解feature map的维度变化，给出size的计算式：o=⌈(i-k+2p)/2 + 1⌉，其中o为输出size，i为输入size，k为kernel size，p为padding。

#### Inception-A (35x35x384 → 35x35x384)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421081954400.png)



#### Reduction-A (35x35x384 → 17x17x1024)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200421082224.png)

> 其中k=192, l=224, m=256, n=384。

#### Inception-B (17x17x1024 → 17x17x1024)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421082519760.png)

> **疑问：**第三个通道中的两个1x7 Conv可能是作者笔误，Github上该部分是1x7 Conv和7x1 Conv。

#### Reduction-B (17x17x1024 → 8x8x1536)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421082808600.png)

#### Inception-C (8x8x1536 → 8x8x1536)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421082945591.png)

### 4.2 Inception-ResNet-v1

Inception-ResNet网络是在Inception模块中引入ResNet的残差结构，它共有两个版本，其中Inception-ResNet-v1对标Inception-v3，两者计算复杂度类似，而Inception-ResNet-v2对标Inception-v4，两者计算复杂度类似。Inception-ResNet网络结构如下图所示，整体架构与Inception类似。

![Inception-ResNet网络结构](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421084213418.png)

#### Stem

右图两个分别是Inception-ResNet-v1和Inception-ResNet-v2网络的stem模块结构，也即是Inception-v3和Inception-v4网络的stem模块。

Stem模块结构与Inception-v3相同：

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421084509957.png)

#### Inception-ResNet-A(35x35x384 → 17x17x1024)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421084627193.png)

#### Reduction-A(35x35x384 → 17x17x1024)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421085328624.png)

#### Inception-ResNet-B   (17x17x1024 → 17x17x1024)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421085917972.png)

#### Reduction-B (17x17x1024 → 8x8x1536)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421090103964.png)

#### Inception-C (8x8x1536 → 8x8x1536)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421090241300.png)

### 4.3 Inception-ResNet-v2

Inception-ResNet网络是在Inception模块中引入ResNet的残差结构，它共有两个版本，其中Inception-ResNet-v1对标Inception-v3，两者计算复杂度类似，而Inception-ResNet-v2对标Inception-v4，两者计算复杂度类似。Inception-ResNet网络结构如下图所示，整体架构与Inception类似。

![Inception-ResNet网络结构](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421084213418.png)

#### Stem

Stem模块结构与Inception-v4相同：

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421081528215.png)

#### Inception-ResNet-A(35x35x384 → 17x17x1024)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421091129972.png)

#### Reduction-A(35x35x384 → 17x17x1024)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200421082224.png)

#### Inception-B (17x17x1024 → 17x17x1024)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421091222927.png)

#### Reduction-B (17x17x1024 → 8x8x1536)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421092740552.png)

#### Inception-C (8x8x1536 → 8x8x1536)

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421091253374.png)

### 4.4 性能

不同Inception网络的在ImageNet上的对比结果如下表所示，可以看到加入残差结构，并没有很明显地提升模型效果。但是作者发现残差结构有助于加速收敛。所以作者说没有残差结构照样可以训练出很深的网络。

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/image-20200421093114501.png)

### 4.5 Residual Connection

作者重新研究了下residual connection的作用，指出residual connection并不会明显提升模型精度，而是会加快训练收敛：

> Although the residual version converges faster, the final accuracy seems to mainly depend on the model size.

作者发现，如果过滤器数目超过1000，残差网络将变得不稳定，并且网络在训练的早期就‘死亡’了，即迭代上万次之后，在平均池化层之前的层只输出0。即使降低学习率、添加额外的BN层也无法避免。

作者发现，在将残差与其上一层的激活值相加之前，将残差缩放，这样可以使训练过程稳定。通常采用0.1至0.3之间的缩放因子来缩放残差层，然后再将其添加到累加层的激活层。如下图所示，下图的Inception框可以用任意其它sub-network替代，输出乘以一个很小的缩放系数，通常0.1左右，再相加再激活：

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/20200421094056.png)

代码展示得更加清晰：

```python
def forward(self, x):
    out = self.conv2d(x)  # 这里可以是卷积层、可以是Inception模块等任意sub-network
    out = out * self.scale + x  # 乘以一个比例系数再相加
    out = self.relu(out)
    return out
```

引入scale可以解决这个问题，并且进入scale以后，并不会降低精度还能使得网络更加稳定。

## 五、GoogLeNet Tensorflow2.0实战

GitHub地址：https://github.com/freeshow/ComputerVisionStudy

## 六、参考链接

- https://my.oschina.net/u/876354/blog/1637819
- https://blog.csdn.net/abc13526222160/article/details/95472241
- https://juejin.im/post/5db90c41e51d4529e9478258

![](https://freeshow.oss-cn-beijing.aliyuncs.com/blog/公众号宣传码.png)