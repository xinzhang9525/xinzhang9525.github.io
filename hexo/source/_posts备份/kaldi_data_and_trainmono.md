---
title: Kaldi 数据准备
excerpt: kaldi; 语音识别;
index_img: /img/post/answer/lm/lm-smoothing.jpeg
date: 2021-12-07 15:30:00
categories: [语音识别]
comments: true
math: true
---
### Kaldi 数据准备及monophone 训练

### 1. 数据准备

数据准备这块涉及的脚本很杂且每一个样例的脚本都区别很大，但本质上每一个任务需要准备的数据类似

/mnt/users/xin_zhang/github_project/kaldi/egs/digits

#### 1.1 音频及文本表单数据

这些数据通常放在data/train 目录下

##### 1.1.1 必要文件

最重要的文件，也就是每一个项目都必须准备的数据，也是需要自己创建的文件包括:  

 **text**

text文件包含每个utterance对应的transcript。第一列是utterance-id，后面是**分词**后的transcript。utterance-id可以是任意字符串，但是要求唯一。但如果需要告诉kaldi这句话是哪个speaker说的，通常的约定是把speaker-id作为utterance-id的前缀。

```
[xin_zhang@unn01 train] head -5 text
1-040 zero four zero
1-089 zero eight nine
1-104 one zero four
1-231 two three one
1-327 three two seven
```

**wav.scp**   

这个文件告诉kaldi每个utterance-id对应的录音文件的位置。这里是非常简单的方法，用空格分开的两列。第一列是utterance-id(或者recording-id，参考segments)，第二列是录音文件的路径。我们这里的例子第二列是录音文件的路径，但是kaldi实际要求的是extended-filename，这个extended-filename可以是一个wav文件，但是也可以是一个命令，这个命令能产生一个wav文件(用管道符 “|” 进行处理)。

```
[xin_zhang@unn01 train] head -3 wav.scp
1-040 /mnt/users/xin_zhang/github_project/kaldi/egs/digits/data/numbers-en/train/1/1-040.wav
1-089 /mnt/users/xin_zhang/github_project/kaldi/egs/digits/data/numbers-en/train/1/1-089.wav
1-104 /mnt/users/xin_zhang/github_project/kaldi/egs/digits/data/numbers-en/train/1/1-104.wav

```

**utt2spk**

这个文件告诉kaldi，utterance-id对应哪个speaker-id。speaker-id不一定要求准确的对应到某个人，只要大致准确就行。如果我们不知道说话人是谁，那么我们可以把speaker-id设置成utterance-id，这是比较安全的做法。一种常见的错误做法是把未知说话人的句子都对应到一个“全局”的speaker，这样的坏处是它会使得cepstral mean normalization变得无效。

```
[xin_zhang@unn01 train] head -3 utt2spk
1-040 1
1-089 1
1-104 1
```

**spk2utt** 

这个文件和utt2spk反过来，它存储的是speaker和utterance的对应关系，这是一对多的关系，可以使用脚本来得到：

```
utils/utt2spk_to_spk2utt.pl data/train/utt2spk > data/train/spk2utt
```

##### 1.1.2 非必需文件

**segments**

这个文件对应的是wav.scp

```
s5# head -3 data/train/segments
sw02001-A_000098-001156 sw02001-A 0.98 11.56
sw02001-A_001980-002131 sw02001-A 19.8 21.31
sw02001-A_002736-002893 sw02001-A 27.36 28.93
```

这个文件的每行为4列，第1列是utterance-id，第2列是recording-id。什么是recording-id？如果有segments文件，那么wav.scp第一列就不是utterance-id而变成recording-id了。recording-id表示一个录音文件的id，而一个录音文件可能包含多个utterance。因此第3列和第4列告诉kaldi这个utterance-id在录音文件中开始和介绍的实际。比如上面的例子，utterance “sw02001-A_000098-001156”在sw02001-A这个录音文件里，开始和结束时间分别是0.98秒到11.56秒。

**spk2gender**

这个文件不是必须的，它说明说话人的性别。

```
[xin_zhang@unn01 train] head -3 spk2gender
1 m
2 m
3 m
```



#### 1.2. 语言模型及词典数据

##### 1.2.1 lang

一般最终使用的词典数据都放置于data/lang 路径下：

```
[xin_zhang@unn01 digits] ls data/lang
G.fst  L_disambig.fst  L.fst  oov.int  oov.txt  phones  phones.txt  tmp  topo  words.txt
```

lang目录看起来很简单，没有几个文件，但是不要被表象迷惑了，因为phones是一个目录：

```
[xin_zhang@unn01 lang] ls phones
align_lexicon.int  context_indep.int  disambig.int         extra_questions.txt  nonsilence.txt        optional_silence.txt  sets.int     silence.int           wdisambig.txt        word_boundary.txt
align_lexicon.txt  context_indep.txt  disambig.txt         nonsilence.csl       optional_silence.csl  roots.int             sets.txt     silence.txt           wdisambig_words.int
context_indep.csl  disambig.csl       extra_questions.int  nonsilence.int       optional_silence.int  roots.txt             silence.csl  wdisambig_phones.int  word_boundary.int

```

phones子目录包括phone set的很多信息。它下面有一些同名但是不同后缀的文件，有3种后缀：txt、int和csl。其中txt是人可读的格式。我们不需要自己来生成这些文件，Kaldi提供了一个通用的utils/prepare_lang.sh，只需要我们为这个脚本提供一些参数。在介绍这个脚本之前，我们先了解一下lang下面文件的内容和格式。了解了它们的格式之后，我们再来介绍怎么使用prepare_lang.sh来快速的生成这个目录的内容。

**lang目录的内容**

phones.txt和words.txt，这两个文件都是符号表(符号到ID的映射)，这是OpenFst的格式。

```
[xin_zhang@unn01 lang] head phones.txt
<eps> 0
sil 1
sil_B 2
sil_E 3
sil_I 4
[xin_zhang@unn01 lang] head words.txt
<eps> 0
!SIL 1
<UNK> 2
eight 3
five 4
```

L.fst是lexicon的WFST格式，L_disambig.fst引入了#1、#2等消歧符号并且增加了输入为#0的self-loop

G.fst是语言模型的FST，只有在测试时才需要.

oov.txt里是未登录词(Out of Vocabulary Words)。

```
[xin_zhang@unn01 lang] cat oov.txt
<UNK>
```

这里把所有未登录词映射成一个特殊词。比较重要的一点是这个词通常只有一个特殊的phone，而且这个特殊的phone用来对应所有的"garbage phone"，比如把各种噪声、特殊声音都用这个phone来表示。我们可以看一下它对应的phone：

```
[xin_zhang@unn01 lang] grep -w UNK data/local/dict/lexicon.txt
<UNK> spn
```



topo 这个文件是HMM的拓扑结构定义

```
[xin_zhang@unn01 lang] cat topo
<Topology>
<TopologyEntry>
<ForPhones>
11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90
</ForPhones>
<State> 0 <PdfClass> 0 <Transition> 0 0.75 <Transition> 1 0.25 </State>
<State> 1 <PdfClass> 1 <Transition> 1 0.75 <Transition> 2 0.25 </State>
<State> 2 <PdfClass> 2 <Transition> 2 0.75 <Transition> 3 0.25 </State>
<State> 3 </State>
</TopologyEntry>
<TopologyEntry>
<ForPhones>
1 2 3 4 5 6 7 8 9 10
</ForPhones>
<State> 0 <PdfClass> 0 <Transition> 0 0.25 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 </State>
<State> 1 <PdfClass> 1 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 <Transition> 4 0.25 </State>
<State> 2 <PdfClass> 2 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 <Transition> 4 0.25 </State>
<State> 3 <PdfClass> 3 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 <Transition> 4 0.25 </State>
<State> 4 <PdfClass> 4 <Transition> 4 0.75 <Transition> 5 0.25 </State>
<State> 5 </State>
</TopologyEntry>
</Topology>

```

phone 1-10是各种静音和噪声，普通的phone通常都是3状态的，而silence这里是5状态的，因为silence通常更长。这么多phone的原因是word-position的依赖。

```
ah_B 11
ah_E 12
ah_I 13
ah_S 14
```

第一个是ah_B表示处于词开始的ah，而ah_E表示它是一个词的最后一个音素。如果一个词由三个以上音素组成，那么中间的都是用_I表示。如果一个词只有一个音素，那么用 _S表示，这种表示方法在NLP的序列标注里也非常常见。

**data/lang/phones**

这个目录下的很多文件都是定义不同的phone set，它们通常包含三种格式且包含相同的信息，下面只介绍txt格式的文件：

**context_indep.txt**

```
[xin_zhang@unn01 phones] head context_indep.txt
sil
sil_B
sil_E
sil_I
sil_S
```

这个文件的内容是上下文无关的phone，对于这些phone，我们构建的决策树不会问关于它左边和右边上下文的问题。事实上，我们会构建一个很小的决策树，它只会问central phone和HMM状态(它是第几个状态)的问题。当然这依赖于”roots.txt”文件，下面我们会介绍这个文件。关于决策树的更多内容可参考[How decision trees are used in Kaldi](http://kaldi-asr.org/doc/tree_externals.html)

这些phone由于word-position的依赖有很多的变体；但并非每种变体都会用到。这里的SIL是可选的silence(在两个词之间可能会有静音)，SIL_B是作为一个词的开始(这是不可能的)，SIL_I是指一个词的中间有静音(这也不太可能存在)，SIL_E是一个词结尾的静音(这也不可能存在)，而SIL_S表示某个词只有这一个静音的phone。当我们的transcript显式的标注出静音的时候，我们可能定义一个叫”静音”的词的时候，我们可以用SIL_S来作为这个词的发音。当然通常很少会碰到这种情况，所以除了SIL，其它的SIL_*很少会出现。

**silence.txt和nonsilence.txt**

这两个文件分别包含silence的phone和非silence的phone，并且它们是没有交集的，而且它们的并集是所有的phone。在我们这个例子里，silence.txt和context_indep.txt是恰巧一样的。所谓的非silence的phone，我们认为是可以对它进行各种线性变换的phone。比如全局的LDA和MLLT；或者说话人相关的自适应变换比如fMLLR。根据我们的经验，silence是不值得做这些变换的(比如我们认为不同人说同一个”a”是有差别的，但是我们认为静音对于所有人都是一样的，比如背景噪音是没有必要根据说话人做什么变换的)。我们的实际经验是把所有的silence、背景噪声和发音噪声都作为”silence”的phone，而其它”真正”的音素当成非silence phone。

```
[xin_zhang@unn01 phones] head -3 nonsilence.txt silence.txt
==> nonsilence.txt <==
ah_B
ah_E
ah_I

==> silence.txt <==
sil
sil_B
sil_E
```

**disambig.txt**

这个文件包含所有的消岐符号，消歧符号出现了phones.txt里，我们把它当成phone。关于消歧符号请参考http://kaldi-asr.org/doc/graph.html#graph_disambig。

```
[xin_zhang@unn01 phones] cat disambig.txt
#0
#1
```

**optional_silence.txt**

可选的silence，在两个词之间可能会有silence，因此需要定义可选的silence。

```
[xin_zhang@unn01 phones] cat optional_silence.txt
sil
```

通过在lexicon FST的每个词的结尾(也包括一个utterance的开始)增加一个可选的SIL，我们可以实现这两个词之间的可选的经验(可选的元音是两个词说得很快中间就没有静音了)。为什么在phones里指明而不只是出现了L.fst里的原因是比较复杂的，我们这里不深入讨论。



**sets.txt**

这个文件的每一行是一个phone集合，把phone聚类在一起的目的是用于创建上下文相关的问题。在Kaldi里构建决策树时我们并不使用语言学家定义的问题，而是自动聚类出来的问题，所谓的一个问题其实就是一个phone的集合，可以参考[Kaldi教程 monophone模型训练](http://fancyerii.github.io/kaldidoc/tutorial2monophone模型训练)。

在这里，sets.txt的每一行就是一个phone的所有word-postion的依赖的变体：

```
[xin_zhang@unn01 phones] head sets.txt
sil sil_B sil_E sil_I sil_S
spn spn_B spn_E spn_I spn_S
ah_B ah_E ah_I ah_S
ao_B ao_E ao_I ao_S
ay_B ay_E ay_I ay_S
```

**extra_questions.txt**

除了自动聚类的问题，我们也可以通过这个文件提供决策树聚类的问题。

```
[xin_zhang@unn01 phones] cat extra_questions.txt
ah_B ao_B ay_B eh_B ey_B f_B hh_B ih_B iy_B k_B n_B ow_B r_B s_B t_B th_B uw_B w_B v_B z_B
ah_E ao_E ay_E eh_E ey_E f_E hh_E ih_E iy_E k_E n_E ow_E r_E s_E t_E th_E uw_E w_E v_E z_E
ah_I ao_I ay_I eh_I ey_I f_I hh_I ih_I iy_I k_I n_I ow_I r_I s_I t_I th_I uw_I w_I v_I z_I
ah_S ao_S ay_S eh_S ey_S f_S hh_S ih_S iy_S k_S n_S ow_S r_S s_S t_S th_S uw_S w_S v_S z_S
sil spn
sil_B spn_B
sil_E spn_E
sil_I spn_I
sil_S spn_S
```

前面的4个问题(也就是4行)是对于普通的phone的位置的问题，比如第一行的问题用自然语言描述就是：”请问这个phone是不是在词开始的位置？”

第五个问题是：”这个phone是不是silence？” 而后面的问题是问silence phone的位置问题。

**word_boundary.txt**

这个文件指明每个phone的”边界”信息。比如SIL是nonword，而SPN_B是位于词的开始。

```
[xin_zhang@unn01 phones] head word_boundary.txt
sil nonword
sil_B begin
sil_E end
sil_I internal
sil_S singleton
```

**roots.txt**

```
[xin_zhang@unn01 phones] head roots.txt
shared split sil sil_B sil_E sil_I sil_S
shared split spn spn_B spn_E spn_I spn_S
shared split ah_B ah_E ah_I ah_S
shared split ao_B ao_E ao_I ao_S
shared split ay_B ay_E ay_I ay_S
shared split eh_B eh_E eh_I eh_S
```

这个文件说明怎么对决策树进行聚类。

在这里，我们暂时先忽略shared和split——这是构建决策树的特殊选项(更多信息请参考[How decision trees are used in Kaldi](http://kaldi-asr.org/doc/tree_externals.html))。这里我们我们重点关注一行里的那些phone，比如SIL SIL_B SIL_E SIL_I SIL_S，它是决策树的树根。也就是说每一行是一个决策树，不同行的phone是不可能聚类到一起的。对于有重音和带调的语言，我们通常把它们放到一行。我们通常把同一个音素的不同位置变体都放在一起。此外同一个HMM的三个不同状态(silence通常是五个状态)也放到一棵树的树根来聚类，当然Kaldi的问题会问它是第几个状态。这一点是和其它的系统不同的——在那里，比如一棵树的树根就是/iy/的中间(第二个)状态。而Kaldi里是把/iy/的三个状态放到一棵树下的，因此有可能b-iy+t的第一个状态和b-iy+d的第二个状态聚到一个叶子上，但是在其它系统是不可能的。



##### 1.2.2 创建lang目录

Kaldi提供了一个工具来自动的生成lang目录下这些文件:

```
utils/prepare_lang.sh data/local/dict "<UNK>" data/local/lang data/lang
```

第一个参数是data/local/dict/，这是需要我们提提取准备好的目录, 第二个参数指定OOV对应到哪个词，这里是。我们前面的oov.txt就是根据这个参数生成的。第三个参数data/local/lang是临时的输出目录。第四个参数data/lang就是最终的输出。

所以最终我们需要准备的数据是data/local/dict/：

```
[xin_zhang@unn01 digits] ls data/local/dict/
lexiconp.txt  lexicon.txt  nonsilence_phones.txt  optional_silence.txt  silence_phones.txt
```

这两个文件定义了所有的phones

```
[xin_zhang@unn01 dict] head -3 nonsilence_phones.txt
ah
ao
ay
[xin_zhang@unn01 dict] head -3 silence_phones.txt
sil
spn
```

lexicon.txt是发音词典：

```
[xin_zhang@unn01 dict] head lexicon.txt
!SIL sil
<UNK> spn
eight ey t
five f ay v
four f ao r
```

它告诉我们每个词是由哪些phone组成，我们也可以提供一个lexiconp.txt。lexiconp.txt比lexicon.txt多一列，它的第二列是一个概率值。因为一个词可能有多种发音，但是它们的概率是不同的，所以我们可以指定概率。我们可以把lexicon.txt看成概率都是1.0。有些认为需要把概率归一化，也就是如果一个词有两个发音，那么这两个概率加起来是一。但是最佳实践并不是这样，最好的做法是让两个发音里最可能的值为1.0，而另外一个根据它和最可能的比例设置。因此这样两个值加起来是大于1的。

```
[xin_zhang@unn01 dict] head lexiconp.txt
!SIL 1.0        sil
<UNK> 1.0       spn
eight 1.0       ey t
five 1.0        f ay v
four 1.0        f ao r
```

utils/prepare_lang.sh脚本有很多参数：

```
usage: utils/prepare_lang.sh <dict-src-dir> <oov-dict-entry> <tmp-dir> <lang-dir>
e.g.: utils/prepare_lang.sh data/local/dict <SPOKEN_NOISE> data/local/lang data/lang
options:
     --num-sil-states <number of states>             # default: 5, #states in silence models.
     --num-nonsil-states <number of states>          # default: 3, #states in non-silence models.
     --position-dependent-phones (true|false)        # default: true; if true, use _B, _E, _S & _I
                                                     # markers on phones to indicate word-internal positions.
     --share-silence-phones (true|false)             # default: false; if true, share pdfs of
                                                     # all non-silence phones.
     --sil-prob <probability of silence>             # default: 0.5 [must have 0 < silprob < 1]
```



#### 1.3 特征提取

基于1.1 中准备好的音频表单数据，就可以用来提取训练模型所需的特征：

















































 

