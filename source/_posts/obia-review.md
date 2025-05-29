---
title: 综述阅读：深度学习遇见 OBIA：任务、挑战、策略与展望
date: 2025-04-15 19:02:02
tags:
  - OBIA
  - Review
categories:
  - 遥感综述
---

## 综述信息

| 属性                                               | 详细信息                                                     |
| -------------------------------------------------- | ------------------------------------------------------------ |
| **标题**                                           | 🔬 *[Deep Learning Meets OBIA: Tasks, Challenges, Strategies, and Perspectives](https://arxiv.org/abs/2408.01607)*<br>🔬 深度学习遇见 OBIA：任务、挑战、策略与展望 |
| **期刊**                                           | 📚 *Arxiv*                                                    |
| **日期**                                           | ⏲️ 2024-08                                                    |
| **作者**                                           | 👩‍🔬 Lei Ma                                                    |
| <span style='white-space:nowrap'>**关键词**</span> | `Classification`、`segmentation`、`deep learning`、`object detection`、`change detection` |

## OBIA传统任务

综述中介绍了7类遥感领域的传统任务，以及相关对象基的方法，在这里我只介绍我感兴趣的部分；

### 1. 分割

传统OBIA分割方法分为：

#### 1.1. 基于边缘分割

基于边缘的分割逻辑是识别并封闭（原文为：... identify and close edges ...）边缘来描绘对象，基于边缘分割的步骤是：1）边界检测；2）边缘闭合；

**边缘检测**的过程可以分为三个步骤：滤波 → 增强 → 检测；这些识别边缘的算法基本上都是通过局部梯度算子识别边缘；相关方法有：

| **方法名称**  | **作者**   | **年份** | **介绍**                                                     |
| ------------- | ---------- | -------- | ------------------------------------------------------------ |
| **Sobel**     | I. Sobel   | 1968     | 基于**一阶导数**的经典边缘检测算子；<br/>用梯度计算捕捉像素值剧烈变化； |
| **Canny**     | J. Canny   | 1986     | 高精度低噪声边缘检测算子；                                   |
| **Watershed** | L. Vincent | 1991     | 模拟水流的图像分割算法；                                     |

需要注意<u>边缘检测</u>得到的边缘常常不完整/闭合，需要进行**边缘闭合**得到完整对象；需要注意的是综述中提到了还提到了[邻域搜索算法](https://scholar.google.com/scholar?cluster=17111321705906172858&hl=zh-CN&as_sdt=0,5)可以用于边缘闭合（我在网上没有看到相关对应文章的解析）；相关方法如下：

| **方法名称**                      | **作者**   | **年份** | **介绍**                                         |
| --------------------------------- | ---------- | -------- | ------------------------------------------------ |
| **霍夫变换** \| `Hough transform` | N. Kiryati | 1991     | 提取几何形状；图像越复杂，所需霍夫空间维数越高； |

#### 1.2. 基于区域分割

基于区域的分割算法是先确定区域，再检测边缘；基于区域的分割算法分为：<u>区域增长</u>、<u>区域分割</u>两种模式；

区域增长方法通常依赖于种子选择和相似性标准来确定区域的增长和合并；相关算法有：

| **方法名称**                      | **作者**   | **年份** |
| ---- | ---- | ---- |
| **多分辨率分割** \| `MRS` | G. J. Hay | 2003 |
| **均值漂移分割** \| `MS` | D. Comaniciu | 2002 |
| **分形网络进化方法** \| `FNEA` | S. M. De Jong | 2007 |

区域分割算法则是迭代的将图像分割成更小的子区域，直到子区域均匀；

### 2. 分类

OBIA发展历史：
1. 基于专家知识设置阈值（问题：对具有明确边界和确定性特征的数据是有效的，现实世界数据中常见的**不确定性**和**模糊性**时，它们的效用有限）；

2. Mathieu等人[41]和Jacquin 等人[48]引入了可以处理分割类别中固有不确定性的**模糊规则分类方法**🤔；
    | **论文标题** | **论文标题**                                                 | **年份**         |
    | ------------ | ------------------------------------------------------------ | ---------------- |
    | **41**       | 🔬 *Mapping private gardens in urban areas using object-oriented techniques and very high-resolution satellite imagery* | R. Mathieu, 2007 |
    | **48**       | 🔬 *A hybrid object-based classification approach for mapping urban sprawl in periurban environment* | A. Jacquin, 2008 |

3. 人们开始转向决策树 | `Decision tree` 、SVM、RF等机器学习方法；

### 3. 时序分析⭐️

**卫星图像时间序列** | `SITS` 已成为研究地球表面动态变化的无价资源，提供了跟踪陆地特征随时间变化的增强能力；

SITS因为星载传感器的分辨率提高，有研究从PBIA转向OBIA，SITS分析方法有：1）相似度测量方法（DTW）；2）基于统计假设与模型的方法🤔；3）机器学习方法[略]；

#### 3.1. 相似度测量

这部分提到就是我之前看DTW+SITS相关论文，该领域的大多数研究都集中在DTW上，但He等人建议进一步探索其他方法，如基于<span style="color: cyan">**形状距离**</span> | `SBD` 和 **全局对齐核** | `GAK` ，并且发现GAK测量通常比其他方法更适合<u>农业制图</u>。相关论文如下：

| **序号** | **论文标题**                                                 | **年份**        |
| -------- | ------------------------------------------------------------ | --------------- |
| **96**   | 🔬 *Evaluation of advanced time series  similarity measures for object-based cropland mapping* | W. He, **2023** |

#### 3.2. 基于统计假设与模型

> 基于统计假设、模型的方法通常旨在通过<u>**建模**</u>密集时间序列来实现<u>变化检测</u>，这有效地缓解了噪声和光照变化等问题；

基于像素的这类模型有： `LandTrendr`、`BFAST`、 `CCDC`，但一些轻微扰动会影像模型性能；为了解决这个问题，⼀些研究人员转向了**分段对象分析**🤔，利用对象的光谱、形态和纹理特征来提高SITS分析结果。相关论文为：

| **序列** | **论文标题**                                                 | **作者&年份**   |
| -------- | ------------------------------------------------------------ | --------------- |
| **93**   | 🔬 *Object-based continuous monitoring of land disturbances from dense Landsat time series* | S. Ye, **2023** |

### 4. 特征提取 

特征提取通常在<u>图像分割之后</u>进行，经典的对象特征包括光谱、几何、纹理和上下文信息，**深度学习模型已被用于 OBIA 中的特征提取**，即将训练好的神经网络看作特征提取器；
| **序号** | **论文标题**                                                 | **作者&年份**        |
| -------- | ------------------------------------------------------------ | -------------------- |
| **118**  | 🔬 *Spectral‒spatial feature extraction for hyperspectral image classification: A dimension reduction and deep learning approach* | W. Zhao, <u>2016</u> |

## 深度学习+OBIA的进展与挑战

综述中，这章节开始介绍了OBIA中流行的深度学习模型（这里提及的各篇论文的标题中均有`object-based`关键词，**都是基于OBIA范式**），主要有：

| **深度学习模型** | **变体模型**           | **应用任务**                                        | **序号**  |
| ---------------- | ---------------------- | --------------------------------------------------- | --------- |
| **CNN**          | R-CNN、FCN、Mask R-CNN | 通过卷积层提取局部空间特征、<br/>适用于分类与分割； | [130-137] |
| **RNN**          | LSTM                   | 用于对象级时间序列分析                              | [141-145] |
| **GNN**          | GCN                    | 利用邻接关系建模空间上下文                          | [147-150] |

### 1. OBIA本质解析

当前研究存在对OBIA本质的误解，需澄清以下关键点：

1. **OBIA应当被视为范式而非算法**：
    * OBIA是分析框架，包含分割、特征提取、分类等步骤；而深度学习则是这几个步骤中可选择的工具；
    * 需要注意的是，普通分类模型用到OBIA中被看成是某一步骤的具体实现，而带有标签输出的语义分割模型则可视为OBIA的一种实现形式。
2. OBIA不局限于遥感影像，也可整合OSM数据；

### 2. 当前挑战

OBIA与深度学习结合过程中存在如下问题：

* <del>**分析单元不一致**</del>：当前使用OBIA解析的SITS分析常使用时间尺度上一致对象解析基于对象的时间序列分析，但这忽略了对象在时间尺度上的实际变化；
* **训练样本不足**：OBIA压缩了数据量与计算量，但是深度学习需要大样本，因此，对于基于深度学习的OBIA而言， 样本增强方法似乎是必要的；
* **解释性差**：OBIA倾向于使用基于规则的分类方法对分割对象进行分类，这提供了强大的可解释性；而深度学习本身存在可解释性差的问题；
* **语义分割模型**：语义分割仍然面临着平衡分类性能与**边界描绘**（由于下采样、上采用结构）的主流困境🤔。

## 深度学习应用于OBIA的现有研究

### 1. OBIA与深度学习分类块

OBIA分类任务中最困难的部分是将这些<u>不规则地理物体</u>转换为固定大小的CNN输入 **块** | `patch` 。当前研究有两种策略进行patch提取：1）识别物体到的边界框并重采样到固定大小；2）根据候选点提取相应的patch；相关论文如下：

| **序号** | **论文标题**                                                 | **作者&年份**              | **总结**                                            |
| -------- | ------------------------------------------------------------ | -------------------------- | --------------------------------------------------- |
| **90**   | *Novel shape indices for vector landscape pattern analysis*  | C. Zhang， <u>2016</u>     | 可以通过计算对象**惯性矩**确定一个patch框的主方向； |
| **163**  | *Tansferable object-based framework based on deep convolutional neural networks for building extraction* | R. D. Majd, <u>2019</u>    | 分析目标对象的周围信息同质性，更改patch框的大小；   |
| **11**   | *Using convolutional neural  network to identify irregular segmentation objects from very  high-resolution remote sensing imagery* | T. Fu, <u>2018</u>         | 直接用对象中心的patch代表对象；                     |
| **168**  | *Improved  object-based convolutional neural network (IOCNN) to classify very high-resolution remote sensing images* | X. Lv, <u>2021</u>         | 使用 `a moment bounding box` 来找寻候选点🤔；        |
| **12**   | *Exploring multiscale object-based convolutional neural network  (multi-OCNN) for remote sensing image classification at high spatial  resolution* | V. S. Martins, <u>2020</u> | 使用 **骨架化算法** + **二叉树采样** 找寻候选点🤔；  |

不同大小的patch没有必要重采样到一个大小并投入网络，在论文[168]中，论文作者将对象分为<u>一般对象</u>与<u>线性对象</u>，并将从这两种对象中提取的patch分别投入不同的分类网络；

![image-20250416103455899](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/image-20250416103455899-1744770896046.png)

<center>figure 3. 多patch尺寸网络分类分割对象</center>

### 2. OBIA与深度学习提取特征

深度学习生成的栅格数据（<u>语义分割模型</u>生成的概率热力图）可以直接作为特征为OBIA分类过程做出贡献；此外，综述还提及了语义分割模型直接识别土地覆盖相关研究，在这里就不过多提及，相关论文如下：

| **序号** | **标题**                                                     | **作者&年份**                 | **总结**                                                     |
| -------- | ------------------------------------------------------------ | ----------------------------- | ------------------------------------------------------------ |
| **127**  | *Landslide detection using deep learning and  object-based image analysis* | O. Ghorbanzadeh, <u>2022</u>  | 训练一个检测滑坡概率的模型，该模型结果参与**OBIA分割、分类**； |
| **108**  | *Automated detection of rock glaciers using deep learning and  object-based image analysis* | B. A. Robson, <u>2020</u>     | CNN+高斯滤波生成冰川<u>概率热力图</u>；                      |
| **37**   | *Transferable instance segmentation of dwellings in a refugee  camp-integrating CNN and OBIA* | O. Ghorbanzadeh， <u>2021</u> | 使用CNN得到的概率，对MRS分割进行进行<u>合并、分类分割对象</u>； |

![image-20250416104454327](https://soppy-ie-1351762962.cos.ap-chongqing.myqcloud.com/soppy-ie/image-20250416104454327-1744771494409.png)

<center>figure 4. 语义分割模型输出作为OBIA分割、分类输入</center>

### 3. 使用OBIA细化深度学习像素分类结果

OBIA还可以对深度学习分类结果进行**基于对象的后分类细化**（Oject-Based Post-Classification Refinement，OBPR），该方法可以被视为一种滤波方法；传统的OBPR方法通常是使用 <u>多数投票法</u> 对基于像素的分类器结果进行滤波，目前也存在使用 **条件随机场** | `CRF` 将基于像素的分类概率于对象单元相结合[191]；相关论文如下：

| **序号** | **标题**                                                     | **作者&年份**      | **总结**                                                     |
| -------- | ------------------------------------------------------------ | ------------------ | ------------------------------------------------------------ |
| **188**  | *Fully convolutional networks and geographic object-based image  analysis for the classification of VHR imagery* | N.Mboga, 2019      | 使用全卷积网络进行像素分类，如何使用<u>多数投票法</u>进行对象分类； |
| **191**  | *Mapping large-scale and  fine-grained urban functional zones from VHR images using a  multi-scale semantic segmentation network and object based approach* | S. Du, <u>2021</u> | 使用<u>条件随机场</u>将像素级分类概率与对象单元结合；        |
| **130**  | *Object-based convolutional neural  network for high-resolution imagery classification* | W. Zhao, 2017      | 将深度学习生成特征与像素级对象特征结合，训练两层MLP以确定对象标签； |
| **192**  | *Joint Deep Learning for land cover and land use  classification* | C. Zhang, 2019     | 使用MLP与CNN构建模型，然后使用**马尔可夫过程**进行分类的迭代更新； |

### 4. 基于图的OBIA深度学习

传统OBIA在分析对象时，不考虑相邻对象提供的上下文。为了解决这一局限，可以引入基于图的方法来进行OBIA。在构建图时，每个对象代表一个点，相邻对象间可以构建**边**连接；在<u>基于对象的时间序列分析</u>中，如果对象在空间上重叠，**边**也可以连接连续时间步的对象；相关论文如下：

| **序号** | **标题**                                                     | **作者&年份**  | **总结**                                                     |
| -------- | ------------------------------------------------------------ | -------------- | ------------------------------------------------------------ |
| **193**  | *A fully learnable context-driven object-based model for mapping  land cover using multi-view data from unmanned aircraft systems* | T. Liu, 2018   | 该方法为每个对象构建一个图，并使用兼容性矩阵与加权图分析作为OBIA后续步骤； |
| **201**  | *CNN-enhanced  heterogeneous graph convolutional network: Inferring land use from land  cover with a case study of park segmentation* | Z.-Q, **2022** | 将图提供的**上下文**信息与CNN提取的**特征**结合，可以比一些语义分割模型实现更高的精度； |

综述提及了一些用于图处理的方法：
* **图卷积网络** | `GCN` [195]；
* **图注意力网络** | `GAT` [197]；
* `GraphSAGE` [196]；
* `ResGatedGCN` [198]；
* `GraphTransformer` [199]；
