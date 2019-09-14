

## 文章内容：
### abstract：
介绍了一下文章思路，用cell来构建网络，并用DAG来搜索一个cell的构建，CAS的目的是构建一个在有限资源约束下的网络来进行图像分割。最后在cityspaces 和 camvid 上进行了验证。

### introduce & Relate work：
首先介绍了一些其他的工作，首先基于CNN的图像语义分割 运用一些技术如dilated convolutions 增加了感受野，并且不丢失信息，但是推理速度会变慢。 于是就有一些手工的方法提高速度 如（resizing or cropping the input ,pruning the network channels） 但是这个有三个缺点：1 难以在运行速度和性能上平衡。2. 手工设计很耗费时间 3. 泛化性差。

然后就是由 DART 和 NasNet 这两篇论文启发来进行模型的构造
DART:提供了梯度更新的想法， 梯度更新搜索速度更快
NASNet: 提供了构建cell来减小搜索空间提高泛化性的思路

之后还介绍了一下前沿的一些工作，其中特别提到了一下DPC 和CAS十分契合 不过DPC只优化了 multi-scale 没有优化其他的而且使用random 搜索。

### Customizable Architecture Search
#### 可微搜索算法：
前一部分描述了DART中可微搜索结构的原理，后一部分开始描述基于算法的改进。
将计算花费考虑了进去，并给出了新的 α更新公式。
![新的结构代价函数](https://upload-images.jianshu.io/upload_images/19070527-27a1e9a401660be8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![Lcost 计算](https://upload-images.jianshu.io/upload_images/19070527-7e4c0de9f47e03cf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)![α的更新](https://upload-images.jianshu.io/upload_images/19070527-5f43db61c7b6c7fa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中$c_o$的计算方法为 在网络中将所有节点均设置为一个操作然后用一种指标去衡量他(CPU/GPU time, parameters num,MAC)记做$c_0^{'}$, 所以$c_0$被定义为 $c_o=c_o^{'}-c_{id}^{'}$。

#### backbone Architecture Search
这里介绍了一下 backbone 的搜寻具体算法
首先backbone 有 8个cell 前三个是卷积cell， 后5个是设计的cell.backbone 的大结构是人为设计好的，只需要搜索cell的结构，有两种第一种是normal cell,第二种是reduction cell。 reduction cell 会将图像的长宽减半。 每个cell 与6个节点，前两个是i-1,i-2 是前两个节点的输出，最后一个是output。
 网络中有几个operation可以选择，这些选择也选择了一些轻量的操作。来让模型运行的更快。


#### Multi-Scale Cell Search
这里大体和backbone的搜索差不多，不过由于只有一个 multi-scale cell，multi-scale cell的计算成本要远低于 backbone 部分，所以这里的节点数选择为 N=9, 在这输入之前小的 feature map 需要上采样。 Multi-Scale cell 的操作设计也选择了一些有助于帮助图像语义分割的操作。（SSP 和 ecomposed convolution）。

### Implementation&Experiments
#### implementation
实现部分：对backbone 和 muitl-scale 分别进行优化。
流程是这样的 ：
1.用语义分割任务来训练出最合适的 normal cell 和 reduction cell 结构，
2.用ImageNet 进行预训练来初始化，backbone的权重。

3. 再加上muitl-cell 来搜索最佳的结构，此时pre-trained 的 backbone的权重也同时被优化了，一箭双雕。

#### Experiments
在carspaces 和 camvid上进行了实验，首先观察了cell的变化，最开始倾向去简单操作然后，再倾向于复杂操作来提高mloU。
之后对比了 $\lambda$ 对性能的影响。以及constraition 的选择对 mloU的影响。结论是在 $\lambda$小的时候其增加对性能的影响较小。
最后用生成的模型和其他人工模型进行了对比，CAS取得了不俗的成功，并用camVid 数据集验证了繁华性。