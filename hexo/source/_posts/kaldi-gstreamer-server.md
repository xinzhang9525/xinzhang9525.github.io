---
title: kaldi-gstreamer-server 搭建基于kaldi的语音识别服务
excerpt: kaldi; 语音识别;
index_img: /img/post/answer/lm/lm-smoothing.jpeg
date: 2021-11-23 15:30:00
categories: [语音识别]
comments: true
math: true
---
### kaldi-gstreamer-server docker环境搭建

https://github.com/jcsilva/docker-kaldi-gstreamer-server

Pull Docker Hub 

```
docker pull jcsilva/docker-kaldi-gstreamer-server
```

准备模型以及yaml配置文件,此处demo搭建使用的是0-10的英文数字识别的模型

```
[root@centos_asr kaldi_models]$ ls /root/xin/kaldi_models/digits
final.mdl  HCLG.fst  words.txt
```

```
[root@centos_asr kaldi_models]$ head digits_tri1.yaml
timeout-decoder : 10
decoder:
   model: xin/kaldi_models/digits/final.mdl
   word-syms: xin/kaldi_models/digits/words.txt
   fst: xin/kaldi_models/digits/HCLG.fst
   silence-phones: "1:2:3:4:5"
out-dir: tmp

use-vad: False
silence-timeout: 60
```



运行docker环境，并把本地模型数据暴露给docker

```
docker run -it -p 8080:80 -v /root/xin/kaldi_models:/opt/xin/kaldi_models jcsilva/docker-kaldi-gstreamer-server:latest /bin/bash
```

在container中启动服务

```
/opt/start.sh -y /opt/xin/kaldi_models/digits_tri1.yaml
```



服务启动成功之后即可用于识别准备好的音频：

```
root@cd4dcab22d74:/opt# python kaldi-gstreamer-server/kaldigstserver/client.py -u ws://1.14.104.94:8080/client/ws/speech -r 32000 xin/kaldi_models/1-789.wav
seven eight nine.

```



















































 

