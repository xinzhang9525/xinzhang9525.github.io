---
title: Logic Attention Based Neighborhood Aggregation for Inductive Knowledge Graph Embedding
excerpt: Knowledge Graph; Embedding;
index_img: /img/post/answer/lm/lm-smoothing.jpeg
date: 2021-09-16 16:00:00
categories: [Graph Embedding]
comments: true
math: true
---
### Logic Attention Based Neighborhood Aggregation for Inductive Knowledge Graph Embedding

1. 背景

   基于知识图谱$G=(V, E)$, 如何进行知识的推理是十分重要的任务。假设$(s, o, r)$表示图谱中的一条边（发生的事件），其中s,o代表节点，r是关系类型，那么知识推理的一个例子就是：在已知s和r的情况下，推理出o是哪个节点。同理，在缺失s的情况下，预测s是哪个节点。本文没有考虑时间的因素，如果考虑$(s,o,r,t)$, 其中t是代表该事件发生的时间，更强大的知识图谱推理，具有补全实体或者时间的能力。

   实现推理的一类工具是节点的编码。过去对于静态图的编码取得了一定的效果，如deep_walk, node2vec, Line等，但是这些编码方式，要求所有的节点都出现在图谱中；而现实中，随着时间推移，一定会有之前没包含在图谱的节点出现。面对这种情况，过去的静态编码方法，只能纳入新的数据，以全量数据重新训练模型。这在工程应用中，不仅计算代价可能较大，并且频繁的更新模型，会造成模型结果的不稳定。

   本文的编码方法：具有推理能力，且可以处理新增节点的快速编码(out of sample).

2. 动机

   （1）当图编码过程完成后，快速的给新出现的节点进行编码。在训练编码的过程中，采用aggregator的方式，利用相邻节点以及涉及到的关系类型，对节点进行编码。该框架没有downstream任务，可以无监督的训练。

   （2）本文主要是讲述构造aggregator的方式，其中作者想要构造的聚合器有如下性质:

   ​	· 聚合时，邻居节点是不应该考虑顺序的。例如简单的mean-pooling是可以的，而LSTM是不合适的，这一点应该根据实际问题出发，酌情考虑。作者的出发点是，当新增节点(emerging entity)出现时，节点Chicago_Bulls 和 American的重要性跟他们在邻居序列中的位置无关。

   ​	· 整体上来说，各类关系之间存在信息的包含，于是在聚合时，aggregator要有识别冗余信息(play_for 包含work_as的信息)以及关注重点关系（通常我们需要索引 live_in的时候，需要关注到与其相关的信息 e.g.play_for Chicago_Bulls）的机制。

   ​	<img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200402143903095.png" alt="image-20200402143903095" style="zoom:67%;" />

   

3. 框架

   2.1 主要符号

   $\varepsilon$ 和 $\mathcal{R}$表示节点和边的集合，数量分别为n和m. 知识图谱由一系列的三元组组成：
   $$
   \mathcal{K} = \{(s,r,o)|s\in \varepsilon , r\in \mathcal{R},o\in\varepsilon \}
   $$

   对任意$\mathcal{K}$中的三元组(s,r,o),对应的加入(o, r^{-1}, o).

   对任意的实体$e\in \varepsilon$，定义邻居（边和点的集合），及邻居在节点和关系集合的投影分别为:
   $$
   N_{\mathcal K}(e) = \{(r, e')|(e, r, e')\in \mathcal K  \}
   $$

   $$
   N_{\varepsilon}:N_{\mathcal K}(e) \rightarrow \varepsilon
   $$

   $$
   N_{\mathcal R} :N_{\mathcal K}(e) \rightarrow \mathcal R
   $$

   2.2 目标

   在给定的知识图谱上，学习一个邻居聚合器A，对一个节点$e_i \in \varepsilon$
   $$
   A: N_{\mathcal K}(e_i)\rightarrow R^d
   $$
   并且对一个未知的三元组(s,r,o)的可能性建立在A的结果上, 为了区分邻居中的关系r，位置三元组改为(s, q, o)

   <img src="C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200402152134913.png" alt="image-20200402152134913" style="zoom:50%;" />

   2.3 encoder

   如图二所示，对一个节点$e_i$（图中的乔丹大帝和芝加哥），实质是存在两套编码，即图中input embedding 和output embedding, encoder作用在$e_i$的邻居节点的输入编码上。令$e_i$的邻居$(r, e_j) \in N_{\mathcal K} (e_i)$, $e_j$的输入编码记为${e}_j^{I}$, 则特点关系的转移方程如下：
   $$
   T_r(\bold e_j^I) = \bold{e}_j^I - \bold{w}_r^T \bold{e}_j^I\bold{w}_r,
   $$
   目标$e_i$输出编码为：
   $$
   \bold{e}_i^O = A(\{(r,e_j)\in N_{\mathcal K}(i) \}).
   $$
   A的作用实质上是将一堆d维向量，聚集成一根d维向量。作为对比，可以选择简单的平均，以及复杂的LSTM.

   本文采取加权求和的方式，聚合向量:
   $$
   \bold{e}_i^O = \sum_{(r,e_j)\in N_{\mathcal K}(i)}\alpha_{j|i,q} T_r(\bold{e}_j^I)
   $$
   为了使A满足上述的性质， 需仔细的构造$\alpha_{j|i,q}$，该系数不仅与邻居节点编码有关，也与邻居关系r和需要索引的关系q有关。

   2.3.1 逻辑法则机制(logic rule mechanism)

   对一个节点e， 它的邻居关系$r_1, r_2$，可能存在信息的包含关系 ，比如play_for可能推导出live_in，记他们的潜在关联规则为$r_1 \Rightarrow r_2$，定义规则的置信度为：
   $$
   \mathcal P(r_1\Rightarrow r_2)= \frac{\sum_{e\in \varepsilon}1(r_1\in N_{\mathcal R}(e) \wedge r_2\in N_{\mathcal R}(e)) }{ \sum_{e\in \varepsilon}1(r_1\in N_{\mathcal R}(e))}.
   $$

   $$
   \alpha_{j|i,q}^{Logic} = \frac{\mathcal P(r\Rightarrow q)}{max(\{\mathcal P(r'\Rightarrow r)|r' \in N_{\mathcal R}(e_i) \wedge r'\neq r\})}
   $$

   

   2.3.2 神经网络机制(neural network mechanism)

   给定关系q，$e_i$的邻居$e_j$的重要性为：
   $$
   \alpha_{j|i,q}^{NN} = softmax(\alpha_{j|i,q}')=\frac{exp(\alpha'_{j|i,q})}{\sum_{j'\in N_{\varepsilon}(i)} exp(\alpha'_{j'|i,q})}
   $$
   其中：
   $$
   \alpha_{j|i,q}'=\bold u_a^T \cdot tanh(\bold W_a \cdot [z_q ; T_r({\bold e}_j^I)])
   $$
   ${\bold u}_a^T$, ${\bold W}_a$ 和$\bold z_q$是训练参数。

   综合2.3.1和2.3.2，：
   $$
   \bold e_i^O = \sum_{(r,e_j)\in N_{\mathcal K}(i)} (\alpha_{j|i,q}^{Logic} + \alpha_{j|i,q}^{NN}) T_r(\bold e_j^I)
   $$
   2.4 decoder

   从encoder， 给出施加方和承受方的编码${\bold s}^O$和${\bold o}^O$, decoder被用来衡量该训练三元组的合理性(plausibility)，$\bold W_r  \in R^{m\times d}$是用来做关系转移的那个矩阵见2.3第一个方程。则decoder给(s,q,o)的打分函数为:
   $$
   \phi^O(s,q,o)=-|\bold s^O + \bold q - \bold o^O|_{L1},
   $$
   2.5 目标函数

   为了训练模型，需要构造正样本和负样本，$\mathcal K$中的三元组都是正样本，记作$\Delta$, 对一个正样本(s,q,o)，随机替换s或者o，不同时替换：
   $$
   \Delta'_{(s,q,o)} = \{(s',q,o)|s'\in \varepsilon\} \cup \{(s,q,o')|o'\in \varepsilon\}
   $$

   $$
   l^O(s,q,o) = [\gamma - \phi^O(s,q,o)+\phi^O(s',q,o')]_+
   $$

   其中$[x]_+ = max(0, x)$, $\gamma$是一个超参数。于是训练的目标函数为：
   $$
   min \sum_{(s,q,o)\in\Delta}\sum_{(s',q,o')\in \Delta'_{(s,q,o)}} l^O(s,q,o).
   $$

4. 备注

   （1）本文在编码器中整合aggregator，解决了网络新增节点的问题，预计速度较快，因为只用lookup几个邻居节点和边的编码，加权求和即可；

   （2）本文是通过糅合聚合器，同时解决编码和新增节点问题，如果工程上已有取得不错效果的编码（如node2vec，line等），可能需要一个独立于编码器的组件，来处理新增节点。可见本专栏另一篇关于恶意软件的论文分享。

   （3）聚合器中的权重，既可以从神经网络中自动提取，同时也可以人为定义；作者提出的聚合器需要满足的性质，可根据应用的具体场景而定，比如邻居关系与发生的时间有紧密关系，则可加入按时间衰减的机制，或者邻居根据发生的时间有天然的序列属性，使用LSTM有可能会挖掘到高度非线性特征，取得更好的效果。

   （4）极端的情况下，新增的节点未与任何已有节点相连，模型就会失效。

   