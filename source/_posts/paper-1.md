---
title: 论文速读：CRM、
date: 2025-07-10 14:01:27
tags:
  - Paper overview
categories:
  - 遥感论文
---

## CRM：使用光学与SAR影像的作物轮作制图

### 1. 论文信息

| <span style='white-space: nowrap'>属性</span>       | 详细信息                                                     |
| --------------------------------------------------- | ------------------------------------------------------------ |
| **标题**                                            | 🔬 *[Mapping Crop Rotation by Using Deeply Synergistic Optical and SAR Time Series](https://www.sciencedirect.com/science/article/pii/S1569843225000421)*<br>🔬 利用深度协同光学和合成孔径雷达时间序列绘制作物轮作图 |
| **期刊**                                            | 📚︎ *Remote Sensing* <br/>🌍 地球科学2区                        |
| **日期**                                            | ⏲️ 2021-10-17                                                 |
| **作者**                                            | 👩‍🔬 Yiqing Liu                                                |
| <span style='white-space: nowrap'>**关键词**</span> | `crop rotation mapping`、`deep learning`、`SAR and optical time series`、`dynamic feature extraction` |

本文的主要内容是提出一个**轮作作物制图**方法（CRM）。主要考虑的作物类型的物候如下图所示：

![image-20250711172145848](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/image-20250711172145848.png)

<center>figure 1.2. 使用的S1/S2数据日期与与物候期NDVI平均值</center>

通过上述基础作物类型的**组合**，论文选择的分类类型为：CF-SR、F-SR、F-DR、RS-SR、RS-DC、其他轮作类型（视为一种分类类型）、水、泥滩、植被、建设用地，共10个类别。

### 2. 卫星与样本数据

本文使用了GEE获取的Sentinel-1/2影像（SAR影像处理略），其中光学影像的去云使用GEE的S2云概率数据集去云，并提取了NDVI、EVI、**kNDVI**。选择kNDVI的原因及引用如下：

> while kNDVI has proven to have a higher sensitivity to vegetative **biophysical** and **physiological** parameters compared to classical vegetation indices.
>
> 与传统植被指数相比，kNDVI已被证明对植被的**生物物理**和**生理**参数具有更高的敏感性。
>
> | 属性     | 详细内容                                                     |
> | -------- | ------------------------------------------------------------ |
> | **论文** | 🔬 [*A Unified Vegetation Index for Quantifying the Terrestrial Biosphere*](https://scholar.google.com.hk/scholar?hl=zh-CN&as_sdt=0%2C5&q=A+Unified+Vegetation+Index+for+Quantifying+the+Terrestrial+Biosphere&btnG=) |
> | **其他** | 📚︎ *Science Advances* \| 综合期刊1区\[TOP\] \| 2021           |

这三个指数的计算方法如下：
$$
\begin{align*}

\text{EVI} &= 2.5 \times \frac{\text{NIR} - \text{red}}{\text{NIR} + 6 \times \text{red} - 7.5 \times \text{blue} + 1} \\\\

\text{NDVI} &= \frac{\text{NIR} - \text{red}}{\text{NIR} + \text{red}} \\\\

\text{kNDVI} &= \tanh \left( \left( \frac{\text{NIR} - \text{red}}{2\sigma} \right)^2 \right)

\end{align*}
$$
其中，$\sigma$ 是一个长度尺度参数，表示对植被稀疏度的敏感性。论文中设置为 $\sigma = 0.5 (NIR+red)$。

论文使用的实验数据为通过<u>实地调研/农民访谈</u>获取的3年（2018-2020）作物轮作数据，总共**689个地面样本**。然后对谷歌地球高分影像（2018）进行目视解译，添加了**201个非耕地样本**。论文通过对真实样本进行样本增强（将真实样本像素的上下左右像素归位同一类型样本）最终获得**4450像素**的样本；

由本段内容可知，论文中样本为像素基样本。最终增强后的样本数为 $5\times (689+201)=4450$。由于增强的样本与真实样本存在空间临近关系，在训练时需考虑**样本标签在近邻区域聚类的问题**。论文中训练时处理如下：

> When feeding our models, 80% of samples were randomly selected from the <u>**ground-truth collection**</u> for training, and the rest were used for validation. **To avoid accuracy uncertainty due to random training and validation split, we applied a fixed set of the shuffled dataset to all competing models**, which eliminated the risk of sample labels clustering in the near neighborhood.
>
> 在训练模型时，我们从<u>**真实数据集**</u>随机选取 80% 的样本用于训练，剩余部分用于验证。**为避免因随机划分训练集和验证集导致的准确率不确定性，我们对所有参比模型应用了固定的打乱数据集**，从而消除了样本标签在邻近区域聚集的风险。
>
> ---
>
> 对于论文原文我有如下疑问：
>
> *ground-truth collection* 对应论文的 *2.3. Ground-Truth Data* 小节，应该是指经过样本增强后的数据。对比实验中都采用**同一打乱顺序**的数据集是否可以消除样本临近聚类的风险存疑（都有影响 = 都没影响🤨）？

### 3. 网络结构

CRM网络的总体任务是接收时序光学指数、SAR数据，输出一个像素基分类结果，网络结构如下图所示：

![image-20250731140623911](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/image-20250731140623911.png)

<center>figure 1.3. CRM网络结构</center>

CRM的两个分支采用相同的处理模块，然后将处理后的时序特征按对应**时间点**拼接后放入全连接层中。下面我们详细了解一下该网络的各个模块。

#### 🕙 时序特征提取模块

论文中将CRM网络中处理光学/SAR特征的模块成为 **流** | `stream` ，将这两个模块称为 **双流** | `dual-stream` 。GPT告诉我这两种称呼近乎等价，但`stream`强调的是两个并行模块的输入特征的异质性，而`branch`则强调这两个模块在网络结构上的分支关系，不同分支可能用于提取同一输入的不同特征，然后融合结果。

CRM采用两个相同的模块分别处理光学/SAR时序数据，其首先使用两层 **一维卷积** | `Conv1D` 提取时序特征，一维卷积层公式如下：
$$
\begin{align*}

cnn_j^l = f\left( \sum_{i=1}^M cnn_i^{l-1} * k_{ij}^l + b_j^l \right)

\end{align*}
$$
其中，$k$ 表示卷积核，$j$ 表示卷积核数量，$b$ 表示偏置项（论文中加号错写为等号），$M$ 表示通道数，$f$ 表示激活函数，$*$ 表示卷积运算。

上述内容是论文原文中的解释，下面讲讲我的理解。公式中的 $l$ 是指卷积层所在层的层数，而 $j$ 表示当前卷积层的第 $j$ 个输出通道。整个公式的意思是，卷积层的输出通道 $j$ 的结果是对**所有输入通道 $i$ 做卷积**并加权求和，加上偏置后再通过激活函数。

> `Conv1D`和`Conv2D`是CNN中两种常用的卷积操作，它们的主要区别在于**输入数据的维度**和**卷积核的移动方式**。
>
> * **一维卷积用于处理序列数据**，输入形状为：`(batch_size, sequence_length, channels)`。卷积核是在“序列长度”这个维度上滑动，用于提取局部序列特征。（注意不是`1x1 Conv2D`）
> * **二维卷积用于处理图像数据**，输入形状为：`(batch_size, height, width, channels)`。卷积核用于提取局部图像特征。

CRM的卷积模块具体设置为：

1. 先用小卷积的`Conv1D`提取局部细节
2. 再接`Dropout`防止过拟合
3. 然后再用大卷积核的`Conv1D`提取全局特征
4. 最后接`MaxPooling`减少特征维度（对一维数据池化，减少时序长度🤨？）

这部分模块的设计参考了如下论文的工作，我的理解是CRM使用此模块代替传统的时序滤波操作（**论文没有提及如何<u>补全</u>去云后的S2影像数据**）。

> This operation was inspired by recent work [28], which successfully showed that the one-dimensional convolutional deep learning framework was capable of extracting effective and hierarchical features layer by layer in multi-temporal classification tasks.
>
> 此操作受近期研究启发[28]，该研究成功证明了一维卷积深度学习框架在多时相分类任务中能够逐层提取有效且分层的特征。
>
> | 属性     | 详细内容                                                     |
> | -------- | ------------------------------------------------------------ |
> | **论文** | 🔬 [*Deep Learning Based Multi-Temporal Crop Classification*](https://www.sciencedirect.com/science/article/abs/pii/S0034425718305418) |
> | **其他** | 📚︎ *Remote Sensing of Environment* \| 地球科学1区\[TOP\] \| 2019 \| 1000+ |

#### 🗒️ attLSTM模块

经过上述的时序特征提取模块，CRM已经提取了 ***健壮时间特征*** 。接着，CRM使用带有注意力机制的LSTM处理这些特征。论文中虽没明确讲解该LSTM模块是什么结构，但根据给出的CRM总览图可猜测其为 ***Seq2seq*** 的结构。适用于 *Seq2seq* 的注意力机制有 ***Bahdanau attention*** 、***Luong Attention*** 等。~~论文中中虽没给出attLSTM的结构，但引用了Luong的论文，可能使用的是使用 *Luong Attention* 的LSTM~~。下面我们来了解一下这些注意力机制。

传统的 *Seq2seq* 架构是先用`encoder`将输入序列编码称为一个`context`向量，`decoder`将`context`作为初始 **隐状态** | `hidden state` ，生成输出序列。这个过程可以参考下图：

![image-20250801142608926](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/image-20250801142608926.png)

<center>传统Seq2seq架构</center>

需要注意的是，`decoder`单元输出的隐状态既作为当前单元的输出，也作为下一个单元的输入（`encoder`、`decoder`的单元结构需保持一致）。由于`decoder`单元的隐状态和输入一致，在其他版本的架构图中，也可省略`decoder`单元的输入表示。

随着序列长度 $n$ 增加，编码器难以将所有输入学习编码为单一`context`向量，难以从输入序列中提取重要特征，这时便需要引入适用于 *Seq2seq* 架构的注意力机制来解决这个问题。

首先介绍 ***Bahdanau Attention*** 机制，这是由Bahdanau在2014提出的一种适用于 *Seq2seq* 架构的**加性**注意力机制（注意力分数为<u>加性计算</u>）。其公式如下：
$$
\begin{align*}

e_{t,i} &= \mathbf{v}^T \tanh\left(\mathbf{W}_\text{decoder} \mathbf{s}_{t-1} + \mathbf{W}_\text{encoder} \mathbf{h}_i\right) \\\\

\mathbf{c}_t &= \sum_{i=1}^{T_x} \alpha_{t,i} \mathbf{h}_i,\;\alpha_{t,i} = \frac{\exp(e_{t,i})}{\sum_{j=1}^{T_x} \exp(e_{t,j})}

\end{align*}
$$
第一个公式中各参数尺寸如下：

| 参数                        | 形状             | 说明                                 |
| --------------------------- | ---------------- | ------------------------------------ |
| $\mathbf{h}_i$              | $d_h \times 1$   | 第 $i$ 个**编码器单元**的隐状态      |
| $\mathbf{s}_{t-1}$          | $d_s \times 1$   | 第 $t-1$ 时刻**解码器单元**的隐状态  |
| $\mathbf{W}_\text{encoder}$ | $d_v \times d_h$ | 投影 $\mathbf{h}_i$ 到注意力空间     |
| $\mathbf{W}_\text{decoder}$ | $d_v \times d_s$ | 投影 $\mathbf{s}_{t-1}$ 到注意力空间 |
| $\mathbf{v}$                | $1 \times d_v$   | 向量转标量（注意力打分）             |

得到 $\mathbf{c}_t$ 后，我们可以使用 $\mathbf{c}_t$ 更新<u>前一个</u>`decoder`输出的隐状态 $\mathbf{s}_{t-1}$ 得到 $\tilde{\mathbf{s}}_t$：
$$
\begin{align*}

\tilde{\mathbf{s}}_t = \tilde{\mathbf{y}}_t = \tanh\left(\mathbf{W}_c \left[\mathbf{c}_t; \mathbf{s}_{t-1}\right]\right)

\end{align*}
$$
其中，$\tilde{\mathbf{s}}_t$ 仅作为 $t$ 时刻的`decoder`单元的输入 $\tilde{\mathbf{y}}_t$ ，$t$ 时刻的输出仍为 $f(\mathbf{s}_t)$。当我们仅使用 *Seq2seq* 架构提取序列特征时（即`decoder`单元直接输出隐状态），上述公式可直接使用。

面对不同任务时，$\tilde{\mathbf{s}}_t$ 的计算不尽相同。一般来说，当`decoder`输出的序列是文本时，`encoder`、`decoder`单元接收的是**词嵌入**（表示词语的向量），输出的是各个词的概率值。这时，`decoder`的隐状态不直接输出，而是经过类`FC`层转换再输出，具体过程可由如下公式表示：
$$
\begin{align*}

P(\mathbf{y}_{t-1}) &= \text{softmax}\left( \mathbf{W}_o\mathbf{s}_{t-1} \right) \\\\

\mathbf{y}_{t-1} &= \arg\max P(\mathbf{y}_{t-1}) \\\\

\mathbf{y}_{t-1}^{\text{embed}} &= \text{Embeding}(\mathbf{y}_{t-1}) \\\\

\tilde{\mathbf{s}}_t &= \tilde{\mathbf{y}}_t = \tanh\left(\mathbf{W}_c \left[\mathbf{c}_t; \mathbf{y}_{t-1}^{\text{embed}}\right]\right)

\end{align*}
$$
其中，$\mathbf{y}_{t-1}$ 是输出的词概率密度向量中最大概率所对应的**词**（不是词嵌入），我们需要对这个词进行嵌入操作得到对应的词嵌入。不管 *Seq2seq* 的输入输出形式如何改变，我们都需要进行模型结构微调，使得`encoder`、`decoder`单元的**I/O格式一致**<sup>原因</sup>。

![image-20250801202600868](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/image-20250801202600868.png)

<center>词嵌入输入的decoder与Bahdanau注意力机制</center>

此外，Luong也提出了 ***Luong Attention*** 机制，这时一种**乘性**注意力机制。与之前机制不同，该机制在`decoder`输出时，采用的是当前时刻的隐状态 $\mathbf{s}_t$ （这时 $\tilde{\mathbf{s}}_t$ 则作为`decoder`单元的输出）；同时，其在计算注意力分数时，使用乘运算代替加运算。

| 名称                        | 得分函数形式                                                 |
| --------------------------- | ------------------------------------------------------------ |
| **general**<sup>常用</sup>  | $e_{t,i}=\mathbf{s}_t^T\mathbf{W}_a\mathbf{h}_i$             |
| dot                         | $e_{t,i}=\mathbf{s}_t^T\mathbf{h}_i$                         |
| concat<sup>可选，加性</sup> | $e_{t,i}=\mathbf{v}_a^T\tanh\left( \mathbf{W}_a[\mathbf{s}_t;\mathbf{h}_i] \right)$ |

得到 $\tilde{\mathbf{e}}_t$ 后的注意力分数计算与上述机制相同，这里只介绍两种机制不同的地方，相关公式如下：
$$
\begin{align*}

\tilde{\mathbf{s}}_t &= \tanh\left(\mathbf{W}_c \left[\mathbf{c}_t; \mathbf{s}_{t}\right]\right) \\\\

\tilde{\mathbf{y}}_{t+1} &= \tanh\left(\mathbf{W}_c \left[\mathbf{c}_t; \tilde{\mathbf{s}}_t\right]\right) 

\end{align*}
$$
得到 $\mathbf{c}_t$ 后，我们可以使用 $\mathbf{c}_t$ 更新<u>当前</u>`decoder`输出的初始隐状态 $\mathbf{s}_{t}$ 得到 $\tilde{\mathbf{s}}_t$ ，这时的  $\tilde{\mathbf{s}}_t$ 仅用作`decoder`的输出（间接参与输入）。然后我们可以再根据输出，计算得到下一个时刻`decoder`的输入 $\tilde{\mathbf{y}}_{t+1}$ 。同样，我们在构建 *Luong Attention Seq2seq* 结构时，也需要考虑`encoder`、`decoder`单元I/O格式的一致性。
$$
\begin{align*}

\mathbf{y}_{t} &= f(\tilde{\mathbf{s}}_t) \\\\

\tilde{\mathbf{y}}_t &= \tanh\left(\mathbf{W}_c \left[\mathbf{c}_t; \mathbf{y}_{t}^{\text{embed}}\right]\right)

\end{align*}
$$

根据论文中给出的公式（$v_{ij} = a(s_{i-1}, h_j)$）不难看出，CRM的attLSTM模块采用的是 *Bahdanau Attention* 机制。

#### 👫 融合模块

根据CRM网络结构图可知，融合模块做的工作是将两个分支输出的时序特征进行一一拼接，最后再经由全连接层输出各类别概率。由论文可知，融合模块由`FC`+`ReLU`+`FC`构成，最后输出长度与类别个数相等的概率向量。需要注意的是，全连接层只接受一维向量，故CRM在将时序特征输入融合模块前，还进行了`flatten`处理。

### 4. 其他

下面选取介绍论文的余下内容。首先，论文的训练与评估细节参考如下表格：

| 内容                                                  | 详情                                                         |
| ----------------------------------------------------- | ------------------------------------------------------------ |
| <span style='white-space: nowrap'>**训练设置**</span> | 所有深度学习模型都使用**Adam优化器**和**交叉熵损失函数**进行训练，Adam配置参数与推荐值一致：$\beta_1=0.9$、$\beta_2=0.999$、$\varepsilon=1\times 10^{-7}$，且学习率固定为$0.001$。 |
| **评估指标**                                          | $\text{AC}$（ *Global* / *Local* ）、$kappa$、$\text{F1}$（$\text{macro-F1}$） |

论文为了验证CRM方法的性能与模块的有效性，进行了 *对比实验* 、*模块消融实验* 。这与一般网络论文一致，但需要注意的是，论文还对CRM**输入数据**进行了<u>对比实验</u>，这说明CRM模型的光学分支只接受**单一**类型时序数据。实验结果如下图所示：

![image-20250802141937637](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/image-20250802141937637.png)

<center>table 1.1. 对比实验、植被指数对比实验、消融实验结果（Global AC、macro-F1、kappa）</center>

论文除了对各消融/对比项进行了 *Global AC* 的对比实验，还对其进行了各类别的 *Local AC* 的对比实验，结果如下图所示：

![image-20250802142442581](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/image-20250802142442581.png)

<center>table 1.2. 对比/消融实验结果（Local AC）</center>

论文除了使用常见的方法进行模型性能评估，还使用了`t-SNE`方法可视化评估了模型分类性能，这步骤主要比较了原始数据（NDVI/SAR）和经过CRM提取后的特征（CRM最后一层隐藏层）在`t-SNE`方法下的特征空间下的个类别分布情况。这证明了CRM不需要对每一种单作作物的分布进行制图和关联，就能提取出隐藏在原始时序特征中的丰富的作物轮作信息。这段叙述原文如下：

> It further demonstrated the feasibility of extracting abundant crop rotation information hidden in the original time series without mapping and linking individual monoculture crop distribution or utilizing absolute phenological thresholds to segment different crop types [8,10].
>
> 本研究进一步验证了，无需对各类单一作物进行制图与时序关联，也无需依赖绝对物候阈值来划分作物类型，即可从原始卫星时间序列中提取出丰富的作物轮作信息的可行性。

![image-20250802144856031](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/image-20250802144856031.png)

<center>figure 1.7. 基于t-SNE统计方法的特征空间</center>



## 借物表

| 参考资料                                                     | 说明              |
| ------------------------------------------------------------ | ----------------- |
| [Bahdanau attention & Luong Attention - 十里清风](https://blog.csdn.net/sinat_34072381/article/details/106728056) | CSDN文章｜CRM参考 |
