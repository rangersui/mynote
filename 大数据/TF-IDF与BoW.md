[toc]

# TF-IDF

**TF-IDF(Term Frequency-Inverse Document Frequency, 词频-逆文件频率)**是一种用于资讯检索与资讯探勘的常用加权技术。TF-IDF是一种统计方法，评估一字词对于一个文件集或一个语料库中的其中一份文件的重要程度。字词的重要性随着它在文件中出现的次数成正比增加，但同时会随着它在语料库中出现的频率成反比下降。

总而言之，**一个词语在一篇文章中出现次数越多, 同时在所有文档中出现次数越少, 越能够代表该文章。**

## TF

TF（Term Frequency,词频）表示词条在文本中出现的频率，这个数字通常会被归一化（一般是词频除以文章总词数），防止偏向长文件。

![[公式]](https://www.zhihu.com/equation?tex=TF_%7Bi%2Cj%7D%3D%5Cfrac%7Bn_%7Bi%2Cj%7D%7D%7B%5Csum_%7Bk%7D%7Bn_%7Bk%2Cj%7D%7D%7D%5Ctag%7B1%7D+%5C%5C)

一些通用的词语对于主题并没有太大的作用， 反倒是一些出现频率较少的词才能够表达文章的主题， 所以单纯使用是TF不合适的。权重的设计必须满足：一个词预测主题的能力越强，权重越大，反之，权重越小。

## IDF

所有统计的文章中，一些词只是在其中很少几篇文章中出现，那么这样的词对文章的主题的作用很大，这些词的权重应该设计的较大。IDF就是在完成这样的工作。**IDF(Inverse Document Frequency, 逆文件频率)**表示关键词的普遍程度。如果包含词条 ![[公式]](https://www.zhihu.com/equation?tex=i) 的文档越少， IDF越大，则说明该词条具有很好的类别区分能力。某一特定词语的IDF，可以由总文件数目除以包含该词语之文件的数目，再将得到的商取对数得到:

![[公式]](https://www.zhihu.com/equation?tex=IDF_i%3D%5Clog%5Cfrac%7B%5Cleft%7CD+%5Cright%7C%7D%7B1%2B%5Cleft%7Cj%3A+t_i+%5Cin+d_j%5Cright%7C%7D%5Ctag%7B2%7D+%5C%5C)

其中，![[公式]](https://www.zhihu.com/equation?tex=%5Cleft%7CD+%5Cright%7C) 表示所有文档的数量，![[公式]](https://www.zhihu.com/equation?tex=%5Cleft%7Cj%3A+t_i+%5Cin+d_j%5Cright%7C) 表示包含词条 ![[公式]](https://www.zhihu.com/equation?tex=t_i) 的文档数量，为什么这里要加 1 呢？主要是**防止包含词条 ![[公式]](https://www.zhihu.com/equation?tex=t_i) 的数量为 0 从而导致运算出错的现象发生**。

某一特定文件内的高词语频率，以及该词语在整个文件集合中的低文件频率，可以产生出高权重的TF-IDF。因此，TF-IDF倾向于**过滤掉常见的词语，保留重要的词语**，表达为

![[公式]](https://www.zhihu.com/equation?tex=TF+%5Ctext%7B-%7DIDF%3D+TF+%5Ccdot+IDF%5Ctag%7B3%7D+%5C%5C)

参考文献：

[TF-IDF 原理与实现](https://zhuanlan.zhihu.com/p/97273457) 

# BoW

词袋模型，文本向量化表示方法。

英文全称为**Bag-of-words，**简写为BOW，在信息检索中，BOW模型假定对于一个文档，忽略它的单词顺序和语法、句法等要素，将其仅仅看作是若干个词汇的集合，文档中每个单词的出现都是独立的，不依赖于其它单词是否出现。也就是说，文档中任意一个位置出现的任何单词，都不受该文档语意影响而独立选择的。

## 缺陷

词袋模型最重要的是构造词表，然后通过文本为词表中的词赋值，但词袋模型严重缺乏相似词之间的表达。

例如“她很美”和“她越来越漂亮了”，“她像花一样”，“她颜值高”，这几句话意思都是差不多的，但在词袋模型看来，它们的相似性还不如“她不美”，而“她不美”表达的是完全相反的意思.

## 实现

在python中词袋模型BOW通过CountVectorizer函数实现，下面例子为python函数说明中给出的例子。

```python3
CountVectorizer(input='content', encoding='utf-8',  decode_error='strict', strip_accents=None, lowercase=True, preprocessor=None, tokenizer=None, stop_words=None, 
token_pattern='(?u)\b\w\w+\b', ngram_range=(1, 1), analyzer='word', max_df=1.0, min_df=1, max_features=None, vocabulary=None, binary=False, dtype=<class 'numpy.int64'>)
```

CountVectorizer是通过fit_transform函数将文本中的词语转换为词频矩阵，矩阵元素a[i][j] 表示j词在第i个文本下的词频。即各个词语出现的次数，通过get_feature_names()可看到所有文本的关键字，通过toarray()可看到词频矩阵的结果。

参考文献：

[数据处理：CountVectorizer和OneHotEncoder](https://zhuanlan.zhihu.com/p/56531952)