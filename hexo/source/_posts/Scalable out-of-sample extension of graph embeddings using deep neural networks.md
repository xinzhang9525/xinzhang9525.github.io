---
title: Scalable out-of-sample extension of graph embeddings using deep neural networks
excerpt: Knowledge Graph; Embedding;
index_img: /img/post/answer/lm/lm-smoothing.jpeg
date: 2021-09-16 16:30:00
categories: [Graph Embedding]
comments: true
math: true
---
### Scalable out-of-sample extension of graph embeddings using deep neural networks

1. 背景

   对于一系列具有高维特征的样本集$X$（如图片）,如果要进行分类或聚类等任务，一个重要的处理方法是先进行数据降维，再使用分类器或聚类模型，提高精度和效率。

   谱分解问题：令$X=\{x_1, x_2,\dots,x_n \}$ 是需要进行嵌入的特征样本，其中$x_i \in R^d.$ 注意这里的d不是代表编码向量的长度，而是人工提取的特征长度。利用某个核函数$K:R^d \times R^d \rightarrow R$ ,可以定义样本X的相似矩阵A, 其中每个元素为$A_{ij} = K(x_i, x_j),$ 则A的拉普拉斯矩阵和度矩阵分别为：

$$
\mathcal L = I - D^{-1/2}A D^{-1/2},\quad  D = diag(\{\sum_{j=1}^n A_{ij}, i=1,2,\dots,n \})
$$

​		于是每个特征样本的编码转换为求解$\mathcal L$ 的特征值和特征向量的问题，即谱分解$\mathcal L = U \sum U^T$, 其中$\sum$是对角矩阵，且元素为拉普拉斯矩阵的奇异值，且按从高到低排列。取U的前$d'$列得到子矩阵$U_{d'}$，它的每个行向量记为对应样本$x_i$的$d'$维编码。

​		当现有的数据降维模型（编码模型）无法处理或者要付出很大资源的情况下，本文提出了一个深度神经网络，通过拟合谱分解，使该模型能够从节点的特征x，即可得到节点的编码。

2. 动机

   2.1 谱分解问题涉及svd算法求解，该算法耗时很多$\mathcal O (n^3)$, 则新增out of sample节点，重启一次模型的代价很高。

   2.2 对比的baseline算法，为$Nystr\ddot{o}m$拓展方法。对新增节点$x\in R^d$，第p维的拓展为
$$
y_p(x) = \frac{1}{\lambda_p} \sum_{i=1}^n v_{pi} K(x, x_i)
$$
可以看出，该方法计算量会随着训练样本个数的增多，时间复杂度线性增加。

3. 框架

本文的思路很简洁，即训练一套DNN（记作y）去逼近映射上述谱分解映射$z: X\rightarrow U_{d'}$，记$z_i=z(x_i)$表示$U_{d'}$的第 i 行向量。

   3.1 DNN（encoder + decoder）
$$
h_l = \sigma(W_l h_{l-1} + b_l), \quad l=1,2,\dots,N
$$
其中
$$
h_0 = x_i, \quad W_1\in R^{M\times d}, \quad W_l \in R^{M\times M} for l=2,3,\dots,N
$$
于是
$$
y(x) = W_{N+1} h_N + b_{N+1}, \quad 其中\quad W_{N+1}\in R^{d'\times M},b_{N+1}\in R^{d'}.
$$
训练的目标函数：
$$
\Theta^* = arg \min_{\Theta} \frac{1}{n}\sum_{i=1}^n ||z_i-y(x_i)||^2
$$
作者在训练网络的时候，为了提高训练效率和结果的精度， 使用的初始化参数技巧如下：

3.1.1 无监督预训练

该部分是为了逐层初始化encoder的参数，即$W_l,b_i\quad for\quad l=1,2,\dots, N$. 每次引入一个隐层，加上一个临时的解码层$(W_{tmp}, b_{tmp})$,  优化目标:
$$
arg\min_{W_1,b_1,W_{tmp},b_{tmp}} \frac{1}{n}\sum_{i=1}^n ||x_i - y_{tmp}(x_i)||^2
$$
其中
$$
y_{tmp}(x_i) = W_{tmp} h_1 + b_{tmp}, \quad h_1 = \sigma(W_1 h_0 + b_1)
$$
经过几轮随机梯度下降优化所有参数，即通过拟合恒等映射，初始化$W_1,b_1$. 接下来，以$h_1$为输入，引入第二个隐层，再通过拟合恒等隐射，更新第二层隐层参数，以此类推，将N层隐层参数初始化。

3.1.2 有监督的修正(fine-tune)

使用预训练的参数为初始值，优化问题$\Theta^*$. 当DNN训练完成，对out of sample $x$, 即可计算其编码$y(x)$，且该计算消耗的时间与训练样本量无关。

4. 数值结果

   ![image-20200414122510170](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200414122510170.png)

   该表格的第一列，是训练数据中的节点个数，使用baseline和DNN方法得到新增节点的编码，使用NRMSE进行衡量，值越小越精准；时间的单位是秒，用来测试的新增节点个数为40w，可以看到baseline的计算时间是线性的，本文方法计算时间是长度。

5. 备注

工程实践中，如果已有了一套取得不错效果的图编码结果，为了处理新增节点的编码，需要一套与编码方式解耦的模型，能够利用已有的编码结果处理新增节点，同时期望这个新的编码能包含在已有的编码空间。

假设已有的编码函数为$f:X\rightarrow R^{d'}$, 本文通过训练一个DNN网络$y:X\rightarrow R^{d'}$，去逼近f，如此当新的节点x到来，该DNN可以给新节点一个编码$y(x)$， 且可以保证与f对应的空间相同。

假设使用node2vec已经得到一个图$G(V, E)$中节点的编码表embedding，复用训练的语料信息context,其中context的第i行为$c_i = [i, context_{i,1}, \cdots,context_{i,2n}]$, 其中2n为上下文的长度，再训练一个DNN网络$y: R^{2nd}\rightarrow R^d$， 使得$\min \sum_i||y(embedding[context_{i,1}, \cdots, context_{i,2n}])-embedding[i]||^2$ .

