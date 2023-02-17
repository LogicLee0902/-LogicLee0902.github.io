---
layout:     post
title:      "小样本学习与原型网络"
subtitle:   Prototypical Networks for Few-shot Learning 论文笔记
date:       2023-01-30
author:     Leo
header-img: "img/post-prototype.jpg"
catalog: true
tags: ["Deep Learning@Tags@Tags", "Few-shot Learning@NeuralNetwork@Tags"]
lang: zh
katex: true
---
感谢[大佬](https://www.cnblogs.com/xiaohuiduan/p/16244173.html#%E7%AE%97%E6%B3%95%E6%B5%81%E7%A8%8B)

# Few-shot Learning & Prototype Network

## Few-shot Learning

### The Definition of Few-shot Learning

小样本学习是一种学习范式，旨在从少量的训练样本中学习一个模型。这是迁移学习的一个特例，训练数据是有限的。训练数据通常是少量的样本，而测试数据是大量的样本。少次学习的目标是学习一个可以泛化到测试数据的模型。

以分类任务举例，假设一个可以区分A，B的神经网络，当出现了一张新的类型为A的图片，已训练的网络提取特征后，可以判断出这张图片是A。
对于小样本学习，它可能只有一张A的图片，一张B的图片，来了一张新的A图片后，其发现新的图片和A长得类似，与此推断和A属于同一类别。

因此可以看出，小样本学习是对数据建模，从而实现分类效果

### Specific Methods of Few-shot Learning

小样本学习本质上就是N-way K-shot的分类问题，在训练时会分为Support Set和Query Set，Support Set用于训练，Query Set用于测试。

以下N=3, K=3为例，其实际意义为由在数据集中，选了3个类别，每个类别选了3个样本，N类中每类还要再选择X个数据。
N类，每类K个数据构成了Support Set，剩下的N类的每类X个数据构成了Query Set，

在训练集中，以上步骤构成的Support Set和Query Set会被input到Model中进行训练，称之为一个episodes，对于模型来说，其目的则为判断Query Set中的样本与哪一个Support Set最相似

![image.png](https://pic7.58cdn.com.cn/nowater/webim/big/n_v2330edf3028834586b93ca698602281b9.jpg)

## Prototype Network

原型网络是一种基于原型的小样本学习方法，其核心思想是将每个类别的样本看作一个原型，然后通过计算新样本与原型的距离来判断新样本的类别。

### Theory

它可以将Support Set中的数据(大多数情况为图片)$(\text{data}_1, \text{data}_2, \cdots, \text{data}_m)$, 通过一个神经网络得到特征向量$\text{feature}_1, \text{feature}_2, \cdots, \text{feature}_m$，然后将这些特征向量求平均得到原型向量$\text{prototype}_1, \text{prototype}_2, \cdots, \text{prototype}_n$，其中n为类别数。
对于Query Set中的数据，也是通过神经网络得到特征向量，然后计算与原型向量的距离（欧式距离或余弦距离），距离最近![20230203164729](https://cdn.jsdelivr.net/gh/LogicLee0902/ImageBed@main/blogs/imgs/20230203164729.png "yuanli")的原型向量所属的类别即为Query Set中数据的类别。

### Loss Function

典型的Prototype Network的Loss Function是一个交叉熵函数

> 交叉熵(Cross Entropy)函数是用来衡量两个概率分布之间的距离，其值越小，两个分布越接近。

pytorch中CrossEntropyLoss的计算方法如下所示，class代表x实际所属类别,$x[j]$代表x在第j类的概率

$$
\operatorname{loss}(x, \text { class })=-\log \left(\frac{e^{(x[\text { class }])}}{\sum_j e^{(x[j])}}\right)=-x[\text { class }]+\log \left(\sum_j e^{(x[j])}\right)
$$

在实际引用中，Loss需要取负号，以欧式距离为例，距离越远(d则越大，就是公式中的
$x[\text{class}]$，是伪代码中的$d(f_\phi(x, c_k))$)，则代表两者的相似度越低。如果不加负号的话，进行softmax计算，距离越远的则predict概率越大，这明显是错误的。因此，加了一个负号之后，距离越远，进行softmax之后，输出则越小，predict的概率也变小，这才是合理的。

![20230203175620](https://cdn.jsdelivr.net/gh/LogicLee0902/ImageBed@main/blogs/imgs/20230203175620.png)
