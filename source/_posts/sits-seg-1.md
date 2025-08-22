---
title: 时序分割：U-TAE、TSViT、UTS-Former
date: 2025-08-15 20:16:32
tags:
  - Paper overview
categories:
  - 遥感论文
---

> 本文内容来源于网络博客及GPT生成内容，作者并未详细阅读论文原文

## U-TAE

### 1. 论文信息

| <span style='white-space: nowrap'>属性</span> | 详细信息                                                     |
| --------------------------------------------- | ------------------------------------------------------------ |
| **标题**                                      | 🔬 *[Panoptic segmentation of satellite image time series with convolutional temporal attention networks](http://openaccess.thecvf.com/content/ICCV2021/html/Garnot_Panoptic_Segmentation_of_Satellite_Image_Time_Series_With_Convolutional_Temporal_ICCV_2021_paper.html)*<br>🔬 基于卷积时空注意力网络的卫星图像时序**全景分割** |
| **会议**                                      | 📚 *Proceedings of the IEEE/CVF International Conference on Computer Vision (ICCV)* <br/>🏆 CCF-A |
| **日期**                                      | ⏲️ 2021                                                       |
| **作者**                                      | 👨‍🔬 [Vivien Sainte Fare Garnot](https://scholar.google.com/citations?user=2r5slBsAAAAJ&hl=zh-CN&oi=sra) |

该论文的主要工作为提出适用于 **卫星影像时序序列** | `SITS` 的<u>全景分割方法</u>，以及发布<u>全景标注数据集</u>（`PASTIS`）。在深入模型结构之前，我们不妨先了解目前 **像素级密集预测** | `dense predict` 任务中的典型场景。总体上，这些任务可以分为三类：*语义分割*、*实例分割*、*全景分割*。关于这些任务的详细讲解可以参考「[语义分割、实例分割、全景分割的区别 - huarzail](https://blog.csdn.net/huarzail/article/details/131739469)」与「[全景分割（CVPR 2019） - 77wpa](https://blog.csdn.net/i6101206007/article/details/126655228)」这两篇博客，这里我就简要总结一下这些任务：

* **语义分割**（`semantic segmentation`）：对图像中每个像素进行类别预测，输出一张 *完整* 的语义类别图，但不区分同类目标的不同实例。
* **实例分割**（`instance segmentation`）：在语义分割的基础上，为图中 **可数对象** | `thing` 输出 *类别标签* 与 *实例ID* ，但对 **不可数目标** | `stuff` 不进行处理。
* **全景分割**（`panoptic segmentation`）：将语义分割和实例分割统一到同一框架下，为**可数对象**给出 *类别标签* 和 *实例ID* ，对**不可数目标**仅给出 *类别标签* 。

> 以街景影像为例，这里的`thing`实际上是我们在训练时**关注**的可数对象（如人、车等可数对象），而`stuff`则指哪些我们不关心实例数量的类别（如天空、道路等不可数对象），对于这些像素，我们只需要对其语义类别标注即可。需要注意的是，某些`stuff`类别（如建筑）理论上也可以划分为若干实例，但由于任务设定我们将其视为`stuff`处理。

![01bf58f6e8d1b96ff968a194afcd6b6c](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/01bf58f6e8d1b96ff968a194afcd6b6c.png)

<center>语义分割、实例分割与全景分割</center>

在常规的图像语义分割任务中，输入数据通常表示为 $C_\text{in}\times H\times W$，模型只需输出一个尺寸为 $C_\text{out}\times H\times W$ 的特征图，即可完成像素级的分类预测。然而，当处理 ***SITS*** 数据时，除了光谱/通道维度 ($C$)，还需要显式区分 **时间维度 ($T$)**。如果仅将时间序列简单拼接在通道维度上，<u>往往难以捕获序列间的动态演化特征</u>。

因此，更常见的做法是将原始数据组织为 $X\in \mathbb{R}^{T\times C_\text{in}\times H\times W}$  的输入，其中，$T$ 表示时间步数，$C_\text{in}$ 表示输入光谱通道数。这样，模型能够同时感知时间序列信息与空间/光谱特征。在输出端，通常依旧需要得到一个尺寸为 $C_\text{out}\times H\times W$ 的特征图，用于后续任务。

该论文提出的模型架构也遵从上述SITS数据组织范式，其先使用<u>时空编码模块</u>（***U-TAE***）对时序数据进行特征提取得到形如 $C_\text{out}\times H\times W$ 的特征图，然后再使用<u>全景分割模块</u>（***PaPs***）实现后续的全景分割任务。同样，这篇论文在网上也有博客「[解读 - 小菜鸟](https://zhuanlan.zhihu.com/p/421147308)」对其解读。

### 2. 时空编码

时空编码模型名为UTAE，即 ***带时间注意力编码器的UNet*** 。其模型结构如下：

![image-20250817202741001](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/image-20250817202741001.png)

<center>figure 1.2. U-TAE时空编码模块</center>

对于像UTAE这类面向时序影像的 *密集预测模型*，一个核心设计要点在于：如何**消除特征张量中的时间维度**。具体而言，模型需要将跨时间序列的<u>特征压缩或聚合</u>，使得输出特征图中每个像素最终都由一个一维向量表示。通过这种方式，我们可以将原本的 ***时序密集预测问题*** 转化为一个 ***常规密集预测问题***，从而当前模型能够直接借鉴成熟的密集预测模型来完成余下工作。



#### 🎯 归一化

在深入UTAE模型结构前，我们来了解一下模型中常用的归一化操作。

首先，归一化操作是在做什么呢？给定某层的输出 $x$ ，归一化操作就是用某个轴上的**均值**和**方差**把输入数据标准化到标准正态分布（$\hat{x} \sim \mathcal{N}(0, 1)$），然后再用<u>可学习</u>的缩放和平移参数 $\gamma$、$\beta$ 复原尺度和偏置（`bias`）。
$$
\begin{align*}

\hat{x} = \frac{x-\mu}{\sqrt{\sigma^2+\varepsilon}},\;\;y=\gamma\hat{x}+\beta

\end{align*}
$$
模型引入归一化操作通常是为了训练数据时把**数值范围压缩到可控的区间**，使得模型在训练时能设置**更大学习率**，且模型**梯度更加稳定，能快速收敛**<sup>GPT</sup>。

模型中常有如下几类归一化方法，下面我们来详细了解一下它们。

![img](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/664aa7d50bdceb739d34f440d83d839f.png)

<center>激活（中间特征）| <code>activations</code> 归一化分类</center>

在详细介绍这些 *归一化* 方法前，我们先来逐步了解一下 **批** | `Batch` 这一概念：

在深度学习中，训练模型的核心目标是通过**梯度下降法**寻找期望的最优参数解。通常，我们会在训练开始时对模型参数进行初始化，然后通过反向传播算法根据梯度更新参数。理想情况下，如果我们能够获得模型在整个数据分布上的所有输出误差，就能计算出**真实梯度**（$\nabla \mathcal{L}(\theta) = \mathbb{E}_{x \sim \mathcal{D}}(\nabla l(x;\theta))$，其中 $\mathcal{D}$ 表示输入数据的真实分布，$\theta$ 表示模型参数），并据此进行精确的参数更新。

然而，实际情况并不允许我们访问整个数据分布。模型只能在有限的训练数据（训练集）上进行学习，因此我们必须使用小批量样本来估计梯度（**无偏估计**）。由此我们便为模型训练引入了 ***批*** 这一概念：通过在每次迭代中使用一个小样本集合计算梯度（$g_B$），我们既能降低计算开销，又能有效逼近真实梯度，从而实现高效的模型训练。
$$
\begin{align*}

g_B = \frac{1}{B}\sum_{i=1}^B\nabla l(x_i;\theta),\;\;\mathbb{E}[g_B] = \nabla \mathcal{L}(\theta)

\end{align*}
$$

> 在深度学习模型训练时，我们通常将数据分为 训练集 | `train` 、验证集 | `valid` 、测试集 | `test` 三个部分分别用于模型训练的各个阶段：
>
> 🔹 **训练集**用于训练模型参数，整个<u>前向传播</u>和<u>反向传播</u>都是使用的训练集。
>
> 🔹 **验证集**用于在模型训练时检查模型的 *泛化能力* ，我们通常在训练时，定期在验证集上评估模型，常用于对模型进行<u>超参数调参</u>（模型超参数、`Batch size`、`Epoch`等）以及停止<u>模型训练</u>。
>
> 🔹 **测试集**用于最终评估模型的泛化性能，不参与模型任何训练。

**批大小** | `batch_size` 的设置也是一个值得研究的课题。在实际训练中，我们通常采用 ***mini*-batch** 的方式进行前向和反向传播，而不是使用全数据（*full-batch*）或者逐样本（`batch_size=1`）。原因在于：

* 如果批大小设置过小，梯度估计的方差较大，训练过程噪声大，模型收敛会出现抖动；
* 如果批大小过大，虽然梯度噪声减少，但训练的**边际收益递减**，同时显存占用和<u>分布式训练</u>时的通信开销都会显著增加。

> 此外，批大小还会从如下角度影响训练<sup>GPT</sup>：
>
> 🔹 **稳定性与学习率**：大批量允许更大学习率；常见经验则是**线性缩放法则**：当批大小从 $B$ 变为 $kB$ 时，将学习率从 $\eta$ 调到 $k\eta$（配合 **warmup** 更稳）。也有人用 $\sqrt{k}$ 缩放，噪声更接近不变。
>
> 🔹 **数值与混合精度**：大批量常配合 **混合精度**（`AMP`）与**梯度缩放**（`loss scaling`），以避免下溢。
>
> 🔹 **泛化**：小批量产生的“梯度噪声”像一种隐式正则化，经验上常更容易得到平坦解、泛化更好；大批量可能出现“泛化间隙”，可用更强正则（权重衰减、数据增强、长 **warmup**、优化器如 *LARS/LAMB* ）缓解。

顾名思义，**批归一化** | `Batch Norm` 就是对一个**批次**中的元素进行归一化操作。在不同的任务场景中，归一化的对象和维度会根据数据结构调整：

* 在**图像处理任务**中，输入数据通常以`(batch_size, channels, height, width)`的形式存在。此时批归一化会针对**每个通道维度**上的分布进行归一化操作。
* 在**自然语言处理** 或 ***结构化数据任务*** 中，输入特征通常为`(batch_size, feature_dim)`的二维矩阵，批归一化则会对**每个特征维度**（即`feature_dim`的每个维度）上的分布进行归一化操作。

需要注意的是，模型在训练与推理时的*批归一化* 的统计量的**来源不同**。

**训练**时，`BN`会根据当前批次数据计算均值和方差，并基于这两个统计量完成归一化。同时，为了让模型在**推理**时能使用更贴近全局数据的统计量，训练过程中会通过 **指数滑动平均** | `EMA` 持续累积批次统计量，得到全局均值和全局方差的近似值。
$$
\begin{align*}

\mu_{\text{running}} &= \alpha\cdot \mu_{\text{running}} + (1-\alpha)\cdot \mu_B\cdot\sigma_{\text{running}}^2 \\\\

\sigma_{\text{running}}^2 &= \alpha\cdot \sigma_{\text{running}}^2 + (1-\alpha)\cdot \sigma_B^2

\end{align*}
$$
其中，$\alpha$ 为衰减系数（通常取 $0.9$ 或 $0.99$），用于平衡历史统计量和当前批次统计量的权重。

模型训练显然是一个异步过程，我们只有完成一个 *batch* 的前向传播+反向传播后，才能进行下一个 *batch* 的计算。当我们引入多卡分布计算时，实际上是将一个 *global batch* 均匀分配到各个计算卡上计算，其中的一个子批次（*local batch*）的大小为`local_batch_size = global_batch_size / num_gpus`。而`BN`要求在训练时计算一个 *global batch* 内训练数据的**均值**和**方差**，这就需要我们在训练时，把所有 GPU 上的 *local batch* 的统计量**聚合**起来，再算全局的均值和方差。

其他 *归一化* 方法的特点以及使用常见如下表所示：

| 归一化方法                                                | 常用场景                     | 优点                                                         | 缺点 / 注意事项                                    |
| --------------------------------------------------------- | ---------------------------- | ------------------------------------------------------------ | -------------------------------------------------- |
| **BatchNorm**                                             | CNN（大 *batch* 训练）       | 缓解内部协变量偏移，加速收敛，具有轻微正则化效果             | 小 *batch* 时统计不稳定；对 RNN 或序列模型不太适用 |
| **LayerNorm**                                             | Transformer / NLP / ViT      | 对 *batch* 尺寸无依赖，适合堆叠深层网络，序列模型中稳定性高  | 对 CNN 的空间特征处理效果不一定理想；增加计算开销  |
| <span style='white-space: nowrap'>**InstanceNorm**</span> | 风格迁移 / GAN               | **消除样本间风格差异**，使生成图像风格统一                   | 可能破坏全局统计信息，损失图像内容一致性           |
| **GroupNorm**                                             | 小 *batch* CNN / 检测 / 分割 | 对 *batch* 尺寸不敏感，适合小 *batch* 或高分辨率输入，在 CNN 中稳定 | 需调节组数 G，计算稍高；对特征通道划分敏感         |

#### 🪢 空间编/解码

在介绍UTAE空间编码器前，我们不妨先回忆一下UNet的编码器结构。UNet 通过**卷积块**+**下采样**实现对输入影像的多尺度特征提取。具体来说，第 $l$ 层卷积块会提取输入的特征信息，然后通过下采样操作将特征图尺寸减半，作为第 $l+1$ 层卷积块的输入，从而逐步捕捉更大感受野的空间信息。

UTAE处理的是SITS数据，其输入形式为 $X \in \mathbb{R}^{T \times C \times H \times W}$ ，其中 $T$ 表示时间步长，$C$ 为通道数，$H$ 和 $W$ 分别为空间高度和宽度。虽然输入数据与UNet不同，但在空间编码阶段，UTAE对每一个时间步 $t$​ 的特征图进行的操作，本质上类似于UNet编码器中的**卷积+下采样**流程。也就是说，对于每个时间步，UTAE的空间编码器会提取局部空间特征，同时通过多层结构捕获更高层次的空间语义信息。论文采用如下公式描述这一过程：
$$
\begin{gather*}

e^l = [\mathcal{E}_l(e_t^{l-1})]_{t=0}^T\;\;\text{for}\;\;l\in [1,L]

\end{gather*}
$$
其中，$[\cdot]$ 表示沿**时间维度**的拼接操作，$e_t^{l-1} \in \mathbb{R}^{C_{l-1}\times H_{l-1}\times W_{l-1}}$ 表示时间步 $t$ 在第 $l-1$ 层的输出特征图（$H_l = H/2^{l-1},\;W_l = W/2^{l-1}$），$\mathcal{E}_l(\cdot)$ 表示第 $l$ 层空间编码器。

UTAE空间编码器的核心计算单元是`DownConvBlock`，其结构如下：

```python
class DownConvBlock(TemporallyShareBlock):
  def forward(self, input):
    out = self.down(input)
    out = self.conv1(out)
    out = out + self.conv2(out)	# residual connect, not skip connect
    return out
```

需要注意的是，其中`Conv`组件的具体结构为：(`Conv2D` → `GN` → `ReLU`) $\times$ `kernel_num`，即UTAE空间编码器 *组归一化* 是在卷积单元中实现的。

由于UTAE的空间编码器是 **以时间步 $t$ 为单位**进行空间特征提取的，因此在跨层传递过程中，**时间维度 $T$ 不发生变化**。 在实际实现中，作者采取了一个简洁的方式：UTAE会先将**时间维度**和**批次维度**展开到一起（`input.view(b * t, c, h, w)`），然后再经过`DownConvBlock`处理后还原到原维度（`out = out.view(b, t, c, h, w)`）。

> 📌 如果想深入理解实现细节，可以直接阅读 [UTAE-paps - utae.py](https://github.com/VSainteuf/utae-paps/blob/main/src/backbones/utae.py)；

通过这种方式，UTAE能够在保持时间顺序信息的同时，对每个时间步的空间特征进行高效编码，为后续的时序建模提供富含空间语义的输入。最终第 $l$ 层输出的特征图尺寸为 $T\times C_l\times H_l\times W_l$ 。

由UTAE结构图可知，UTAE空间编码器的各尺度输出在<u>经过时间编码后消除了时间维度</u>，其尺寸变为 $C_l\times H_l\times W_l$ 。因此，UTAE空间解码器的实现则与UNet几乎一致，论文采用如下公式描述这一过程：
$$
\begin{gather*}

d^l = \mathcal{D}_l([\mathcal{D}_l^\text{up}(d^{l+1}), f^l])\;\;\text{for}\;\;l\in [1,L-1]

\end{gather*}
$$
其中，$[\cdot ]$ 表示按通道维度拼接。最终，经过UTAE空间编/解码器的处理，我们得到了尺寸为 $C_1\times H \times W$ 的特征图用于全景分割输入。

#### ⌚️ 时间编码

UTAE的时间编码模块，本质上是通过**注意力机制**来对**时序特征图**进行信息聚合。相比传统方法，作者在两个方向上做了改进：

* **卷积循环UNet**（CR-UNet）：UTAE 通过**对最低分辨率的时序注意力掩码进行双线性插值**，使得各尺度的编码器特征都能参与时序建模。

  > Convolutional-recurrent U-Net networks [41, 36, 28] only process the temporal dimension of the lowest resolution feature map with a temporal encoder. The rest of the skip connections are collapsed with a simple temporal average. This prevents the extraction of spatially adaptive and parcel-specific temporal patterns at higher resolutions. 
  >
  > 卷积循环U-Net网络[41, 36, 28]**仅**通过时间编码器处理最低分辨率特征图的时间维度。其余的跳跃连接通过简单的**时间平均**（即使用求均值的方法压缩时间维度）进行合并。这阻止了在更高分辨率下提取空间自适应和地块特异性时间模式。

* **轻量级时间注意力编码器模块**（Lightweight-Temporal Attention Encoder, L-TAE）：作者选择了 L-TAE 作为时间编码器，同时借助**通道分组策略**提升计算效率与特征对齐稳定性。

  > Based on its performance and computational efficiency, we choose the Lightweight-Temporal Attention Encoder(L-TAE) [10] to handle the temporal dimension. The L-TAE is a simplified multi-head self-attention network [44]in which the attention masks are directly applied to the input sequence of vectors instead of predicted values. Additionally, the L-TAE implements a channel grouping strategy similar to Group Normalization [49].
  >
  > 基于其性能和计算效率，我们选择轻量级时空注意力编码器（L-TAE）[10]来处理时空维度。L-TAE是一种简化的多头自注意力网络[44]，其中注意力掩码直接应用于向量输入序列，而非预测值。此外，L-TAE实现了与**组归一化**[49]类似的**通道分组策略**。

下面我们通过论文中相关公式，来详细了解一下模型的这部分计算过程：
$$
\begin{gather*}

a^{L,1},\cdots,a^{L,G} = \operatorname{LTAE}(e^L),\; \text{applied pixelwise.} \tag{1}\\\\
a^{l,g} = \operatorname{BiLinear(a^{L,g}, (T\times H_l\times W_l))} \tag{2}\\\\
f^l = \operatorname{Conv}_{1\times 1}^l\left( \left[ \sum_{t=1}^Ta_t^{l,g} \odot e_t^{l,g} \right]_{g=1}^G \right)\tag{3}

\end{gather*}
$$
公式$(1)$中，UTAE首先在通道维度上进行**分组**，然后在每个分组内通过 *L-TAE* 计算时序注意力分数。这里得到了最低分辨率时序权重矩阵 $a^{L,g} \in \mathbb{R}^{T\times H_L\times W_L}$，其取值范围为 $[0,1]$。从 $a^{L,g}$ 的形状不难看出，同一个分组内的所有通道共享同一组时序权重矩阵，这种设计可以在保证计算效率的同时，避免对单个通道过度独立建模。

> 既然UTAE的**时间编码**和**空间编码**中都是使用的 *分组策略* 进行计算，这里也引用一下GPT对此的解答：
>
> * **🔹 组归一化**
>
>   在遥感任务中，输入数据通常是**高分辨率、多时间步的影像序列**。如果直接用**批归一化**，当 *batch_size* 太小时，统计量就会出现不稳定的问题。而**实例归一化**在每个样本、每个通道上独立归一化，虽然避免了 ***BN*** 的问题，但可能会<u>过度消除</u>跨空间的对比信息，对语义特征提取不够友好。
>
>   **组归一化**在通道维度分组后进行归一化，即不依赖 *batch_size* ，适合小 *batch* 训练，又保留了一定通道间的统计特征，相比 ***IN*** 更能保留判别性信息。
>
> * **🔹 L-TAE分组策略**
>
>   由于在UTAE空间编码阶段，卷积特征已经通过 ***GN*** 归一化，如果在 *L-TAE* 中改用其他归一化方式，会导致**特征分布不一致**，使得跨模块的特征对齐变差。使用相同的 ***GN分组策略***，可以保证特征的统计性质在 ***空间编码 → 时序编码*** 的过程中保持稳定。

公式$(2)$表示将 *初始* 时序权重矩阵与各尺度时序特征图进行**尺度对齐**操作，保证每一层次时序特征图都能使用时序加权矩阵进行信息聚合操作。

公式$(3)$中，模型利用**时序权重矩阵**对**时序特征**进行加权聚合。

* 其中，$\odot$ 表示逐元素相乘（支持**通道广播**）。也就是说，在某个时刻 $t$，特征图通道向量的<u>每个空间位置</u>都会与注意力<u>权重</u>（$[0,1]$ 范围数值）对应相乘；
* $\sum_{t=1}^T$ 表示对时间维度求和，从而得到时序聚合后的空间特征；
* $[\cdot]_{g=1}^G$ 表示将不同分组的结果在通道维度拼接，得到一个形如 $C_\text{in} \times H_l \times W_l$ 的张量。

最后，模型在拼接后的特征上施加一个`1×1 Conv2D`，实现跨分组信息的融合，并生成第 $l$ 层的编码器跳跃连接输入。

接下来，我们来看看时间编码中的 ***L-TAE*** 模块，这是作者在另一篇论文中提出的一个轻量级时间注意力机制。下面我们通过源码来理解其核心结构与计算逻辑：

```python
class LTAE2d(nn.Module):
    def __init__(
        self,
        in_channels=128,
        n_head=16,
        d_k=4,
        mlp=[256, 128],
        dropout=0.2,
        d_model=256,
        T=1000,
        return_att=False,
        positional_encoding=True,
    ):
        pass

    def forward(self, x, batch_positions=None, pad_mask=None, return_comp=False):
        sz_b, seq_len, d, h, w = x.shape
        
        # 1) 处理 pad_mask，使其扩展到空间维度
        if pad_mask is not None:
            pad_mask = (
                pad_mask.unsqueeze(-1)
                .repeat((1, 1, h))
                .unsqueeze(-1)
                .repeat((1, 1, 1, w))
            )  # (B,T,H,W)
            pad_mask = (
                pad_mask.permute(0, 2, 3, 1).contiguous().view(sz_b * h * w, seq_len)
            )
        
        # 2) 展开空间维度, 形状变化: (B,T,C,H,W) → (B·H·W,T,C), 记 N=B·H·W
        out = x.permute(0, 3, 4, 1, 2).contiguous().view(sz_b * h * w, seq_len, d)
        
        # 3) 输入归一化: GroupNorm(num_groups=n_head, num_channels=in_channels)
        # 这里先把形状变为 (N,C,T) 以便 GroupNorm , 然后再还原为 (N,T,C)
        out = self.in_norm(out.permute(0, 2, 1)).permute(0, 2, 1)

        # 4) (可选) 使用Conv1D实现通道投影: C → d_model
        # 这里的形状变化同上, 最终输出的尺寸为 (N,T,C|d_model)
        if self.inconv is not None:
            out = self.inconv(out.permute(0, 2, 1)).permute(0, 2, 1)

        # 5) (可选) 加入位置编码(时间编码)
        if self.positional_encoder is not None:
            # 每个Batch都有一个时间序列表示，各影像的时间刻
            # 这操作只是把一个值(T)转换为了一个矩阵(T,H,W), 让每幅影像的每个位置都有时间刻对应
            # 时间刻的具体值可能是 *序列号* ，也可能 *DayOfYear* 
            bp = (
                batch_positions.unsqueeze(-1)	# (B, T) → (B, T, 1)
                .repeat((1, 1, h))
                .unsqueeze(-1)
                .repeat((1, 1, 1, w))
            )  # (B,T,H,W)
            bp = bp.permute(0, 2, 3, 1).contiguous().view(sz_b * h * w, seq_len) # (N,T)
            out = out + self.positional_encoder(bp) # (N,T,d_model)

        # 6) 多头注意力
        # 输入 out: (N,T,d_model)
        # 输出 out: (Head,N,d_v), 其中 d_v = d_model/Head
        # 输出 attn: (Head,N,T)，表示每个 head 对时间维度的注意力分布
        out, attn = self.attention_heads(out, pad_mask=pad_mask)
        
        # 这里的Q时模型中的可学习参数，每一个Head一个，不依赖外部输入:
        # 	self.Q = nn.Parameter(torch.zeros((n_head, d_k))) 
        # 	其形状为(Head, d_k)
        # 在实际计算时，UTAE会复制N个Q构成矩阵, 以便快速计算
        # 	q = torch.stack([self.Q for _ in range(sz_b)],dim=1).view(-1, d_k)
        # 	其形状为(Head·N, d_k)
        # 同样, k,v也会进行将Batch和Head维度展开，以便快速计算
        # 	k = self.fc1_k(v).view(sz_b, seq_len, n_head, d_k)
        # 	k = k.permute(2, 0, 1, 3).contiguous().view(-1, seq_len, d_k)
        # 	其形状为(Head·N, T, d_k)
        
        # 	v = torch.stack(v.split(v.shape[-1] // n_head, dim=-1)).view(n_head * sz_b, seq_len, -1)
        # 	其形状为(Head·N, T, d_v), 其中 d_v = d_in/Head
        
        # 然后我们便可得到注意力分数矩阵 attn, 其形状为 (Head·N, 1, T)
        
        # 7) 最低分辨率时序特征图特征聚合结果
        # 这里的 out 实际上就是使用 attn 对最低分辨率时序特征图进行时间维度信息聚合的结果
        # 在UTAE实现代码中，不需要再次对该分辨率做聚合
        out = (
            out.permute(1, 0, 2).contiguous().view(sz_b * h * w, -1)
        )  # Concatenate heads
        out = self.dropout(self.mlp(out))
        out = self.out_norm(out) if self.out_norm is not None else out
        out = out.view(sz_b, h, w, -1).permute(0, 3, 1, 2) # (B,C_out,H,W)

        # 7) 重塑注意力权重矩阵为 (Head,B,T,H,W)
        attn = attn.view(self.n_head, sz_b, h, w, seq_len).permute(
            0, 1, 4, 2, 3
        )  # (Head,B,T,H,W)

        if self.return_att:
            return out, attn
        else:
            return out
```

从代码逻辑不难发现，**LTAE实际上是基于注意力机制实现的**，其计算流程大致为：先将最低分辨率影像在空间维度拉伸，然后将拉伸后的时序数据（需做位置编码，尺寸为`(N,T,C)`）作为 *Key*、*Value* 与一个可训练查询向量 *Q* 进行注意力计算，得到了 *Q* 针对不同时间刻、不同位置的像素的注意力权重，然后我们便可使用这些权重在<u>时间维度</u>进行**信息聚合**。

> 在理解 *LTAE* 的时间注意力机制时，可以将其中的查询向量 ***Q*** 与 ***ViT*** 的`[CLS]` *token* 做一个对比。二者在形式上虽然不同，但本质上都是通过为注意力机制引入可训练参数来完成**全局聚合信息**。
>
> 在 *LTAE* 中，查询向量 *Q* 对全空间位置共享，对每个空间位置的**时序特征序列**进行注意力权重计算，并用该权重信息聚合时序信息。换句话说，*Q* 并不是直接表征输入的一部分，而是充当了一个“**全局探针**”，帮助模型在不同时间片之间建立权重分布。而在 *ViT* 中，我们也是透过`[CLS]` *token* 观察输入特征序列，从而得到空间尺度的注意力权重矩阵（需重塑差值），并用于`[CLS]` *token* 与各 *patch token* 的信息聚合。
>
> 从**注意力权重分布**的角度来看，两者又有高度相似之处。在 *ViT* 中，常见的可视化方法是观察 `[CLS]`（记作 $e_0$）对其他 *patch token*（记作 $e_1, e_2, \dots, e_N$）的注意力权重，先得到一个尺寸为 $1\times N$ 的注意力向量，再重塑插值为 $H\times W$ 形状的权重矩阵；而*LTAE* 则为每个<u>空间位置</u>都计算一个 $1\times T$ 的注意力向量。两者本质上非常接近——都是在用一个序列外的查询向量，去查询序列的注意力分布情况。
>
> 此外，需要注意的是，*ViT* 的`[CLS]`是在**输入阶段**就引入的，并且与其他 *token* 一样需要位置编码，以保留**顺序信息**；而 *LTAE* 的 Q 则是**模块内部的固定查询向量**，不承担输入表征的职责，因此无需位置编码。
>

从代码逻辑不难发现，**LTAE实际上是基于注意力机制实现的**，其计算流程大致如下：

1. **空间展平**：将输入影像在空间维度展开，把每个像素点的位置视为一个“序列样本”，这样输入由`(B,T,C,H,W)`转换为`(N,T,C)`，其中`N = B·H·W`。

2. **归一化与通道投影**：对时间维度上的特征做 *组归一化*，可选使用`Conv1d`投影到新的特征维度`d_model`。

3. **时间位置编码**：若启用，会为每个时间步加入位置编码，使模型具备 *时间顺序感知能力* ：

   ```python
   class PositionalEncoder(nn.Module):
       def __init__(self, d, T=1000, repeat=None, offset=0):
           super(PositionalEncoder, self).__init__()
           # token向量维度, 这里实际上是 d_model // n_head
           self.d = d
           self.T = T
           # 支持重复维度, 可以把生成的正余弦编码重复拼接多份. 扩大到更高维度
           self.repeat = repeat
           # 对应经典Transformer位置编码的频率分母: 10000^{2i/d}
           # // 2操作确保偶数和奇数索引使用相同的频率基数
           self.denom = torch.pow(
               T, 2 * (torch.arange(offset, offset + d).float() // 2) / d
           )
           self.updated_location = False
   
       # 前向传播中输入的BP尺寸为: (N,T) 或 (B·H·W, T)
       # 在后续计算中将 N 视为 Batch_size
       # BP在模型训练时提供给utae, 然后再传给ltae和pe, 
       def forward(self, batch_positions):
           if not self.updated_location:
               self.denom = self.denom.to(batch_positions.device)
               self.updated_location = True
           # 位置编码计算核心部分: 
           # 1. batch_positions[:, :, None]给输入增加一个维度，形状变为 (B, T, 1)
           # 2. 除以self.denom得到不同频率的位置值，形状变为 (B, T, d)
           sinusoid_table = (
               batch_positions[:, :, None] / self.denom[None, None, :]
           )  # B x T x d
           sinusoid_table[:, :, 0::2] = torch.sin(sinusoid_table[:, :, 0::2])  # dim 2i
           sinusoid_table[:, :, 1::2] = torch.cos(sinusoid_table[:, :, 1::2])  # dim 2i+1
   
           # 如果设置了repeat参数，会将生成的位置编码在最后一个维度上重复拼接，扩展编码维度
           if self.repeat is not None:
               sinusoid_table = torch.cat(
                   [sinusoid_table for _ in range(self.repeat)], dim=-1
               ) # B x T x d_model
   
           return sinusoid_table
   ```

   需要注意的是，*LTAE* 使用的位置编码是在经典正弦余弦编码的基础上的**改进版本**。传统正弦余弦编码中，我们直接为每个 *token* 生成一个 $d_\text{model}$ 长度的位置编码，而该改进版本只需生成`d = d_model // n_head`维度的基础编码，通过重复拼接得到最终编码（仍为`d_model`维度），这样减少了存储和计算开销。

   > 当我们根据序号生成对应的正弦余弦编码时，使用的可能是图像的 *序列号* 或者 **一年中的天数** | `DOY` 。这里仅作猜测，具体实现还需阅读源码和 *LTAE* 原论文。

4. **多头注意力计算**：将输入序列作为 *Key*、*Value* 与可训练查询向量 *Q* 进行注意力计算。由于这里使用的是多头注意力，实际上LTAE是为每个头单独维护一个查询向量 *Q* 。

   此外，在实现时 *K* 是原序列经<u>线性变换</u>（`FC`）后得到的新序列，而 *V* 则是对原序列的通道维度进行<u>拆分</u>而得到的。

5. **信息聚合**：利用得到的注意力权重在时间维度聚合每个像素点的时序特征，最终输出 `(B,C,H,W)` 的聚合特征图。

代码中实现实现的信息聚合方法有三种：

* **`attn_group`**：将通道划分为`Head`组（同LTAE的组归一化分组），利用每个 *head* 的注意力权重对分组后的特征进行加权，最后拼接结果。这样保留了多头的多样性，输出 `(B,C,H,W)`。
* **`attn_mean`**：对所有 *head* 的注意力权重在 *head* 维度上求平均，得到一个`(B,T,H,W)`的统一权重，再对输入序列特征加权聚合。
* **`mean`**：不使用注意力，直接在时间维度取平均，作为最简单的 *baseline* 方法。

📖 这里就不过多介绍LTAE具体实现细节，进一步阅读可参考原论文：

| <span style='white-space: nowrap'>属性</span> | 详细信息                                                     |
| --------------------------------------------- | ------------------------------------------------------------ |
| **标题**                                      | 🔬 *[Lightweight Temporal Self-Attention for Classifying Satellite Image Time Series](https://link.springer.com/chapter/10.1007/978-3-030-65742-0_12)*<br>🔬 轻量级时空**自注意力机制**在卫星图像时间序列分类中的应用 |
| **信息**                                      | 👨‍🔬 [Vivien Sainte Fare Garnot](https://scholar.google.com/citations?user=2r5slBsAAAAJ&hl=zh-CN&oi=sra) \| ⏲️ 2020 |

### 3. 其他

一般来说，在语义分割任务中，经过**编码器—解码器结构**提取和还原特征图之后，直接接入一个`1×1 Conv2D`就可以完成像素级的分类预测。但本文所研究的任务是 *全景分割* 。为此，作者在UTAE模块之后，引入了 ***PaPs（Parcels-as-Points）*** 模块，从而构建出端到端的全景分割框架。该模块的整体结构如图所示：

![image-20250821142934606](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/image-20250821142934606.png)

<center>figure 1.4. <code>PaPs</code>全景分割</center>

由图可知，*PaPs* 模块主要由 ***边界框位置/大小预测***、***地物边界预测***、***地物类型预测*** 等多个子模块组成。通过这些组件，*PaPs* 能够同时输出对象类别、边界框以及实例掩码，从而实现全景分割所要求的「语义 + 实例」的统一建模。在此就不对 *PaPs* 内部的具体 *损失函数设计* 与 *实现细节* 展开说明，若想详细了解可参考论文原文及模型源代码。

此外，论文使用了 *UTAE* 做了语义分割实验对比，实验结果如下：

<img src="https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/image-20250821150149891.png" alt="image-20250821150149891" style="zoom: 35%;" />

<center>table 1.1. UTAE在语义分割任务中与其他方法的<strong>对比实验</strong>，以及模块<strong>消融实现</strong></center>

## TSViT

### 1. 论文信息

| <span style='white-space: nowrap'>属性</span> | 详细信息                                                     |
| --------------------------------------------- | ------------------------------------------------------------ |
| **标题**                                      | 🔬 *[Vits for sits: Vision transformers for satellite image time series](http://openaccess.thecvf.com/content/CVPR2023/html/Tarasiou_ViTs_for_SITS_Vision_Transformers_for_Satellite_Image_Time_Series_CVPR_2023_paper.html)*<br>🔬 适用于SITS的ViT：Transformer在卫星图像时间序列中的应用 |
| **会议**                                      | 📚 *Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)* <br/>🏆 CCF-A |
| **日期**                                      | ⏲️ 2023                                                       |
| **作者**                                      | 👨‍🔬 [Michail Tarasiou](https://scholar.google.com/citations?user=ty8-urQAAAAJ&hl=zh-CN&oi=sra) |

在 *ViT* 中，我们通常使用 *Transformer-only* 的主干网络来提取单幅图像的空间特征。当面对由多时相影像组成的 ***SITS*** 数据时，Transformer能否胜任呢？答案是可以的。

**TSViT**正是将Transformer应用到 *SITS* 的一种方法。论文面向时空遥感序列提出了一种**时空联合特征提取 主干网络** | `Backbone`。该网络能从时序影像中为某一 *patch* 提取时间维度与周围空间的特征信息，并将该特征用于 *pixel-wise* 和 *patch-wise* 等下游预测任务。下面我们来逐步学习下 *TSViT* 的模型结构。

### 2. 模型结构

这篇论文的 **$\text{3. Method}$** 部分对 *TSViT* 的 *整体架构* 和 ***关键改进*** 做了系统而清晰的阐述，非常值得一读。为了更好地理解其核心思想，接下来我们将沿着作者的思路，一步步拆解模型的各个组成模块，看看 *TSViT* 是如何在时空两个维度上同时建模，并最终实现对 *SITS* 数据的高效表征。

####  









## UTS-Former

### 1. 论文信息

| <span style='white-space: nowrap'>属性</span> | 详细信息                                                     |
| --------------------------------------------- | ------------------------------------------------------------ |
| **标题**                                      | 🔬 *[A temporal-spatial deep learning network for winter wheat mapping using time-series Sentinel-2 imagery](https://www.sciencedirect.com/science/article/pii/S0924271624002417)*<br>🔬 基于时间序列Sentinel-2遥感影像的冬小麦空间-时间深度学习网络 |
| **期刊**                                      | 📚 *[ISPRS Journal of Photogrammetry and Remote Sensing](https://www.sciencedirect.com/journal/isprs-journal-of-photogrammetry-and-remote-sensing)* <br/>🌍 地球科学1区 [TOP] |
| **日期**                                      | ⏲️ 2024                                                       |
| **作者**                                      | 👩‍🔬 [Lingling Fan](https://scholar.google.com/citations?user=5ysGWLYAAAAJ&hl=zh-CN&oi=sra) |



## 借物表

| 参考资料                                                     | 说明               |
| ------------------------------------------------------------ | ------------------ |
| [语义分割、实例分割、全景分割的区别 - huarzail](https://blog.csdn.net/huarzail/article/details/131739469) | CSDN文章｜UTAE参考 |
| [深度学习中的组归一化](https://blog.csdn.net/yuanlulu/article/details/84190971) | CSDN文章｜UTAE参考 |

