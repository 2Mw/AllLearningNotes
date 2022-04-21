# notes

[CIKM 2020 | 一文详解美团6篇精选论文](https://tech.meituan.com/2020/12/03/cikm-2020-nlp.html)

推荐系统的专门会议：ACM Conference on Recommender Systems ([RecSys](https://dblp.uni-trier.de/db/conf/recsys/index.html))，但是CCF未列入。

[“知识图谱+”系列：知识图谱+图神经网络](https://zhuanlan.zhihu.com/p/358119044)

[TOC]

## 前言

建议做笔记需要提及以下几点：

* 提出了什么网络
* 这个网络的所属类别
* 解决了之前研究者什么难题
* 有什么优点，有什么启发，有什么新颖指出
* 在什么数据集上取得了什么样的效果
* 对于需要精读研究的论文需要做完全的笔记（标题带⭐表示复现与精读）

## 论文 Reading

### —2022-03

#### DKN⭐

![image-20220325165026465](PaperReading.assets/image-20220325165026465.png)

原文：WANG H, ZHANG F, XIE X, et al. DKN: Deep knowledge-aware network for news recommendation[C]//Proceedings of the 2018 world wide web conference.2018:1835-1844. 

简介：DKN 算法是由 Wang Hongwei 提出的用于新闻推荐的深度知识网络，借助 KGE(knowledge Graph Embedding) 将图结构信息新闻标题和标题中的实体信息转为空间向量，使用 KCNN 将对应的向量转为 news embedding 和 User embedding 来方便数据的训练；文章还借用了注意力机制来对于候选新闻动态提取用户的点击历史。

关键词：推荐算法，知识图谱，新闻推荐，注意力机制，CTR 预测

解决的问题：

1. 现存的方法都不能够完全发现新闻之间的知识级别(knowledge-level)的联系。（需要使用知识图谱）
2. 对于新闻推荐存在三个问题：时效性，如何从点击历史中提取用户兴趣，不能挖掘知识与知识之间的联系。协同过滤的算法在新闻推荐方面较为低效。

数据集：

1. MIND 微软新闻数据集

我的评价：

1. 梦开始的地方，第一个复现（未复现完）的算法。
1. 缺点就是部分还需要特征工程。
2. 从这篇文章开始，了解到 embedding 的方方面面，以及当前将知识图谱转为 Embedding 的几种方法。
3. 还接触到了注意力机制，以及怎么用，虽然不知道具体的用途。
4. 文章中还存在几处我觉得在 2022 年可以进行改进的地方：
   * 在进行句子表示学习的地方，作者使用的是 CNN 的方法来进行学习。现在 NLP 优秀的算法层出不穷，可以使用 Transformer 或者 Bert 等模型来进行改进
   * 在进行 Attention 的地方说不定可以加几个 Mask 试一试
5. 未来发论文的其他方向：
   * 模型的可解释性，即类似美团外卖中推荐某个东西显示推荐原因。
   * 模型的轻量化，借助最近看的知识蒸馏的方法实现工程上的优化。

论文精读：

​		现存的推荐方法不能完全挖掘新闻之间知识级别的潜在联系，并且推荐结果也只限制于简单的模式而不能够合理扩展。DKN 是基于内容的点击率预测深度学习推荐算法，DKN 的关键组件就是 KCNN，KCNN 能够输出多个通道并且有 word-entity-aligned 卷积神经网络将新闻的语义级别(semantic-level)和知识级别(knowledge-level)的表示方法融合在一起。作者为了能够解决用户有不同兴趣的问题，使用了注意力模块来根据候选新闻来动态从用户点击历史中聚合对应的特征。DKN 算法在微软的新闻数据集 MIND 上取得了 F1 和 AUC 基准上的提高。

​		新闻标题是文本信息，需要将其转为向量空间中方便计算。首先将新闻标题转为 word embedding，然后中提取出新闻中的 Entity ，根据对应 wikiID 在维基百科数据库[^1][^2]中的找到对应的知识图谱，使用知识图谱嵌入 (Knowledge Graph Embedding) 的方法将 entity 词条中的信息转换到向量空间中。常见 KGE 方法有 TransE，TransH, TransR, TransD 方法转为向量，文章中将其称为 entity embedding，作者是在前人的基础上完成转化的。由于只有 word embedding 和 entity embedding 并不能充分体现对应的 entity 在知识图谱中的位置和其周围节点的信息，作者还引入了 context embedding 的概念，即根据 entity 节点周围节点之间的 entity embedding 的加权平均来计算。

#### RippleNet

![image-20220327194225149](PaperReading.assets/image-20220327194225149.png)

原文：WANG H, ZHANG F, WANG J, et al. Ripplenet: Propagating user preferences on the knowledge graph for recommender systems[C]//Proceedings of the 27th ACM international conference on information and knowledge management.2018:417-426. 

简介：受到现实生活中水中波纹传播的启发，Wang Hongwei 提出 RippleNet 推荐算法。其使用知识图谱作为边界信息(side information)，结合了基于知识图谱推荐算法两种途径 embedding-based 和 path-based 的优点，将知识图谱中节点之间的相关度计算类似水纹传播一样进行计算，根据上一个 Ripple Set 来计算下一个 Ripple Set 之间的相关度，最终预测用户对应某个 item 的点击率预测。

关键词：推荐系统，知识图谱，CTR 预测，波纹网络

解决的问题：

1. 解决了传统协同过滤算法中存在的数据稀疏性和冷启动问题
2. 当前的 embedding-based 的推荐算法不够直观和高效，path-based 的算法需要依靠手工来刻画特征较为繁琐。RippleNet 结合了两者的优势并且一定程度上解决了两者的问题。
3. RippleNet 在解释性方面有着很好的特点。

数据集：

1. MovieLens-1M
2. Book-Crossing
3. Bing-News

我的评价：

1. 推荐算法领域基于知识图谱的较为经典的论文，值得复现。
2. 相比于 DKN 强化了知识图谱处理的部分。

论文精读（未做）：

#### MKR

原文：WANG H, ZHANG F, ZHAO M, et al. Multi-task feature learning for knowledge graph enhanced recommendation[C]//The world wide web conference.2019:2000-2010. 

![image-20220328084111804](PaperReading.assets/image-20220328084111804.png)

简介：MKR 算法是由 Wang Hongwei 提出的基于知识图谱的多任务特征学习的推荐算法，他提出了新的 Cross & Compress 单元将两个任务进行关联，用于建立 item 和 entity 之间的关系，可以从中自动共享潜在的信息特征，并且能够学习 item 之间的高阶联系。作者自己还提出了一种新的 KGE 方法，最终结果也再多个数据集上取得了 SOTA 的结果。

关键词：推荐系统；知识图谱；CTR 预测；Top-K 预测；多任务学习；公式证明

解决的问题：

1. 经典 CF 问题，稀疏性，冷启动
2. PER 严重依赖于手工刻画 meta-paths / meta-graph 的问题
3. DKN 不是一种端对端的训练方式，并且很难将 side information 融入到文本中
4. RippleNet 中存在对于边信息的刻画不足。
5. CKE 方法中 KGE 的方法 (TransR) 在推荐系统的应用中效果不是那么好。

数据集：

* MovieLens-1M
* Book-Crossing
* Last.FM
* Bing-News

我的评价：

1. 作者横向对比了多个基于知识图谱的推荐算法之间的优缺点
2. 作者对于 Cross&Compress 单元有着严格理论证明 (多项式逼近)
3. 文章中使用到了 Negative Sampling (负采样)，还有其他文章提到了 Label Smoothing (标签平滑)，填坑



#### KGNN-LS

![image-20220328083718596](PaperReading.assets/image-20220328083718596.png)

原文：WANG H, ZHANG F, ZHANG M, et al. Knowledge-aware graph neural networks with label smoothness regularization for recommender systems[C]//Proceedings of the 25th ACM SIGKDD international conference on knowledge discovery & data mining.2019:968-977. 

简介：KGNN-LS 算法是由 Wang Hongwei 在其 KGNN for RS 的基础上进一步改进，是一种混合推荐方法，结合了 Path-based 和 Embedding-based 两种方法的优点。KGNN-LS 不仅能够捕捉 item 之间的语义信息，还能够提供个性化推荐。

关键词：推荐系统；图神经网络；知识图谱；标签平滑；CTR 预测

解决的问题：

1. 传统 CF 问题
2. 现存方法存在的依赖于手工特征工程，不能够端到端处理以及没有很好的伸缩性的问题
3. 基于 KG 的方法大都采用同构二部图(homogeneous bipartie graph) 或者 uesr/item-similarity graph。
4. 他之前的 KGNN for RS 存在过拟合的问题，这次他使用标签平滑正则化来解决。

数据集：

1. MovieLen-20M
2. Book-Crossing
3. Last.FM
4. Dianping-Food

我的评价：

1. 需要 GNN 的前置知识，理论部分看不懂
2. 有着严格的数学证明在 GNN 中如何让进行 Label Smoothness Regularization，并且证明 LS 相当于在图结构上的 Label Propagation



#### KGAT

![image-20220413162742252](PaperReading.assets/image-20220413162742252.png)

原文：WANG X, HE X, CAO Y, et al. Kgat: Knowledge graph attention network for recommendation[C]//Proceedings of the 25th ACM SIGKDD international conference on knowledge discovery & data mining.2019:950-958. 

简介：KGAT 算法是使用注意力机制基于知识图谱的端对端推荐算法，其不仅能够挖掘图谱中的低阶信息，还能够挖掘数据中的高阶隐晦的特征信息。其通过从邻居节点迭代传播 embedding 从而提炼中节点的 embedding，并且能够区分邻居节点的重要程度。

关键词：

解决的问题：

1. FM 的方法不能完全从用户一系列行为中充分挖掘出用户之间协同信号。
2. 先前的基于知识图谱的的方法存在无法充分捕捉数据之间高阶联系的问题。

数据集：

1. Amazon-book
2. Last-FM
3. Yelp2018

我的评价：





#### TGIN

![image-20220325164410214](PaperReading.assets/image-20220325164410214.png)

原文：Jiang W, Jiao Y, Wang Q, et al. Triangle Graph Interest Network for Click-through Rate Prediction[C]//Proceedings of the Fifteenth ACM International Conference on Web Search and Data Mining. 2022: 401-409.

简介：TGIN 是由阿里巴巴团队提出的用于推荐系统关于三角图兴趣网络的点击率预测的算法。在 item-item 图中抽取出很多“三角内同构，三角外异构”的三角形，一个三角形内的 item 至少拥有一个共同的属性，然后通过 item 周围的 K-Order 三角形来提取出特征，来得到最终用户点击率预测结果。

关键词：推荐算法，CTR 预测，商品推荐，图神经网络，三角图

解决的问题：

1. 像 YoutubeNet，DeepFM，DIN 等算法只是基于用户的历史数据来总结用户的兴趣特征，但是存在数据稀疏性的问题，并且不能跳脱出用户特定的兴趣特征，不能很好的探索新的感兴趣的领域（信息茧房）
2. 像 RippleNet，KGAT，ATBRG，MTBRN 等算法，虽然结合了图结构，但是其都缺乏一种有效的机制来捕捉用户一些难以捉摸(elusive)的兴趣，更倾向于向用户推荐时髦的、较为相似的的产品。
3. 以上问题即用户难以捉摸的动机(elusive motivation)和推荐多样性限制(diversity)的问题。

数据集：

1. 亚马逊的书籍和电子产品商品的数据集，取得了 state-of-the-art 的成果
2. 阿里巴巴在线广告系统数据集，他家的数据集他说了算。

我的评价：

1. 只看图就知道算法模型对于我来说有点过于复杂了，比如前期的进行 item 之间三角形选取的时候，使用到**行列式点过程**(Determinantal Point Process) 来生成 item 对应的 K-Order 图结构。
2. 对于之前模型存在的无法解决推荐多样性的问题，可以直接在图结构上使用三角图的思想来优化推荐系统的模型。

### 2022-04

#### Wide & Deep

![image-20220404103741975](PaperReading.assets/image-20220404103741975.png)

参考：[推荐系统之ctr预估-Wide＆Deep模型解析](https://zhuanlan.zhihu.com/p/74957209)

原文：CHENG H-T, KOC L, HARMSEN J, et al. Wide & deep learning for recommender systems[C]//Proceedings of the 1st workshop on deep learning for recommender systems.2016:7-10. 

简介：Wide & Deep 是应用在 Google Play 应用商店的推荐算法模型，是经过百万人的考验的算法。模型采用了 Wide 的线性模型和 Deep 的深度神经网络模型，同时增强了模型的拟合能力和泛化能力。文中通过 Memorization 和 Generalization 描述了模型的拟合和泛化能力，前者推荐结果较为保守，推荐用户之前有过联系的 item，后者推荐结果趋于多样性。对于 wide 和 deep 两个子模型，文中采用 joint training 的方式将两个模型结合起来。

关键词：推荐算法；CTR 预测；

解决的问题：

1. wide 的模型需要做更多特征工程的处理。
2. deep 的模型在学习稀疏矩阵得到的 embedding 不够高效

数据集：

1. Google Play 自己的数据

我的评价：

1. 文中结合了 wide 和 deep 两种模型的有点，使用 joint training 的方法将两种模型结合起来，吸取了两个模型的优点，同时兼顾了拟合能力和泛化能力。
2. 此模型既不同于 CF-Based 的也不同于 CB-Based 方法，较为独特，缺点就是 wide 部分需要进行特征工程。

#### DeepFM⭐

![image-20220404112821459](PaperReading.assets/image-20220404112821459.png)

原文：GUO H, TANG R, YE Y, et al. DeepFM: a factorization-machine based neural network for CTR prediction[C]//Proceedings of the 26th International Joint Conference on Artificial Intelligence.2017:1725-1731. 

简介：DeepFM 是应用在华为应用商店的推荐算法模型。DeepFM 是在 Wide & Deep 上改进而来的算法，是一个端对端的模型，该模型相比于之前的算法模型，能够同时对高阶信息和低阶信息建模，并且无需预训练和进行特征工程。

关键词：推荐算法；CTR 预测；FM 分解机；

解决的问题：

1. 解决之前模型无法同时建模高阶信息和低阶信息的问题。线性模型无法泛化到高阶特征联系中，FM 只能建模高阶信息。
2. Wide & Deep 模型需要手工特征工程的问题，DeepFM无需进行特征工程，并且在 wide 和 deep 部分能够将输入进行共享，无需区分连续型数据和离散型数据。

数据集：

1. Criteo 数据集
2. 华为应用商店数据集

我的评价：

1. DeepFM 在 Wide & Deep 模型的基础上进行了改进，无需进行手工特征工程。
2. DeepFM 可以算是较为经典 FM 的算法，模型看上去也没那么复杂
3. 文中提到神经网络模型 CNN，RNN 都是应用在网格或者序列型的数据上，对于推荐系统上都不尽如人意，可以尝试结合 GNN。

论文精读（待填坑）：

​	

参考读物：

1. [Factorization Machines (d2l.ai)](http://d2l.ai/chapter_recommender-systems/fm.html)
2. [初探Factorization Machine(I). 揭開推薦系統中的明星演算法的神秘面紗](https://yulongtsai.medium.com/factorization-machine-63160bc2c06b)

#### xDeepFM

![image-20220405171252335](PaperReading.assets/image-20220405171252335.png)

原文：LIAN J, ZHOU X, ZHANG F, et al. xdeepfm: Combining explicit and implicit feature interactions for recommender systems[C]//Proceedings of the 24th ACM SIGKDD international conference on knowledge discovery & data mining.2018:1754-1763. 

简介：xDeepFM 是基于 DeepFM 推荐算法的基础上改进而来，其不仅能够显式构建特征，还能够隐式学习任意高阶和低阶的特征信息。该模型能够在有界的特征联系上高效学习，并且学习的级别是基于 embedding 向量，经过 CIN 网络、线性网络模型和朴素多层感知机得到最终的预测结果。

关键词：推荐系统；CTR 预测；FM 分解机；

解决的问题：

1. 传统的 cross feature 方法获取优质的特征信息耗费时间和精力较多。
2. 在大型 web 网络上的数据是较为稀疏，很难手工提取出所有的特征；并且手工提取的特征泛化能力较弱。
3. FNN 和 PNN 等模型大都关注建模高阶信息而忽略了低阶信息。
4. 相比于 Wide & Deep 和 DeepFM 的 bit-wise 模型，xDeepFM 模型使用 vector-wise 的方法，属性内的 embedding 向量不会互相影响。

数据集：

1. Criteo 数据集
2. DianPing 数据集
3. Bing News 数据集

我的评价：

1. xDeepFM 基于 Deep & Cross (DCN)模型上进行了改进提出了 CIN 网络模型
2. 也是基于 Deep & Wide 和 DeepFM 模型进行改进，使用 vector-wise 取代 bit-wise 性能更优。

#### Factorization Machines⭐

![image-20220407175147534](PaperReading.assets/image-20220407175147534.png)

原文：RENDLE S. Factorization machines[C]//2010 IEEE International conference on data mining.IEEE,2010:995-1000. 

简介：Factorization Machines 是使用分解参数的预测方法，可以在十分稀疏的数据上完成预测的模型。FM 模型能以线性时间复杂度的情况下计算得出结果。FM 模型也是经典的算法，后续的很多算法模型比如 Wide & Deep ，DeepFM，xDeepFM都是基于 FM 模型而来。

关键词：推荐算法；FM 模型；CTR 预测；

解决的问题：

1. SVM 在十分稀疏的数据的情况下会导致预测失败，FM 可以解决稀疏的问题。
2. 作者将 FM 模型的计算时间复杂度从 $O(kn^2)$ 降到 $O(kn)$，极大提高了计算效率。

数据集：

1. 无

我的评价：

1. 十分经典的奠基型论文

#### DCN⭐

![image-20220411114831845](PaperReading.assets/image-20220411114831845.png)

原文：WANG R, FU B, FU G, et al. Deep & cross network for ad click predictions[C]//Proceedings of the ADKDD'17.2017:1-7. 

简介：DCN 推荐算法模型全称 Deep & Cross Network，作者提出了新的 CrossNet 来捕捉各个特征之间的交叉特征(Cross Features)，能够在指定范围的n阶特征上进行高效的计算。并且结合了 DNN 来捕捉更加隐性的高阶特征。该推荐算法模型无需先前工作中大量耗时费力的特征工程，并且计算时间复杂度也只是随输入数据的长度线性增加，计算更加高效。

关键词：推荐算法；CTR 预测；Deep & Cross

解决的问题：

1. 无需做大量耗时费力的特征工程，计算更高效。
2. 使用 CrossNet 和 DNN 来捕捉低阶和高阶的特征信息，以及捕捉不易发现的隐性特征。
3. FM 或者 FFM 模型在扩展到高阶特征的时候，参数数量会随着输入量的增加而成倍的增加，计算复杂度高。

数据集：

1. criteo 数据集

我的评价：

1. 相比于之前的 wide & deep 模型，deep 部分几乎没有什么差别，主要区别是 wide 部分和 cross 部分。 wide & deep 是直接将输入数据进行点积就进行计算输出了，cross 部分是将各个特征之间进行点积并且和一个标量权重进行相乘和 Deep 部分的网络参数一起进行训练优化。
2. 与 DCN 较为相似的是 DeepFM 模型，deep 部分同样类似。两者之间的主要区别就是对于特定阶特征的计算部分，对于 DeepFM 比如对于二阶特征 i 和 j 的权重 $w_{ij}$ 分解为两个权重向量之间的点积 $<v_i,v_j>$，而 DCN 部分两交叉项(cross term)之间相乘的权重是对应一个矩阵中的标量，并且可以参数共享，相比与FM部分计算更高效。

#### DCN v2⭐

![image-20220411122952753](PaperReading.assets/image-20220411122952753.png)

原文：Wang R, Shivanna R, Cheng D, et al. DCN V2: Improved deep & cross network and practical lessons for web-scale learning to rank systems[C]//Proceedings of the Web Conference 2021. 2021: 1785-1797.

简介：DCN v2 是在 DCN 的基础上进行改进而来的推荐算法模型，在原有模型的基础上提高了计算效率，并且相比于之前的模型，DCN v2 成功应用在谷歌的实时 rank 排名系统上。并且 DCN v2 引入了矩阵低秩分解的方法来高效计算算法，能够在保证准确度的基础上提高模型的计算速度。文章作者也使用实验数据证明传统的基于 ReLU 的神经网络在学习高阶特征交叉的时候效率更低。

关键词：推荐算法；Rank 算法；DCN；实时系统；

解决的问题：

1. 初代 DCN 算法在大规模网页流量的基础上无法进行实时运算，计算效率较低。
2. 初代 DCN 在计算输入数据的特征特定的维度处理，不支持任意长度的输入。
3. 该模型不仅在离线模型上取得了 SOTA 的成果，也在在线模型上成功进行实验，能够满足在线推荐的实时性要求。

数据集：

1. criteo 数据集
2. movielens 数据集

我的评价：

1. 很多文章中都使用到了多项式逼近(polynomial approximation)的思想，可以学习一下
2. DCN 和 DCN v2 都是同一个作者，DCN v2 在原先的基础上改进了 CrossNet 的部分并且加入了矩阵低秩分解的点，将 DCN 网络推向了新的高峰。

#### FFM

![image-20220419131344991](PaperReading.assets/image-20220419131344991.png)

原文：Juan Y, Zhuang Y, Chin W S, et al. Field-aware factorization machines for CTR prediction[C]//Proceedings of the 10th ACM conference on recommender systems. 2016: 43-50.

简介：在 FM 因子分解机的基础上添加了域(field)的概念，

关键词：推荐算法；CTR 预测；FM 因子分解机；

解决的问题：

1. FM 中特征交叉的方式没有考虑到不同特征之间的共性（同域）和差异性（异域）的问题。
1. FM 中的特征交叉不够细粒度。

数据集：

1. criteo 数据集
2. avazu 数据集

我的评价：

1. FFM 相比于 FM 模型精确度更高，特征刻画更加精细
2. 但是 FFM 模型时间复杂度更高，并且参数较多，必须设置正则化和早停训练策略。
3. Bi-FFM 相比之下参数更少。

参考：

1. [深入FFM原理与实践](https://tech.meituan.com/2016/03/03/deep-understanding-of-ffm-principles-and-practices.html)
2. [FM及FFM算法](https://codeantenna.com/a/ZFbApvJVQH)
3. [推荐系统系列（二）：FFM 算法理论与实践](https://www.6aiq.com/article/1590363925275)

#### Autoint

![image-20220421093927071](PaperReading.assets/image-20220421093927071.png)

原文：Song W, Shi C, Xiao Z, et al. Autoint: Automatic feature interaction learning via self-attentive neural networks[C]//Proceedings of the 28th ACM International Conference on Information and Knowledge Management. 2019: 1161-1170.

简介：AutoInt 算法是基于注意力机制的端到端推荐算法模型，可以用于自动显式建模特征之间的高阶和低阶的关联，并且该算法模型相比于之前的模型有着较好的可解释性。作者将注意力机制引入推荐算法领域，根据多头自注意力机制提出了新的 Interacting Layer，既能够将高维稀疏的输入数据映射到低维子空间中，也能够将各个不同的特征融合起来构成高阶特征信息共同训练，并且无需大量手工费时的特征工程。

关键词：推荐系统；CTR 预测；注意力机制；可解释模型

解决的问题：

1. 对于推荐系统的原始数据存在数据过于稀疏并且维度高的问题，大多数据是离散型或者分类型(categorical)的数据，不易处理和建模，容易导致过拟合。
2. 对于高阶信息，大多 DNN 的算法模型(Wide & Deep, DeepFM)都是隐式建模高阶，可解释性较差
3. 显式建模高阶特征信息的模型(DCN, xDeepFM)不能够很好解释哪些交叉特征是有用的。
4. 作者使用多头注意力机制既能显式建模低阶和高阶信息，并且能够将高维稀疏的向量映射到低维子空间中。

数据集：

1. Criteo 数据集
2. avazu 数据集
3. KDD 12 数据集
4. MovieLens-1M 数据集

我的评价：

1. 所见的基于协同过滤算法中使用注意力机制来替代全连接层深度神经网络来进行建模和训练。
2. 这个模型也是基于协同过滤中具有可解释性特点的模型
3. 建议复现

## RS-Wiki

### 1. 推荐系统评测指标

* 准确率(precision)和召回率(recall)

  > 这里的准确率和精确率(accuracy)是不同的。

  <img src="PaperReading.assets/image-20220331220052143.png" alt="image-20220331220052143" style="zoom:67%;" />

  准确率：即在所有预测为A类的结果中，确实属于A类的比例。
  $$
  P = \frac{TP}{TP+FP}
  $$
  召回率：比如在所有想要给用户推荐的结果中，把用户喜欢的结果推荐出的比例。
  $$
  R = \frac{TP}{TP+FN}
  $$
  通常情况下准确率越高，召回率越低；召回率越高，准确率越低。

  将两个指标进行作图可以得到 P-R 图：

  <img src="PaperReading.assets/image-20220331221551350.png" alt="image-20220331221551350" style="zoom:67%;" />

  

  对于三个模型得到的 P-R 图，取三者的平衡点(BEP, Break-Even Point)，BEP 值越大表明模型更优。

* F1

* DCG

* AUC

### 2. 常见数据集

数据集介绍：[Recommender Systems Datasets (ucsd.edu)](https://cseweb.ucsd.edu/~jmcauley/datasets.html)

* [MovieLens 数据集](https://grouplens.org/datasets/movielens/)
* [Book-Crossing Dataset](http://www2.informatik.uni-freiburg.de/~cziegler/BX/)
* [criteo 数据集](https://www.kaggle.com/datasets/mrkmakr/criteo-dataset)

## 其他

### 人物

🔵上海交大王鸿伟

[个人网站](https://hongweiw.net/)，[王鸿伟 - 知乎](https://www.zhihu.com/people/hwwang55/answers)

🔵其他大神：

[推荐系统干货总结（第二节 相关学者）](https://zhuanlan.zhihu.com/p/34004488)

🔵知乎推荐算法博主

[潜心](https://www.zhihu.com/people/gzy-33-55/posts)

### 推荐读物

1. [研究生发论文是先有idea再做实验，还是先做实验再有idea](https://www.zhihu.com/question/315337121/answer/627502932)

   ![img](PaperReading.assets\v2-950c23b6beb2374fdb7c560a9ee93252_r.jpg)

2. [没有导师的指导，研究生如何阅读文献、提出创见、写论文？](https://www.zhihu.com/question/23647187/answer/568803695)

   **最开始的时候阅读论文，最好能细致一点，把论文之间的引用关系理清楚，把近几年的发展脉络理清楚。**我当年开始第一个工作的时候，就是把我论文需要引用的二十多篇论文的主要思想、方法都写了下来，把引用关系画成了一个DAG图。**当你入门之后，你需要有快速阅读一篇文章并掌握其核心贡献点的能力**，而不要再花费很多时间来标注。

3. [Recommender Systems  (d2l.ai)](http://d2l.ai/chapter_recommender-systems/index.html)

4. [推荐系统论文阅读 - 简书](https://www.jianshu.com/u/720c6853ff98)

3. [推荐算法文章 - alg](https://www.6aiq.com/member/alg)

## 参考文献

[^1]: David Milne and Ian H Witten. 2008. Learning to link with wikipedia. In CIKM. ACM, 509–518.
[^2]:Avirup Sil and Alexander Yates. 2013. Re-ranking for joint named-entity recognition and linking. In Proceedings of the 22nd ACM international conference on Conference on information & knowledge management. ACM, 2369–2374.
