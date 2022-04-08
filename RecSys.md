# RecSys Notes

Take notes of the knowledge of recommendation system whatever in the manners of traditional ways or deep learning ways.

## 零. 初步

### 1. 推荐系统评测指标

* 准确率(precision)和召回率(recall)

  > 这里的准确率和精确率(accuracy)是不同的。

  <img src="RecSys.assets/image-20220331220052143.png" alt="image-20220331220052143" style="zoom:67%;" />

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

  <img src="RecSys.assets/image-20220331221551350.png" alt="image-20220331221551350" style="zoom:67%;" />

  

  对于三个模型得到的 P-R 图，取三者的平衡点(BEP, Break-Even Point)，BEP 值越大表明模型更优。

* F1

* DCG

### 2. 常见数据集

数据集介绍：[Recommender Systems Datasets (ucsd.edu)](https://cseweb.ucsd.edu/~jmcauley/datasets.html)

* [MovieLens 数据集](https://grouplens.org/datasets/movielens/)
* [Book-Crossing Dataset](http://www2.informatik.uni-freiburg.de/~cziegler/BX/)

## 一. 协同过滤算法

协同过滤算法(Collaborative Filtering)属于传统推荐算法，其下又可以细分为很多类，如图：

![image-20220405101514566](RecSys.assets/image-20220405101514566.png)

学术界也对于协同过滤算法有很多深入的研究，提出了很多办法，比如基于邻域(neighbor-based)的方法，隐语义模型(latent factor model)，基于图的随机游走算法（random walk on graph）等。其中应用最广泛的就是基于邻域的方法。

### 1. 基于邻域的推荐算法

基于邻域的算法主要分为以下两种：

1. 基于用户的协同过滤算法(UserCF)，给用户推荐和他兴趣相似的用户喜欢的物品
2. 基于物品的协同过滤算法(ItemCF)，给用户推荐和他之前喜欢的物品相似的物品

🔵用户/物品相似度的计算：

1. 余弦相似度

   余弦相似度是通过测量两个向量夹角的余弦值来度量其之间的相似性。其结果和向量的长度无关，只与向量的方向有关。
   $$
   \text{similarity}=\cos(\theta)=\dfrac{A\cdot B}{||A||\ ||B||}
   $$
   对于离散值的计算有：
   $$
   \text{similarity}=\dfrac{|N(u)\cap N(v)|}{\sqrt{|N(u)|\cdot |N(v)|}}
   $$
   比如在用户推荐中，用户 A 对物品 {a, b, d} 有过交互，用户 B 对物品 {a, c} 有过交互，则用户A和用户B之间兴趣的相似度为：
   $$
   s_{AB}=\dfrac{|\{a,b,d\}\cap \{a,c\}|}{\sqrt{|\{a,b,d\}|\cdot |\{a,c\}|}}=\dfrac1{\sqrt6}
   $$
   
2. Jaccard 相似系数

   也可以成为交并比
   $$
   s=\dfrac{|A\cap B|}{|A\cup B|}
   $$

3. 皮尔逊相关系数

   用于度量两个变量 X 和 Y 之间的现象相关程度，值介于-1和1之间，定义为两个变量的协方差除以它们的标准差的乘积：
   $$
   \rho_{X,Y}=\dfrac{\text{cov}(X,Y)}{\rho_X\rho_Y}{\displaystyle ={\frac {\sum \limits _{i=1}^{n}(X_{i}-{\overline {X}})(Y_{i}-{\overline {Y}})}{{\sqrt {\sum \limits _{i=1}^{n}(X_{i}-{\overline {X}})^{2}}}{\sqrt {\sum \limits _{i=1}^{n}(Y_{i}-{\overline {Y}})^{2}}}}}}
   $$
   当 $|\rho|=1$ 时表明两个变量之间有着明显的线性关系，并且当 $\rho=1$ 为正相关，-1 时为负相关。

4. 改进版用户相似度

   最简单的余弦相似度过于粗糙，如果两个用户都曾经买过新华字典，丝毫不能说明他们的兴趣相似，因为绝大部分中国人小时候都买过。如果两个用户对冷门物品采取过同样的行为则更能说明他们兴趣的相似度，因此因该对热门物品推荐进行惩罚：
   $$
   \text{similarity}=\dfrac{\displaystyle\sum_{i\in N(u)\cap N(v)}\frac1{\log_2{1+|N(i)|}}}{\sqrt{|N(u)|\cdot |N(v)|}}
   $$
   

🔵UserCF

> UserCF在实际应用中并不多(2012年《推荐系统实战》)

基于用户的协同过滤算法主要分为两个步骤：

1. 找到和目标用户兴趣度相似的用户集合
2. 找到这个集合中用户喜欢的，且用户没有见过的物品推荐给目标用户

UserCF的缺点：

* 随着网站用户数目越来越多，用户相似度矩阵计算越来越困难，计算复杂度是 $O(n^2)$ 的
* UserCF 的可解释性很差

🔵ItemCF

> ItemCF是当前业界应用最多的算法。Amazon，Netflix，YouTube 等都是基于该算法改进。

基于物品的协同过滤算法是给用户推荐和他们之前喜欢的物品相似的物品。

ItemCF 不是根据计算物品内容来计算物品之间的的相似度，主要是提高分析用户的行为记录计算物品之间的相似度。

ItemCF 主要分为两步：

1. 计算物品之间的相似度
2. 根据物品的相似度和用户的历史行为给用户生成推荐列表

### 2. 基于模型的推荐算法

🔵因子分解机（Factorization Machines）

改进前：
$$
\hat y(x)=\omega_0+\sum_{i=1}^n\omega_ix_i+\sum_{i=1}^n\sum_{j=i+1}^n\omega_{ij}x_ix_j
$$
改进后：
$$
\hat y(x)=\omega_0+\sum_{i=1}^n\omega_ix_i+\sum_{i=1}^n\sum_{j=i+1}^n<v_i,v_j>x_ix_j
$$
将二阶信息中的权重 $w_{ij} $ 分解为 $<v_i,v_j>$ 

改进后的公式时间复杂度为 $O(kn^2)$ 级别的，大佬又将二次项降为线性时间复杂度 $O(kn)$：
$$
\sum_{i=1}^n\sum_{j=i+1}^n<v_i,v_j>x_ix_j=\dfrac12\sum_{f=1}^k((\sum_{i=1}^nv_{i,f}x_i)^2-\sum_{i=1}^n(v_{i,f}x_i)^2)
$$
