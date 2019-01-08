---
title: Learning To Ranking学习笔记
tags:
  - 排序学习
  - 笔记总结
  - 推荐系统

categories:
  - 推荐系统
  - 排序模型
---

#### 优化目标
为了避免优化函数h是一个常量，在loss fuction上增加一个平滑项τ， 0<τ≤1。在实际应用中τ为固定常数。
![](assets/markdown-img-paste-20181225083907295.png)


#### Reference
[gbRank & logsitcRank自顶向下](https://mlnote.com/2016/09/18/gbRank-logsitRank-from-up-to-bottom/)
[From RankNet to LambdaRank to LambdaMART: An Overview](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.180.634&rep=rep1&type=pdf)
