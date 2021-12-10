# 深度学习

[TOC]

视频资料：[李宏毅ML 2021 Spring](https://speech.ee.ntu.edu.tw/~hylee/ml/2021-spring.html)

## 初始环境与配置

> 所有操作最好在`Anaconda Prompt`中进行。

### Cuda，cudnn安装

Cuda安装需要和电脑中的显卡驱动匹配。cudnn要安装同cuda相同的版本。

### PIP换国内源

```sh
# 清华源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
# 北外
pip config set global.index-url https://mirrors.bfsu.edu.cn/pypi/web/simple
# 或：
# 阿里源
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
# 腾讯源
pip config set global.index-url http://mirrors.cloud.tencent.com/pypi/simple
# 豆瓣源
pip config set global.index-url http://pypi.douban.com/simple/
```

需要**关闭**代理软件。

### Conda换源

需要找到`.condarc`文件，在文件中添加对应配置，比如[anaconda北外镜像 ](https://mirrors.bfsu.edu.cn/help/anaconda/)

在`.condarc`文件添加：

```txt
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/main
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/r
  - https://mirrors.bfsu.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.bfsu.edu.cn/anaconda/cloud
  msys2: https://mirrors.bfsu.edu.cn/anaconda/cloud
  bioconda: https://mirrors.bfsu.edu.cn/anaconda/cloud
  menpo: https://mirrors.bfsu.edu.cn/anaconda/cloud
  pytorch: https://mirrors.bfsu.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.bfsu.edu.cn/anaconda/cloud
```

运行 `conda clean -i` 清除索引缓存，保证用的是镜像站提供的索引。

运行 `conda create -n myenv numpy` 测试。

### Jupiter NoteBook工作目录配置

Jupiter NoteBook默认工作目录在C盘，看不了其他硬盘的文件。

* `jupyter notebook --generate-config`产生notebook配置文件
* 打开文件找到 `c.NotebookApp.notebook_dir = ''`并且修改到自己想要的目录
* 将快捷方式中的`"%USERPROFILE%/"`删去即可。

### 安装框架：

Pytorch：[PyTorch Install](https://pytorch.org/get-started/locally/)

```sh
conda install pytorch torchvision torchaudio cudatoolkit=11.1 -c pytorch -c conda-forge
# 或者
pip3 install torch==1.9.1+cu111 torchvision==0.10.1+cu111 torchaudio===0.9.1 -f https://download.pytorch.org/whl/torch_stable.html
```

### Jupyter闪退解决

问题1：

```
Exception in callback <TaskWakeupMethWrapper object at 0x00000288B64989D0>(<Future finis...0b5"\r\n\r\n'>)
handle: <Handle <TaskWakeupMethWrapper object at 0x00000288B64989D0>(<Future finis...0b5"\r\n\r\n'>)>
...
Bad file descriptor (C:\projects\libzmq\src\epoll.cpp:100)
```

解决方案：[Link 1](https://blog.csdn.net/weixin_40981660/article/details/119897759)，降级`pyzmq`

```
pip install pyzmq==19.0.2
```

### 安装Jupiter插件

1. 黑色护眼模式jupyterthemes

   ```sh
   pip install jupyterthemes 			# 安装
   pip install --upgrade jupyterthemes	# 更新
   jt -l								# 查看可用主题
   ```

2. nbextensions（Anaconda2021.05不建议使用）

   [链接](https://github.com/ipython-contrib/jupyter_contrib_nbextensions)

   * 安装此扩展

     ```sh
     # pip
     pip install jupyter_contrib_nbextensions
     # conda
     conda install -c conda-forge jupyter_contrib_nbextensions
     ```

   * 安装js和css文件

     ```sh
     jupyter contrib nbextension install --user
     ```

   * 开启和关闭扩展

     ```sh
     jupyter nbextension enable codefolding/main
     jupyter nbextension disable codefolding/main
     ```

   * 安装开启nbextensions的配置器

     ```sh
     pip install jupyter_nbextensions_configurator
     jupyter nbextensions_configurator enable --user
     ```


### Jupyter魔法函数

> 可以直接在IPython中使用

参考：[Built-in magic commands — IPython](https://ipython.readthedocs.io/en/stable/interactive/magics.html)

常见函数：

* ` %matplotlib inline`函数，它的含义就是告诉IPython，我们的绘图模式是内嵌（inline）模式，即将绘图直接显示在当前的网页上。

* `%timeit`计时测试参数，`%%timeit`为测试整个代码块

  例如：

  ```python
  %timeit -r1 -n2 time.sleep(2)
  # 2.01 s ± 0 ns per loop (mean ± std. dev. of 1 run, 2 loop each)
  ```

  表示这段代码运行一次(`-r`)，每次循环两次(`-n`)所消耗的时间。

  ```python
  %%timeit -n2 -r1 
  time.sleep(1)
  print('sep')
  time.sleep(1)
  '''
  sep
  sep
  2.02 s ± 0 ns per loop (mean ± std. dev. of 1 run, 2 loops each)
  '''
  ```

  也可以使用在一个代码块中。

* `%run` 允许`.py`的python代码

* `%cd` 运行shell命令，类似还有`%pwd`

* `%%latex`渲染LaTEx

### Colab使用

开始shell

```python
!ls  # 这个会开启新的shell
!nvidia-smi  # 查看显卡配置
!gdown --id 'token' --output a.jpg # 从谷歌drive下载文件
%cd	 # notebook 魔法函数 cd使用魔法函数
```

和自己的GD挂载：

```python
from google.colab import drive
drive.mount('/content/dreve',force_remount=True)
```

## 框架使用技巧

### Pandas

🔵`read_csv()`

读取csv文件，默认参数`header=0`，读取表格第一行为表头。 

🔵填充`NAN`值

```python
inputs, outputs = data.iloc[:, 0:2], data.iloc[:, 2]
inputs = inputs.fillna(inputs.mean())
```



### Numpy

🔵广播

广播对不同形状(shape)的数组进行数值计算的方式， 对数组的算术运算通常在相应的元素上进行。

* 两个矩阵形状相同，就是对应位置的相乘

  ```python
  a = np.array([1,2,3,4]) 
  b = np.array([10,20,30,40]) 
  print (a*b) // [10,40,90,160]
  ```

* 两矩阵形状不同

  加入a的shape为`(s1,s2,s3,s4...,sn)`,b不为标量且其shape为`(x1,x2,x3)`，b的维度小于a且有$x_1=s_{n-2},x_2=s_{n-1},x_3=s_{n}$之类，则可以进行广播。

🔵节约内存

```python
Y = Y + X		# (X)
Y[:] = Y + X	# (√)
```

### PyTorch

🔵数据集处理：

`torch.utils.data.Dataset`

Dataset分为两种：1. map-style dataset 2. iterable-style dataset

常用的是map-style dataset

映射形式的数据集的类需要实现`__getitem__()` 和 `__len__()`方法来通过index进行数据访问和查看数据集的长度。

For example, such a dataset, when accessed with dataset[idx], could read the idx-th image and its corresponding label from a folder on the disk.

模板：

```python
class ImgDataset(dataset.Dataset):  # 继承父类dataset
    def __init__(self, x, y=None, transform=None):
        self.x = x
        self.y = y
        if y is not None:
            self.y = tc.LongTensor(y)  # y 应该为Long Tensor
        self.transform = transform

    def __len__(self):  # 实现函数__len__
        return len(self.x)

    def __getitem__(self, index):  # 实现函数__getitem__
        X = self.x[index]
        if self.transform is not None:
            X = self.transform(X)
        if self.y is not None:
            return X, self.y[index]
        else:
            return X
```

`torch.utils.data.DataLoader` 是pytorch数据加载中最实用的包，能够以迭代的形式加载数据集。

其可以：

* 映射以及迭代的方式加载数据集（map-style and iterable-style）
* 自定义数据加载方式
* 自动分批（batch）
* 单线程或者多线程加载数据
* …………

🔵`nn.Module`的使用：

用于搭建网络模型：

```python
class Classifier(nn.Module):    # 继承父类nn.Module
    def __init__(self):
        super(Classifier, self).__init__()
        # torch.nn.Conv2d(in_channels, out_channels, kernel_size, stride, padding)
        # torch.nn.MaxPool2d(kernel_size, stride, padding)
        # 原图输入维度 (128, 128, 3)
        self.cnn = nn.Sequential(
            # 输入为彩色图片，输出通道64 ，3X3的卷积核，1步长， 填充长度为1=>为了凑128，不然126不容易解决
            nn.Conv2d(3, 64, 3, 1, 1),  # (128, 128, 64)
            nn.BatchNorm2d(64), # norm2d的输入为特征数量，并进行标准化，这里可以设置momentum和eps
            nn.ReLU(),  # 进行ReLU函数转换
            nn.MaxPool2d(2, 2, 0),  # (64, 64, 64)

            # 上一个conv + maxpool的输出为这次的输入
            """
            	...
           	 """

            nn.Conv2d(512, 512, 3, 1, 1),  # (8, 8, 512)
            nn.BatchNorm2d(512),
            nn.ReLU(),
            nn.MaxPool2d(2, 2, 0),  # (4, 4, 512)
        )

        self.fc = nn.Sequential(
            nn.Linear(512 * 4 * 4, 1024),
            nn.Dropout(0.5),
            nn.ReLU(),
            nn.Linear(1024, 512),
            nn.Dropout(0.5),
            nn.ReLU(),
            nn.Linear(512, 11)
        )

    def forward(self, x):
        out = self.cnn(x) # 进行CNN转换
        out = out.view(out.size()[0], -1)   # 总感觉这句话没什么用
        return self.fc(out) # 输入网络，有四层网络：输入层，隐藏层1，隐藏层2，输出层。最后输出11维度的向量（有11种类别的食品）。
```

🔵随机种子

参考：

* [torch.backends.cudnn.benchmark](https://zhuanlan.zhihu.com/p/73711222)
* [torch.backends.cudnn.deterministic](https://zhuanlan.zhihu.com/p/141063432)

`torch.backends.cudnn.deterministic`将这个 flag 置为`True`的话，每次返回的卷积算法将是确定的，即默认算法。如果配合上设置 Torch 的随机种子为固定值的话，应该可以保证每次运行网络的时候相同输入的输出是固定的

```python
def same_seeds(seed):
    torch.manual_seed(seed)
    if torch.cuda.is_available():
        torch.cuda.manual_seed(seed)
        torch.cuda.manual_seed_all(seed)  # 为所有GPU都进行种子随机
    np.random.seed(seed)  
    torch.backends.cudnn.benchmark = False
    torch.backends.cudnn.deterministic = True
```

### matplotlib



### seaborn



### 垃圾回收

> gc包是python自带的包

```python
import gc

del train, train_label, train_x, train_y, val_x, val_y
gc.collect()
```

### 数据类型相互转换

```python
# numpy ndarray -> python list
a = np.array(matrix)
a.tolist()
# torch.tensor(gpu) -> numpy ndarray
var.cpu().data.numpy()
```

## 一 回归regression

### 损失函数

MAE(mean absolute error)：$L=\dfrac1N\sum_ne_n,e=|y-\hat y|$

MSE(mean square error): $L=\dfrac1N\sum_ne_n, e=(y-\hat y)^2$

### 优化器(optimizer)之梯度下降法(GD)：

参考：

* [批量梯度下降(BGD)、随机梯度下降(SGD)、小批量梯度下降(MBGD)](https://zhuanlan.zhihu.com/p/72929546)
* [优化总结：SGD，Momentum，AdaGrad，RMSProp，Adam](https://blog.csdn.net/u010089444/article/details/76725843)

目的就是找到$y=\omega x+b$中最优的
$$
w^*,b^*=\arg \min_{w,b} L
$$
**训练方法：**

梯度下降法（Gradient Descent）

1. 随机选取一个初始值$\omega_0,b_0$
2. 计算对应的斜率$\dfrac{\alpha L}{\alpha \omega}|_{w=w_0}$
3. 移动新的位置：$w_1=w_0-\eta \dfrac{\alpha L}{\alpha \omega}|_{w=w_0}$（$\eta$是学习速率，为超参数）

缺陷：

<img src="https://i.loli.net/2021/10/26/LkR5nNZS3hXloUt.png" alt="image-20210928004231918" style="zoom:50%;" />

会陷入局部最优点，可能会找不到最低全局最低点，可以通过选取多个不同初始点来进行解决。

### 学习速率

### 激活函数

一个函数可以写成多个函数无限加成得到，有点类似于傅里叶变换和逆变换。

<img src="https://i.loli.net/2021/10/26/1r7uasMvp5iKfco.png" alt="image-20210928010124002" style="zoom:50%;" />

* sigmoid函数：$z=\dfrac1{1+e^{-y}}, y=\omega x +b$

  红线函数可以写成：${\color{\red}red}=b+\sum_i c_i\text{sigmoid}(b_i+\sum_j w_{ij}x)$

<img src="https://i.loli.net/2021/10/26/KYforlPGSzX3vDn.png" alt="image-20210928010930279" style="zoom:50%;" />

<img src="https://i.loli.net/2021/10/26/sfPpBz86umIj9l1.png" alt="image-20210928011124526" style="zoom:50%;" />

​	使用线性代数即可以表示为：$t=b+c^T\sigma(b+Wx)$

* ReLU函数（Rectified Linear Unit）

  $z=c\max(0,b+\omega x)$

  ![image-20210928012905189](https://i.loli.net/2021/10/26/GI9jiY68d5oBRCV.png)

### 更新参数

update和epoch的区别，一个epoch中可能会更新多次参数

<img src="https://i.loli.net/2021/10/26/BAdgteWv8ErhH29.png" alt="image-20210928012524132" style="zoom:50%;" />

### 过拟合与欠拟合

解决过拟合：

* 添加数据集
* 数据增强
* 早停机制
* 正则化
* Dropout
* K-fold交叉验证法

## 二 训练技巧

### 局部最优点和鞍点

> Local minimum and Saddle point可以统称为critical point

如何判断梯度为0的点，来判断是否是局部最优点还是鞍点？
$$
L(\theta)\approx L(\theta')+(\theta-\theta')^Tg+\dfrac12(\theta-\theta')^TH(\theta-\theta')^T
$$
斜率为0，可以简化为：
$$
L(\theta)\approx L(\theta')+\dfrac12(\theta-\theta')^TH(\theta-\theta')\\
令\quad v=(\theta-\theta')^T\\
L(\theta)\approx L(\theta')+\dfrac12v^THv^T
$$
对于二阶导：根据“大小小大”规则来判断是局部最小值或者最大值；如果二阶导某点附近有大有小即表示为鞍点。

算出海森矩阵（Hessian Matrix）的特征值，如果有正有负则是Saddle Point。

> 在高维空间中，遇到的鞍点要比局部最优点常见。

🔵好的局部最优值：

<img src="https://i.loli.net/2021/10/26/31GJEnR6B5WM9bX.png" alt="image-20211011222434558" style="zoom: 67%;" />

Flat局部最优要比Sharp局部最优更**好**，当在验证集上进行偏移的时候，Sharp的变化更大，误差更大。

### 训练方法—Batch和Momentum

🔵为什么使用Batch：

使用全部数据集进行更新参数只能更新一次。

<img src="https://i.loli.net/2021/10/26/4td2ng6IwJWxa5Z.png" alt="image-20211011220401996" style="zoom:50%;" />

需要合理设置Batch大小，当batch较小的时候，大小为10和1000并差不了多少，但是分割的数量累加上去，batch为10的训练时间可能要比batch为1000的要长。

但是更小的batch要比大的batch训练更优。

🔵Momentum

<img src="https://i.loli.net/2021/10/26/bwhsopONBiYlQPT.png" alt="image-20211011223008769" style="zoom:50%;" />

### 自适应学习速率

原来的更新参数：
$$
\theta_i^{t+1}=\theta_i^{t}-\eta g_i^t,\quad g_i^t=\dfrac{\partial L}{\partial \theta_i}|_{\theta=\theta^t}
$$
自适应学习速率：
$$
\theta_i^{t+1}=\theta_i^{t}-\dfrac\eta{\sigma_i^t} g_i^t,\quad g_i^t=\dfrac{\partial L}{\partial \theta_i}|_{\theta=\theta^t}
$$
这里的$\sigma_i^t$是一个依赖梯度的参数。

不同的学习速率策略：

* AdaGrad

  <img src="https://i.loli.net/2021/10/26/GPjWMSzhyU9XBse.png" alt="image-20211012101244735" style="zoom:50%;" />

* RMSProp（改进版AdaGrad）

  在梯度大的时候移动小，在梯度小的时候移动大

  <img src="https://i.loli.net/2021/10/26/YTPsBxZWEqv2kwg.png" alt="image-20211012101619584" style="zoom:50%;" />

* Adam：RMSProp+momentum

### 学习速率调度

* 学习速率衰减（Learning Rate Decay）随着训练进行接近终点的时候，减小学习速率。
* 学习速率预热（Warm up），学习速率先增大再变小

<img src="https://i.loli.net/2021/10/26/V4n6QApLhFbtPRS.png" alt="image-20211012102458093" style="zoom:67%;" />

### Softmax函数

Softmax函数，或称归一化指数函数。
$$
{\displaystyle \sigma (\mathbf {z} )_{j}={\frac {e^{z_{j}}}{\sum _{k=1}^{K}e^{z_{k}}}}},\text{for}\;j=1,...,K.
$$
目的就是让结果值归一化再$[0,1]$范围内。

### 逻辑回归

进行分类的损失函数一般使用交叉熵（Cross Entropy），最小化交叉熵同最大化可能性是等价的。

交叉熵函数：
$$
e=-\sum_i\hat y_i\ln y_i'
$$

### 批量标准化（Batch Normalization）

基本方法：
$$
x=\dfrac{x-\text{mean}(x)}{\text{std}(x)}
$$
其他归一化：

- batchNorm是在batch上，对NHW做归一化，对小batchsize效果不好；
- layerNorm在通道方向上，对CHW归一化，主要对RNN作用明显；
- instanceNorm在图像像素上，对HW做归一化，用在风格化迁移；
- GroupNorm将channel分组，然后再做归一化；
- SwitchableNorm是将BN、LN、IN结合，赋予权重，让网络自己去学习归一化层应该使用什么方法。

<img src="https://i.loli.net/2021/10/26/oO5QjNvFCeD8Vzr.png" alt="image-20211013103344912" style="zoom:67%;" />

参考：[pytorch常用normalization函数](https://www.cnblogs.com/wanghui-garcia/p/10877700.html)

## 三 决策树

> 好处：可解释，能处理分类和数值类的特征
>
> 缺点：不稳定，过拟合，不容易并行计算（由根到叶子需要顺序进行）

### 决策流程

<img src="https://i.loli.net/2021/10/26/BxEozcTK7gCtPAi.png" alt="image-20211017094034770" style="zoom:80%;" />

决策树学习目的是为了产生泛化能力强的决策树，

<img src="https://i.loli.net/2021/10/26/Kli5Sz3RvXLT6tA.png" alt="image-20211017094640233" style="zoom:80%;" />

以下三种情形会导致递归返回：

1. 当前结点包含的样本同属于一个分类
2. 当前属性值为空，无法分类
3. 当前节点包含的样本集合为空，无法划分

在（2）情况下，会将当前节点划分为叶节点，其本身划分为含样本最多的结点；在（3）情况下，同样将此结点划分为叶节点，其父类为含样本最多的节点。

### 信息增益 增益率 基尼指数

<h4>信息熵</h4>

度量样本集合中纯度的指标。

假定当前样本集合$D$中的第 $k$ 类样本所站的比例为 $p_k\ (k=1,2,...,|y|)$，则D的信息熵定义为：
$$
\text{Ent}(D)=-\sum_{k=1}^{|\mathscr{y}|}p_k\log_2p_k
$$
**信息熵越小，纯度越高。**

<h4>信息增益</h4>

对于样本的离散属性 $a$ 可能有 $V$ 个不同的取值${a^1,a^2,...,a^V}$，对于信息增益，对应分类样本数越多，影响越大。信息增益定义为：
$$
\text{Gain}(D,a)=\text{Ent}(D)-\sum_{k=1}^{|V|}\dfrac{|D^v|}{|D|}Ent(D^v)
$$

<h4>增益率</h4>

$$
\text{Gain\_ratio}(D, a)=\dfrac{\text{Gain}(D,a)}{IV(a)}, \; IV(a)=-\sum_{k=1}^{|V|}\dfrac{|D^v|}{|D|}\log_2\dfrac{|D^v|}{|D|}
$$

> 增益率可能对取值数目较少的属性有所偏好

<h4>基尼指数</h4>

数据集的纯度也可以使用**基尼值**来表示：
$$
\text{Gini}(D)=\sum_{k=1}^{|y|}\sum_{k' \neq k}p_kp_{k'}=1-\sum_{k=1}^{|y|}p_k^2
$$
基尼指数定义： 
$$
\text{Gini\_index}(D, a)=\sum_{v=1}^v\dfrac{|D^v|}{|D|}\text{Gini}(D^v)
$$
基尼指数最小的属性作为最优划分属性。

### 剪枝操作

<h4>预剪枝</h4>

<h4>后剪枝</h4>

### 改进方法

随机森林：生成多个决策树，随机Baging几棵树并且有可能重复。

Boosting：Gradient Boosting

## 四 SVM支持向量机

参考：

[明白支持向量机SVM](https://zhuanlan.zhihu.com/p/74484361)

> SVM解决了两个问题：
>
> 1. 线性可分问题——无数个超平面哪个最合适
> 2. 将线性可分的问题推广到线性不可分——核函数

### 线性可分和线性不可分

<img src="https://i.loli.net/2021/10/26/r5bB6IAixZ8SvUV.png" alt="image-20211013125411522" style="zoom:67%;" />

判断标准：是否可以由一个直线将两个类别分开（2D），三维是平面，高维是超平面。

如果低维不可分，可以放在高维空间上进行分割。

线性可分的精确定义：

假设：
$$
X_i=(x_{i1},x_{i2}), \omega=(\omega_{1},\omega_{2})\\
若\;y_i=+1,则\; \omega^TX_i+b>0\\
若\;y_i=-1,则\; \omega^TX_i+b<0\\
对于\; \forall i=1\sim N,有\; y_i(\omega^TX_i+b)>0
$$

### 最大边距超平面

> 最优化问题

<img src="https://i.loli.net/2021/10/26/EJodhH4GT3OrnFD.png" alt="image-20211013130905212" style="zoom:50%;" />

对于线性可分问题，可能有无数个超平面将类别分开。

<img src="https://i.loli.net/2021/10/26/uHAXCEJo9fDtnTy.png" alt="image-20211013171914343" style="zoom:50%;" />

图中的三个点就是**支持向量**。

🔵样本空间中任意一点$x$到超平面$(\omega,b)$的距离可以表示为：
$$
r=\dfrac{|\omega^Tx+b|}{||\omega||},\; ||\omega||为L2范数
$$
证明：

<img src="https://i.loli.net/2021/10/26/vW8a9P3X7SjVewg.png" alt="image-20211013165229599"  />



如图，假设B点为A点在超平面的投影，点A和点B之间的距离为$\gamma$，超平面可以表示为$(\vec\omega,b)$，则$\vec{BA}=\gamma*\dfrac{\omega}{||\omega||}$

又由于 $\vec{OB}=\vec{OA}-\vec{BA}，i.e.\;x_B=x_A-\vec{BA}$ ， 对于B点有$\omega^Tx_B+b=0$，因此有

$\omega^T(x_A-\gamma*\dfrac{\omega}{||\omega||})+b=0$，得$\omega^Tx_A-\gamma*{||\omega||}+b=0$ 即 $r=\dfrac{\omega^Tx_A+b}{||\omega||}$。

🔵为什么支持向量对应超平面为$\omega^Tx+b=\pm 1$?

* $\pm 1$绝对值相同保证所求超平面与两个对应平面距离相同
* $\omega^Tx+b=1$与$\alpha\omega^Tx+\alpha b=\alpha$所表示的平面相同，用1可以方便表示。

<img src="https://i.loli.net/2021/10/26/AOBp2JgtIZ8kR3u.png" alt="image-20211013172616828" style="zoom:67%;" />

🔵为什么最优化目标为 ${\displaystyle \min_{\omega,b}}\dfrac12||\omega||^2$？

由于支持向量机的边界平面为$\omega^Tx+b= 1$，因此离超平面最近点的距离为$r=\dfrac{1}{||\omega||}$，两个边界超平面的**间隔**$\gamma=\dfrac{2}{||\omega||}$



🔵边距最大化的三个充分必要条件：

1. 能够分开类别
2. 与之平行的两条平行线拥有最大边距（Large Margin）
3. 位于两条平行线中间，到所有支持向量机的距离相同

### 核函数

比如典型的线性不可分问题——异或，以及非线性决策边界。

> 核函数即将数据有低维映射到高维

<img src="https://i.loli.net/2021/10/26/TCpjhqYvDikPV4H.png" alt="image-20211013161608903" style="zoom: 50%;" />

Cover定理：

🔵如何寻找核函数：

* 相似度：Gussian Kernel

* 无需找出核函数具体形式：

  <img src="https://i.loli.net/2021/10/26/F5ZBIHGP6fh4Vkd.png" alt="image-20211013163207061" style="zoom:50%;" />



🔵常见的核函数：

| 名称       | 表达式$\kappa(x_i,x_j)$                   | 参数               |
| ---------- | ----------------------------------------- | ------------------ |
| 线性核     | $x_i^Tx_j$                                |                    |
| 多项式核   | $(x_i^Tx_j)^d$                            | $d\ge1$            |
| 高斯核     | $\exp(-\dfrac{||x_i-x_j||^2}{2\sigma^2})$ | $\sigma>0$         |
| 拉普拉斯核 | $\exp(-\dfrac{||x_i-x_j||}{\sigma})$      | $\sigma>0$         |
| Sigmoid核  | $\tanh(\beta x_i^Tx_j+\theta)$            | $\beta>0,\theta<0$ |

> 高斯核也叫RBF核，在sklearn中默认参数是高斯核

### 凸二次规划

🔵二次规划定义：

目标函数是二次项的：$\dfrac12||\omega||^2=\dfrac12\sum\omega_i^2$

限制条件是一次项的：$y_i(\omega^TX_i+b)\ge1(i=1\sim N)$

二次规划问题要么无解，要么无穷多解。

🔵KKT条件

推广到多个约束：
$$ { }
\min_x\;f(x) \tag{A} \\ 
s.t.\;h_i(x)=0\;(i=1,...,n)\\
\quad\ \ \;g_j(x)=0\;(j=1,...,n) 
$$
则对应的拉格朗日函数为：
$$
L(x,\lambda,\mu)=f(x)+\sum_{i=1}^m\lambda_ih_i(x)+\sum_{j=1}^n\mu_jh_j(x) \tag{B}
$$
对于主问题$(A)$，在公式$(B)$的基础上，得到原问题的对偶问题为：
$$
\begin{align*}
\Gamma(\lambda,\mu)& =\inf_{x\in D}L(x,\lambda,\mu)\\&=\inf_{x\in D}(f(x)+\sum_{i=1}^m\lambda_ih_i(x)+\sum_{j=1}^n\mu_jh_j(x))\\
\end{align*}
$$



### 软间隔与正则化

<img src="https://i.loli.net/2021/10/26/cHmS1MJGwPYaKEZ.png" alt="image-20211014103151297" style="zoom:67%;" />

在实际中，很难确定适合的超平面将所有的样本严格分开，“**软间隔**”即允许支持向量机在一些样本上出错。

优化目标可以写为：
$$
\min_{\omega,b}\frac12||\omega||^2+C\sum_{i=1}^m\mathscr{l}(y_i(\omega^Tx_i+b)-1),\;\mathscr{l}\text{ is\ loss\ function}
$$
损失函数又可以分为：

* hinge损失：$\max(0,1-z)$
* 指数损失：$\exp(-z)$
* 对数损失：$\log(1+\exp(-z))$

常数C值：

* C值越大，惩罚越高，越有可能过拟合

## 五 CNN和self-attention

参考：

[CNN卷积算法直观理解](https://github.com/vdumoulin/conv_arithmetic)

<img src="https://i.loli.net/2021/10/26/fHXaykAqB1YO97w.png" alt="image-20211014175338105" style="zoom: 80%;" />

### 卷积核

<img src="https://github.com/vdumoulin/conv_arithmetic/raw/master/gif/same_padding_no_strides.gif" style="zoom: 50%;" />

相比于全连接网络有优化的地方：

1. 有固定大小的接收域(3X3, 5X5, 7X7)等，神经元只需要看某个区域信息。
2. 每个神经元之间进行参数共享

<img src="https://i.loli.net/2021/10/26/t7uVsgjO2KqJTEi.png" alt="image-20211014174214195" style="zoom:50%;" />

### 池化

池化有很多种：最大池化，最小池化，平均池化等等

比如最大池化：


<center><img src="https://i.loli.net/2021/10/26/Pxp7uZ6o2AbDE8n.png" alt="image-20211014175050427" style="zoom:50%;" /><img src="https://i.loli.net/2021/10/26/ABCg1JicSqNLlnb.png" alt="image-20211014175106671" style="zoom:50%;" /></center>

往往卷积和池化相互搭配使用。

### 自注意力机制

> 解决的问题：输入数据大小长度每次都有可能不一样

🔵Sequence Labeling:

有n个输入数据，并且输出n个结果，并且n个数据之间存在某种关系，上下文关系。

<img src="https://i.loli.net/2021/10/26/ai2JxrG8kMfw9T4.png" alt="image-20211014191729389" style="zoom:50%;" />

Self-attention层可以叠加多次。相关Paper：[Attention is all you need.](https://arxiv.org/abs/1706.03762)

🔵自注意力如何从输入处理到输出：

<img src="https://i.loli.net/2021/10/26/FTetB7C65hq2cfk.png" alt="image-20211017155151714" style="zoom:80%;" />

在对于 $a^1$输出$b^1$的时候需要计算$a^1$与其他$a^n$之间的关系。

> $W^q, W^k, W^v$，是需要学习的参数

方法之点乘法：
$$
q=a^1\cdot W^q\\
k=a^2\cdot W^k\\
\alpha=q\cdot k
$$
这个$\alpha$也叫注意力分数（Attention Score）

<img src="https://i.loli.net/2021/10/26/3BPGQTjFwpVE1u5.png" alt="image-20211017155958625" style="zoom:50%;" />

在求$b^1$的时候，需要对所有的输入$a^m,m=(1,..,n)$都乘以$W^k$矩阵，$a^1$也需要乘得到$k^m$，并且乘以$W^q$矩阵得到query矩阵$q^1$。

接着求解对应的注意力分数（Attention Score）,$\alpha_{1,m}=q^1\cdot k^m, m = (1,...,n)$，然后对$\alpha_{1,m}$进行SoftMax处理得到$\alpha'_{1,m}$。

<img src="https://i.loli.net/2021/10/26/CWE4qKHFiwjOogJ.png" alt="image-20211017160518350" style="zoom:50%;" />

再令$v^m=W^va^m,m=(1,...,n)$，然后进行逐项相加得到$b^1=\displaystyle{\sum_i} a'_{1,i}v^i$。

### 多注意力

> multi-head self-attention. 其中head count也是HP

$q$由于工程中需要关注的因素不止一个，需要关注多方面的因素才能够让模型更优。

![image-20211017163431071](https://i.loli.net/2021/10/26/M67axv8OwXCnKSI.png)

将每个$q,k,v$分为多个head，各个head之间内部进行处理。

<h4>位置编码（positional encoding）</h4>

对于多个head之间并没有位置之间的关联，每个位置都添加一个特定的位置向量。

<img src="https://i.loli.net/2021/10/26/HtEP69XxdojvwlD.png" alt="image-20211017163754270" style="zoom:67%;" />

### CNN与自注意力机制的关系

参考：[On the Relationship between Self-Attention and Convolutional Layers](https://arxiv.org/abs/1911.03584)

CNN可以看作一个简化版的Self-Attention

CNN只考虑感知野中的数据。

<img src="https://i.loli.net/2021/10/26/Or5Hs9U41FtKJ2E.png" alt="image-20211017165120993" style="zoom:50%;" />

## 六 transformer

sequence to sequence. 输出的结果有模型决定。
