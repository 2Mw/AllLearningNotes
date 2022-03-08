# DL分类笔记

> 用于整理不同方向的知识点

## NLP

### 词嵌入之word2vec

**基本介绍**：

文本是一种非结构化的数据信息，是不可以被直接计算的，因此需要进行文本表示(Word Representation)，其作用就是将非结构化的数据转为结构化的数据，方便计算机对其计算处理。

文本表示方法有三种：独热编码、整数编码以及词嵌入(word2vec, GloVe)。

独热编码对于有成千上万个词的时候，向量长度就会非常长，并且99%的信息都是0，并且其无法表达词与词之间的关系，过于稀疏导致计算和存储效率都不高；

整数编码即给每一个词标序号比如1，2，3...，其缺点也是无法表达词与词之间的关系。

词嵌入可以将文本以一种低维向量的方式来表示，不像独热编码那么长，并且语义相似的词在向量空间中也相近。word2vec是基于统计方法来获取词向量的方法，这种算法有两种训练模式：一是通过上下文来预测当前词(CBOW)，二是通过当前词来预测上下文(Skip-gram)

<img src="E:\Notes\DeepLearning\notes\DL分支笔记.assets\2019-09-26-cbow-16463705524281.png" alt="CBOW通过上下文来预测当前值" style="zoom:50%;" />

<img src="E:\Notes\DeepLearning\notes\DL分支笔记.assets\2019-09-26-Skip-gram-16463705551473.png" alt="Skip-gram用当前词来预测上下文" style="zoom:50%;" />

为了提高训练速度，word2vec可以采用的方法有：Negative sampling(负采样)，Hierarchical Softmax。两者配合subsampling可以取得较好的效果。

**优缺点：**

* Word2vec 会考虑上下文，效果要更好（不如18年之后的方法），速度快。
* 由于词和向量是一对一的关系所以**多义词**的问题无法解决。通用性强但是无法针对特定任务做动态优化。



**参考：**

1. [一文看懂词嵌入 word embedding（2种主流算法+与其他文本表示比较）](https://easyai.tech/ai-definition/word-embedding/)
2. [秒懂词向量Word2vec的本质](https://zhuanlan.zhihu.com/p/26306795)

### Transformer

参考：

1. [machine-learning-notes](https://luweikxy.gitbook.io/machine-learning-notes/)
2. [详解Transformer](https://zhuanlan.zhihu.com/p/48508221)



## RS

[深度学习与推荐系统完结篇（知识、论文、源码、数据集&行业应用）](https://mp.weixin.qq.com/s/n5ZplTadX6jtXIwJmXHivg)