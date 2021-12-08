---
title: Kaldi 数据准备
excerpt: kaldi; 语音识别;
index_img: /img/post/answer/lm/lm-smoothing.jpeg
date: 2021-10-25 10:30:00
categories: [语音识别]
comments: true
math: true
---
### Kaldi 数据准备

#### 1. 音频及文本数据

以TTS生成的数据为例，构造了一批音频和ref的测试集:

/mnt/users/xin_zhang/github_project/kaldi/egs/smarthome/data 并以此为基础从头介绍 构建基于kaldi的ASR系统训练流程。

##### 1.1 data文件说明

###### 1.1.1 必要文件

最重要的文件，也就是每一个项目都必须准备的数据包括:  

 **text**

text文件包含每个utterance对应的transcript。第一列是utterance-id，后面是**分词**后的transcript。utterance-id可以是任意字符串，但是要求唯一。但如果需要告诉kaldi这句话是哪个speaker说的，通常的约定是把speaker-id作为utterance-id的前缀。比如ASRDP_5645是一个speaker-id，_1表示ASRDP_5645说的第一句话。

```
[xin_zhang@unn2 train] head -3 text
ASRDP_5645_1 播放 一首 歌
ASRDP_5645_2 给 我 播放 歌 吧
ASRDP_5645_3 我 要 听 前一个 歌
```

**wav.scp**   

这个文件告诉kaldi每个utterance-id对应的录音文件的位置。这里是非常简单的方法，用空格分开的两列。第一列是utterance-id(或者recording-id，参考segments)，第二列是录音文件的路径。我们这里的例子第二列是录音文件的路径，但是kaldi实际要求的是extended-filename，这个extended-filename可以是一个wav文件，但是也可以是一个命令，这个命令能产生一个wav文件(用管道符 “|” 进行处理)。

```
[xin_zhang@unn2 train] head -3 wav.scp
ASRDP_5645_1 /mnt/users/xin_zhang/.../data/ASRDP-5645/1.wav
ASRDP_5645_2 /mnt/users/xin_zhang/.../data/ASRDP-5645/2.wav
ASRDP_5645_3 /mnt/users/xin_zhang/.../data/audio/ASRDP-5645/3.wav
```

**utt2spk**

这个文件告诉kaldi，utterance-id对应哪个speaker-id。speaker-id不一定要求准确的对应到某个人，只要大致准确就行。如果我们不知道说话人是谁，那么我们可以把speaker-id设置成utterance-id，这是比较安全的做法。一种常见的错误做法是把未知说话人的句子都对应到一个“全局”的speaker，这样的坏处是它会使得cepstral mean normalization变得无效。

```
[xin_zhang@unn2 train] head -3 utt2spk
ASRDP_5645_1 ASRDP_5645
ASRDP_5645_2 ASRDP_5645
ASRDP_5645_3 ASRDP_5645
```

**spk2utt** 

这个文件和utt2spk反过来，它存储的是speaker和utterance的对应关系，这是一对多的关系，可以使用脚本来得到：

```
utils/utt2spk_to_spk2utt.pl data/train/utt2spk > data/train/spk2utt
```

###### 1.1.2 非必需文件

**segments**

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
s5# head -3 ../../rm/s5/data/train/spk2gender
adg0 f
ahh0 m
ajp0 m
```

##### 1.2 data文件生成

对于准备的TTS数据集，创建项目：

/mnt/users/xin_zhang/github_project/kaldi/egs/smarthome/s5  

用于生成wav.scp，text，utt2spk，spk2utt的脚本为./local/prepare_data.sh, 其中，text的生成用到了中文分词工具jieba.



#### 2. 语言模型及词典数据

##### 2.1 lexcion

需要自己准备的数据一般位于data/local/dict下，我们先看看其他的例子中的数据，以egs/aishell/s5为例：

```
[xin_zhang@unn2 s5] ls data/local/dict
extra_questions.txt  lexiconp.txt  lexicon.txt  nonsilence_phones.txt  optional_silence.txt  silence_phones.txt
```

一共有6个文件





1.生成全量的word token list

2. 用开源词典查找发音，没有的话就是oov，生成lexcion
3. 从lexicon中提取所有的音素生成phones

















































 

