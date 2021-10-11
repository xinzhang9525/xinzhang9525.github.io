---
title: FlowScope:Spotting Money Laundering Based on Graphs
excerpt: Knowledge Graph; Embedding;
index_img: /img/post/answer/lm/lm-smoothing.jpeg
date: 2021-09-16 16:40:00
categories: [Graph Embedding]
comments: true
math: true
---

# 一、总览

顾名思义，原文旨在通过图相关方法，发现洗钱行为。洗钱行为表示犯罪分子通过银行将大量的非法所得转移至无法追溯的终点，而不被发现。现有的关于反欺诈的图相关方法主要关注紧密子图的发现，而不考虑洗钱过程中存在大量金钱转移的事实，从而降低了探查准确率。原文提出的FlowScope 方法不仅可以量化整个洗钱过程，而且从理论层面可以提出了正常交易中不被探查的金额上限。

原文地址：http://www.shichuan.org/doc/78.pdf

源码地址：https://github.com/aplaceof/FlowScope

# 二、简介与相关工作

在已有的很多反洗钱算法中，均有不同程度的缺陷，其中比较典型的问题包括：

1. 忽略金钱转移链的模式，同时忽略交易过程中的复杂依赖，特别是节点间的复杂资金往来记录，从而会降低准确率。
2. 有些考虑金钱转移链的模式的，需要用监督学习的方案进行学习，而反洗钱场景中，有效标签量非常难以获取。同时，在很多通过对抗网络获得的模型，因为样本的严重不平衡，可能存在模型健壮性欠佳的情况。
3. 部分以业务经验为基础的算法，虽然短期有效，但容易被违法者攻破。

而FlowScope主要包含以下优势：

1. 提出一种新的洗钱发现方法，这种方法可以很好的发现紧密交易和多步骤洗钱的信息。
2. 严谨的理论证明，不仅证明此方法的有效性，同时可以给出犯罪分子在该模型下可以转出资金而不被发现的上限。
3. 高效性和健壮性。
4. 可扩展性，同时在上面的github地址给出扩展样例。

# 三、问题定义

在定义问题前，我们阐述洗钱过程中的两个重要特点（从我个人工作经历来看，对下面两点高度赞同）：

1. 密集的转账记录：参与洗钱卡，会产生大量的转入转出记录，并且会总的转入转出金额很大。
2. 中间账户的余额通常很小：中间账户一般作为桥梁，不会存放资金。（个人经验：在个别情况下，还存在转入后快速转出，中间停留时间在数分钟到数小时不等。）

此方法重点关注这类型的账号。

核心问题可以定义为：

在一个资金流水图中$\mathcal{G} = (\mathcal{V}, \mathcal{E})$，其中卡号作为节点$\mathcal{V}$，转移资金量作为边$\mathcal{E}$。那么目的可以是找到一个$\mathcal{G}$的子图，并满足以下条件：

1. 交易流水在转入这批中间账户和转出这批中间账户的过程中有很大量。
2. 由下面的定义的新标准，保证资金流动量在子图中最大。那么定义的ML metric则即为重要。

下图展示在后续解释中可能出现的重要符号：

![11111](G:\work\模型中心\19. X_lab\自己读的内容\17_FLOWSCOPE\11111.jpg)

通常，图$\mathcal{G} = (\mathcal{V}, \mathcal{E})​$中的节点$\mathcal{V} = \mathcal{X} \bigcup \mathcal{W} \bigcup \mathcal{Y}​$，其中$\mathcal{W}​$表示洗钱桥梁，$\mathcal{X}​$和$\mathcal{Y}​$分别代表金钱来源（整体情况为转入中间层）和资金去向（整体情况为从中间层转出）的卡号。一条边$(i,j)\in \mathcal{E}​$表示在节点集合$\mathcal{V}中，​$卡号$v_i​$转至卡号$v_j​$的资金量$e_{ij}​$。

# 四、核心

这部分主要分三个小点，阅读建议着重理解ML metric。只有足够好的理解第一部分，才能更好的理解第二部分。

## 1. ML metric

定义ML metric的目的是发现紧密资金转移的现象。

定义节点子集$\mathcal{S} = \mathcal{A} \bigcup \mathcal{M_1} \bigcup ... \bigcup \mathcal{M_{k-2} \bigcup \mathcal{C}}$，其中$\mathcal{A} \subseteq \mathcal{X}, \mathcal{M_i} \subseteq \mathcal{W}, \mathcal{C} \subseteq \mathcal{Y}$，其中在$\mathcal{M_1}, ... , \mathcal{M_{k-2}}$中可能存在节点重合的情况。但在实际情况中，如果资金链条形成闭环更容易被执法者发现，因此我们主动忽略这样的影响。在上面的式子中，我们可以认为总共有$k$层数据，其中第0层为$\mathcal{X}$，第$k$层为$\mathcal{Y}$。

同时定义$e_{ij}​$表示为从节点$v_i​$转至节点$v_j​$的资金总量。那么对于节点$v_i \in \mathcal{M_l}, l\in {1, 2, ... , k-2}​$，定义在子集$\mathcal{S}​$总的资金转出量与资金转入量分别为：
$$
d^+_i(s) = \sum_{v_j\in M_{l+1}\wedge(i,j)\in \mathcal{E}} e_{ij}
\\
d^-_i(s) = \sum_{v_k\in M_{l-1}\wedge(k,i)\in \mathcal{E}} e_{ki}
$$
下面定义在子集$\mathcal{S}$下资金流转的最小，最大量分别为：
$$
f_i(s) = \min\{d^+_i(S), d^-_i(S)\}, \forall v_i \in \mathcal{M_l}
\\
q_i(s) = \max\{d^+_i(S), d^-_i(S)\}, \forall v_i \in \mathcal{M_l}
$$
那么，则可以定义ML metric为：
$$
g^k(\mathcal{S}) = \frac{1}{|\mathcal{S}|}\sum^{k-2}_{l=1}\sum_{v_i\in \mathcal{M_l}} f_i(\mathcal{S}) - \lambda (q_i(\mathcal{S}) - f_i(\mathcal{S}))
\\
= \frac{1}{|\mathcal{S}|}\sum^{k-2}_{l=1}\sum_{v_i\in \mathcal{M_l}} (1+\lambda)f_i(\mathcal{S}) - \lambda q_i(\mathcal{S}), k\geq 3
$$
有定义可知$q_i(\mathcal{S}) - f_i(\mathcal{S})$是剩余在$v_i$卡号上的余额，这个可以被认为是针对ML过程中桥梁卡号通常余额很小的惩罚项。对上式中的$\lambda$可以定义为转账过程中必须产生的手续费等开销。这样我们定义的$g(\mathcal{S})$则可以认为是洗钱后的剩余利润。

![2222](G:\work\模型中心\19. X_lab\自己读的内容\17_FLOWSCOPE\2222.jpg)

举个例子，上图中表示的是一个ML过程。A（资金流入卡号），M（桥梁卡号），C（资金流出卡号）分别有4,12,2个卡号。$v_5$转移了将近452.1M元，同时$q_5(\mathcal{S}) - f_5(\mathcal{S}) \approx 0$，这就表示此卡有大量的流水转入转出，并且卡账剩余金额很少。ML metric 可以很好的捕捉到这些特点。

## 2. FlowScope

FlowScope的提出，目的在于找到一个$\mathcal{S}$使得我们的目标$g(\mathcal{S})$最大化。先制作一个优先树，那么对于任意节点$v_i$存在的权重定义如下：
$$
w_i(\mathcal{S}) = \{\begin{array}{cols} 
					f_i(\mathcal{S}) - \frac{\lambda}{1+\lambda}q_i(\mathcal{S}),  & \mbox{if} & v_i \in \mathcal{M_l}
					\\
					d_i(\mathcal{S}), & \mbox{if} & v_i \in \mathcal{A} \cup \mathcal{C}
					\end{array}
$$
那么下面则可以给出相应的贪婪优化方法：

1. 确定图。
2. 从现有图中移除有最小权重的点（为了最大化我们的目标）。
3. 更新图中受影响节点的权重，并计算当前图下的目标$g(\mathcal{S})$。
4. 重复2-3过程，直至A，M，C中任意一个集合的节点全部被移除。
5. 得到最大的$g(\hat{\mathcal{S}})$，同时可以找到对应子图$\hat{\mathcal{S}}$。

![33333](G:\work\模型中心\19. X_lab\自己读的内容\17_FLOWSCOPE\33333.jpg)

截止此处，原文中最核心的部分已经介绍完成。我个人对以下两个部分有一点疑问：

1. 这里有一个比较讨厌的部分是对桥梁节点的“层级”进行定义或者先验已知。这部分内容很讨厌的原因是，我们不仅不太好区分到底作为桥梁的卡有多少个“层级”，同时难以区分到底哪张卡应该隶属于哪个层级也尝尝在实际情况中遇到困难。
2. 在贪婪算法部分，停止条件是当一个节点集合为空则停止搜索。这部分内容从直觉上说可能会有点问题。在实现的时候会做一些测试。

## 3. 分析部分

### 3.1 Theoretical Bound

给定图和metric ML定义。那么对于任意子图，存在：
$$
g(\hat{\mathcal{S}}) \geq \frac{|\mathcal{M^\prime}|}{|\mathcal{S^\prime}|}(g(\mathcal{S}^\star) - \lambda \epsilon)
$$
其中$\epsilon = \max_{v_i \in \mathcal{S}^\star} \{q_i(\mathcal{V}) - q_i(\mathcal{S^\star})\}​$，表示为在该子图内转移资金量最多的账户接收转出到其他非洗钱账户的金额。

### 3.2  Bounding Money Laundering

给定子图，该子图每个账号进行洗钱行为但不被发现的资金量上限为：
$$
\frac{\sum_{v_i \in \mathcal{S}^\star}f_i(\mathcal{S}^\star)}{n_0} \leq \frac{1}{1-\lambda \eta}(\frac{|\mathcal{S}^\prime|}{|\mathcal{M}^\prime|}g(\hat{\mathcal{S}}) + \lambda \epsilon)
$$
其中新出现的符号中，$n_0​$表示该子图的卡号数量，$\eta​$表示可能存在的其他费用。其定义为：
$$
\eta = \frac{\sum_{v_i \in \mathcal{S}^\star}(q_i(\mathcal{S}^\star) - f_i(\mathcal{S}^\star))}{\sum_{v_i \in \mathcal{S}^\star}f_i(\mathcal{S}^\star)}, \eta \in [0, \frac{1}{\lambda}]
$$
这两个部分的内容可以作为排查过程中的其他评价指标进行输出。

# 五、实验

实验数据：

1. CBANK，匿名银行真实数据。
2. CFD（Czech Financial Dataset），另一个匿名银行数据，曾经用于比赛。

对比方法SpokEN，Fraudar，D-Cube，HoloScope，RRCF。评价方法选择FAUC与F1两个评价指标。并主要点评工作效率，模型效果，健壮性。测试效果见图。

![4444](G:\work\模型中心\19. X_lab\自己读的内容\17_FLOWSCOPE\4444.jpg)

核心结论如下：

1. 相比其他方法，此方法能够更快、更准确的发现洗钱行为。模型计算时间随图内边数量增加称线性关系。
2. 在面对更长的资金转移链条时，表现的更稳定更健壮。
3. 模型对相关先验参数不敏感，如：$\lambda$。

# 六、总结

原文提出了一种评价洗钱行文的量化指标，并且求解思路清晰，可执行性强。建议阅读内容如下：

> Guha,S.;Mishra,N.;Roy,G.;andSchrijvers,O. 2016. Robustrandomconferenceutforestbasedanomalydetectionon streams. In International conference on machine learning, 2712–2721.

> Hooi, B.; Song, H. A.; Beutel, A.; Shah, N.; Shin, K.; and Faloutsos, C. 2016. Fraudar: Bounding graph fraud in the face of camouﬂage. In Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining, 895–904. ACM.

> Liu, S.; Hooi, B.; and Faloutsos, C. 2017. Holoscope: Topology-and-spike aware fraud detection. In Proceedings
> ofthe2017ACMonConferenceonInformationandKnowledge Management, 1539–1548. ACM.

> Liu, S.; Hooi, B.; and Faloutsos, C. 2018. A contrast metric for fraud detection in rich graphs. IEEE Transactions on Knowledge and Data Engineering.

> Prakash, B. A.; Sridharan, A.; Seshadri, M.; Machiraju, S.; and Faloutsos, C. 2010. Eigenspokes: Surprising patterns andscalablecommunitychippinginlargegraphs. InPaciﬁcAsiaConferenceonKnowledgeDiscoveryandDataMining, 435–448. Springer.

> Shin, K.; Hooi, B.; Kim, J.; and Faloutsos, C. 2017. Dcube: Dense-block detection in terabyte-scale tensors. In Proceedings of the Tenth ACM International Conference on Web Search and Data Mining, 681–689. ACM.

