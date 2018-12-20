---
title: 《Deep neural network marketplace recommenders in online experiments》阅读笔记
tags:
  - 推荐系统
  - 工程实现
  - 论文笔记
categories:
  - 推荐系统
  - 论文阅读
date: 2018-12-20 10:09:42
---


> 论文作者来自一家电商公司，主要论述了三种方法在推荐系统线上应用的效果：结合商品内容以及用户行为的商品embedding学习；利用前面学习到的商品的embedding表达，结合RNN的用户行为序列建模；以及利用MAB来实现的reranking（其实没有看到探索的介绍，很简单）。论文整体上质量一般，比较有亮点的地方就是学习图像embedding的方法，以及采用Siamese Network的使用。

#### 商品embedding表示学习
> 作者先用不同的submodel分别得到商品基于行为、文本、图像，以及位置的embedding，然后将各种来源的embedding concat在一起，然后接了一个attention层以对不同来源的embedding加权，接着用MLP对embedding向量做压缩，得到100的向量embedding向量。作者通过图像领域常用的Siamese Network来对整个网络进行学习，并假设同一用户同一天转化的商品为相似商品，否则为不相似商品。

{% asset_img markdown-img-paste-20181220082313811.png 商品embedding网络结构图 %}

##### 商品各来源embedding计算

* **用户行为**：利用标准的ALS算法，学习得到每个商品基于用户行为数据的表达；
* **文本数据**：作者利用TextCNN训练了一个其业务内的文本分类器，然后将商品的标题和描述过一遍该分类器，将网络的顶层（应该是softmax的前一层）拎出来作为文本的embedding向量；
    ```
    作者这里采用监督学习算法得到文本的embedding表达，相比通过avg(word embedding)得到的embedding表达，效果应该会要好一些，但是选择哪个业务的文本分类模型，也是一个比较偏实际应用的问题。
    ```
* **图像数据**：这个比较有亮点，作者不是采用常规的做法，用图像类标作为监督的目标，而是将商品的标题通过word embedding得到title embedding之后，训练模型，用图像来预测该title embedding，从而学到图像更丰富的表达。作者对这种做法的解释是：标题描述给出了更丰富的信息，比如`wedding dress` vs `summer dress`，假如用类标`dress`，则无法对这种细节信息进行区分。作者用预训练好的Inception-v3作为网络结构，用MSE作为Loss Function；
* **位置信息**：作者没有采用LBS的距离信息来表示embedding，而是通过`user-postcode`行为矩阵，通过ALS学习位置的embedding表达；

##### 各来源embedding的融合
> 作者主要通过一个attention层来对不同来源的embedding做加权。每个来源的embedding都被表示为100维的向量。作者没有详细说明attention的做法，有知道具体做法的欢迎留言。

##### 模型参数学习
作者通过图像领域常用的`Siamese Network`来学习多来源embedding融合后的网络的参数，并假设同一用户同一天有过转化的商品是相似商品，否则是不相似商品。记得14年有一篇NLP的网红文也采用了类似的思想来训练模型。这种思想还是很经典，很值得思考的，在缺少标注数据的场景下，这种做法很值得尝试，比非监督学习算法应该会有一定的效果提升。

#### 基于序列的模型
作者用一个单层的GRU模型，用用户历史行为序列作为输入，来预测用户未来的点击序列。作者采用上节方法得到的商品embedding作为商品的embedding表达。方法上中规中矩，没有太多创新。

#### MAB
本质就是一个点击率预估模型，没有什么亮点可言，略感失望。
{% asset_img markdown-img-paste-2018122009414371.png Deep MAB %}

#### Reference
[1] [《Deep neural network marketplace recommenders in online experiments》](https://arxiv.org/pdf/1809.02130.pdf)
