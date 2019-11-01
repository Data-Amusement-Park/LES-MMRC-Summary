# LES-MMRC-Summary

## 赛题简介

关于赛题的详细信息可以参见官网 [莱斯杯：全国第二届“军事智能机器阅读”挑战赛](https://www.kesci.com/home/competition/5d142d8cbb14e6002c04e14a)

**这里简单回顾下赛题摘要：**本次竞赛将提供面向军事应用场景的大规模中文阅读理解数据集，围绕多文档机器阅读理解进行竞赛，涉及理解、推理等复杂技术。每个问题对应五篇候选文章，问题与篇章中的答案证据句间存在较大的语法与句式变化。需要在多篇章定位与深度理解的基础上，从存在干扰项的多篇文章中搜寻出最优答案，更富挑战性的是问题的答案可能需要结合至少两篇文章的相关内容，进行关联推断才能够准确得出。

决赛的时候，一共有3轮，每一轮问题类型不同，分别是：**单答案问题、多答案问题和推理型问题**，分数比例分别是 **20%、30%和50%**。

**3轮的竞赛成绩如下：**

<div align=center>
<img src="http://image.yingzq.com/img/20191028234602.png" width="500" alt="第一轮：单答案问题" />

第一轮：单答案问题

</div>

<div align=center>
<img src="http://image.yingzq.com/img/20191028235008.png" width="500" alt="第二轮：多答案问题" />

第二轮：多答案问题

</div>

<div align=center>
<img src="http://image.yingzq.com/img/20191028235727.png" width="500" alt="第三轮：推理型问题" />

第三轮：推理型问题

</div>

**最终排名如下：**

<div align=center>
<img src="http://image.yingzq.com/img/20191029000503.png" width="500" alt="最终排名" />

</div>

## 本项目说明

本人参加了本次竞赛，成绩并不理想（最终排名第7），所以为了加深自己对机器阅读理解和本次竞赛的了解，特地整理了最终成绩前十队伍的PPT和代码。

其中有些队伍由于实验室、公司的限制并没有放出PPT和相关代码，所以收集的并不完整，而且几乎没有队伍放出代码。

## 前10团队PPT/PDF（持续更新）

- [第1名-向阳而生团队 PPT](http://image.yingzq.com/les-mmrc/doc/%E7%AC%AC1%E5%90%8D-%E5%90%91%E9%98%B3%E8%80%8C%E7%94%9F%E5%9B%A2%E9%98%9F.pptx)
- [第4名-Lxy团队 PDF](http://image.yingzq.com/les-mmrc/doc/%E7%AC%AC4%E5%90%8D-Lxy%E5%9B%A2%E9%98%9F.pdf)
- [第5名-SJL团队 PPT](http://image.yingzq.com/les-mmrc/doc/%E7%AC%AC5%E5%90%8D-SJL%E5%9B%A2%E9%98%9F.pptx)
- [第7名-Lucky Boys团队 PPT](http://image.yingzq.com/les-mmrc/doc/%E7%AC%AC7%E5%90%8D-Lucky%20Boys%E5%9B%A2%E9%98%9F.pptx)
- [第10名-jy2211团队 PPT](http://image.yingzq.com/les-mmrc/doc/%E7%AC%AC10%E5%90%8D-jy2211%E5%9B%A2%E9%98%9F.pptx)

## 前10团队开源代码（持续更新）

暂无

## 分析

> 注：这部分是我本人结合前排大佬的PPT以及和他们的聊天，自己进行了简单的分析，不能保证完全正确

### 向阳而生团队

可以下载他们的答辩PPT查看，在这里简单说下我的想法：

- 首先本次竞赛每篇document的长度好几千，而`BERT`最大支持512长度，虽然可以利用滑窗的策略处理长度大于512的文本，但是时间代价太大，所以他们使用了`BM25算法`做了段落检索，在保证答案召回率的同时极大的减少了文本长度
- 对于单答案问题，使用标准的`BERT MRC`即可，需要注意的是使用了规则匹配的方式对`短答案会出现赘余问题`做了后处理
- 对于多答案问题，其实分为两种情况：一种是`可分解的多答案`，该类问题一般具有显著的特征，例如有多个问号，前半句和后半句都有疑问词，这种情况直接拆分成两个问题，当做单答案做即可，需要注意的是切分后需要识别第一个问题的实体与第二个问题拼接；另一种是`不可分别的多答案`，该类问题一般是“哪几个”、“哪些”这样的特征值，这种问题不好拆分，所以使用了`mask答案`的策略处理，简单来说，就是多次预测该问题的答案，只不过在本次预测之前，需要将文档中出现的前一次答案去除掉或做mask处理，再进行标准的单答案MRC
- 对于推理问题，使用`两阶阅读策略`，将推理问题阅读的过程分解为`实体阅读`和`答案阅读`两个阶段；该团队在推理问题的表现非常好，值得学习，具体做法如下：
	1. 利用`问题分解`模型，先将原问题分解为两个子问题；训练集的得来是根据推理问题的两个支撑段落，它们分别覆盖问题的不同部分，根据这两段覆盖的范围做标注，主要用最长匹配的算法，覆盖的时候会有三种情况分别处理，最后利用策略进行过滤，保证留下来的标注样本都是比较好的
	2. 通过回答第一个子问题得到中间的`桥接实体`
	3. 将该实体与第二个问题拼接重构成普通问题
	4. 进行标准MRC模型进行预测

### baseline团队

baseline的老哥由于种种原因并没有放出PPT，但是在大赛群里简单说了一下思路：**pipeline，5个模型。第一个模型用于判断答案数量；第二个模型用于检索，将document分成20段，检索出top5；第三个模型 阅读理解 抽取答案，得到五个候选；第四个模型 答案排序，基于候选答案上下文排序，取top3；第五个模型 答案关联排序，依据答案数量 直接生成结果。**

我的理解是：

- 第一个模型是一个2分类，分别代表该问题有1个答案、2个答案；具体实现是直接输入问题，通过`BERT`后将`[CLS]`位置的向量投影到2维空间，进行2分类。值得注意的是其实本次竞赛也有3答案的问题，但是非常非常少，所以在这里就不考虑了
- 第二个模型是由于`BERT`最大支持512长度字符，所以以句子为基本单元拼接document，达到512长度后便作为`一段`，一个document平均几千字符，所以平均被分成了20段，该模型的目的是检索出当前document最好的一段。而本次比赛一个问题对应5个document，所以是检索出top5；具体实现是根据`一段`中是否包含答案来构造数据集，通过`BERT`后将`[CLS]`位置的向量投影到2维空间，进行2分类
- 第三个模型是标准的MRC，对于上一步得到的top5，抽取答案，得到5个候选答案
- 第四个模型是基于候选答案的上下文排序，**依据是有答案的段落不一定是好段落，能抽出答案的段落才是**；具体做法是抽出top5答案所在的上下文，将问题与上下文拼接后通过`BERT`，将`[CLS]`位置的向量投影到1维空间算出一个rank分数，利用这个分数进行排序，然后取top3对应的答案
- 第五个模型是答案关联排序；不太清楚如何做，我个人认为是将问题和top3的答案的答案拼接在一起后通过`BERT`，将`[CLS]`位置的向量投影到3维空间，分别代表3个答案的rank分数；最后根据第一个模型所预测的答案个数 k，取排序后的top k个答案

### Lxy团队

该团队是初赛和复赛的第一，也放出了相应的答辩PPT，可以下载研究。

Lxy致力于预训练模型架构的优化，给人一种耳目一新的赶脚，简单说一下他们的思路：

- 在文档预处理方面，将文档以句子为单位拆分成长度在512左右的子文档，进行子文档筛选，极大减少了文档长度，并且召回率达到97%以上
- 在问题预处理方面，主要是制定规则来处理`多答案问题`和`推理问题`。比较新颖的是他们利用哈工大pyltp的分词和词性标注工具，对问题进行处理，然后利用`词性`定制了一些`拆分规则`，对多答案问题和推理问题进行拆分
- 在模型细节方面，PPT讲了很多，该团队基于`ERNIE 1.0`，尝试了很多种不同的模型结构，还是挺有意义的：
	- 在`ERNIE`输出层，添加3层Query-Context CoAttention
	- 利用`ERNIE`的最后3层编码结果分别输入给3个Query-Context CoAttention模块
	- etc...

### SJL团队

该团队也放出了PPT，大家可以参考，这里简单说一下思路：

- 由于`BERT`处理字长限制，使用基于`TF-IDF`的段落截断，能定位96%以上答案
- 对于多答案，基于问号进行了问题拆分，也就是处理了有多个问号的多答案情况，值得注意的是，这里的第二问的主语常常是代词，所以对于第二问`拼接了第一问的前40%文本`，虽然简单但是极为有效；对于一问多答的情况，只进行单答案预测，放弃了这种情况
- 对于推理题，训练了一个模型实现原问句拆分；训练集来源是初赛训练集中随机选取2000个问句进行人工标注
- 模型方面在`BERT`输出层额外添加了BiLSTM和DGCNN层，另外还用`[CLS]`位置的向量投影到1维后代表`段落含有答案的概率`，然后再点乘`段落与问句的TF-TF-IDF相似概率`，当做一个`Gate`使用在`Span 概率`上
