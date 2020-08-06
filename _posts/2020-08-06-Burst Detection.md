---
layout: post
title: "Burst Detection"
author: "Niu"
---

# Burst Detection

## 算法流程

<img src="/sdnanjing.github.io/assets/imgs/burst_detection.png" alt="突发检测流程" style="zoom:40%;" />

## 目录结构

```
.
├── data																							# 数据目录
│   ├── season_data																		# 存放拆分后每个季度数据的目录
│   │   ├── season-2.em.csv
│   │   ├── season-3.em.csv
│   │   └── season-4.em.csv
│   ├── season_graph																	# 构建的每个季度的Keywords Graph
│   │   ├── season-2-keywords.em.gml
│   │   ├── season-2-keywords-weighted-edge.em.gml
│   │   ├── season-3-keywords.em.gml
│   │   ├── season-3-keywords-weighted-edge.em.gml
│   │   ├── season-4-keywords.em.gml
│   │   └── season-4-keywords-weighted-edge.em.gml
│   ├── season_model																	# 使用node2vec训练得到的keywords embedding
│   │   ├── season-2-model.em.bin
│   │   ├── season-3-model.em.bin
│   │   └── season-4-model.em.bin
│   ├── season_topic																	# 每个季度事件大类的关键词
│   │   ├── season-2-topic-keywords.em.txt
│   │   ├── season-3-topic-keywords.em.txt
│   │   └── season-4-topic-keywords.em.txt
│   ├── data.em.csv																		# 清洗分词后的数据
│   ├── event-keywords.em.txt													# 每个事件的关键词
│   ├── events.em.txt																	# 聚类后的每个事件
│   ├── keywords.em.csv																# 每篇新闻的关键词
│   ├── keywords.em.vocab															# 生成的关键词表
│   ├── news_em_1year.csv															# 原始数据
├── build_keywords_graph.py														# 用于构建Keywords Graph
├── bursty.py																					# burst detection的相关算法
├── clean_data.py																			# 清洗数据、分词
├── clear.py
├── document_cluster.py																# 用于将文章聚类为Story
├── evaluate.py
├── gather_events.py
├── gather.py
├── keywords_cluster.py																# 使用node2vec训练好的embedding进行聚类
├── keywords_extract.py																# 抽取关键词
├── prepare.py
├── split_data.py																			# 用于按季度拆分数据
└── train_node2vec.py																	# 训练node2vec向量
```

## 算法原理

### 1、抽取关键词

使用TextRank算法进行关键词抽取

### 2、关键词聚类

####  2.1 构建关键词共现图

使用上一步骤中抽取的关键词构建关键词 co-occurrence graph，该图每个节点代表一个关键词$w$，关键词之间的边 $e_{i,j}$ ($w_i$和$w_j$之间的边)的权重为两个关键词节点的共现次数。

图中不满足以下条件的edge会被删除：

1. 边的共现次数少于阈值（一般为3）
2. 共现的条件概率($Pr\{w_j|w_i\}及$$Pr\{w_i|w_j\}$)小于阈值(一般为0.15)

接下来需要在co-occurrence graph上进行Communities Detection

#### 2.2 训练node2vec

通过co-occurrence graph训练出每个节点的embedding

#### 2.3 聚类

使用二分k-Means算法对节点进行聚类，最终每个簇视为一个Community

### 3、Event识别

#### 3.1 Topic聚类

将每个Keywords Community视为一个Document，表示为fix-length（长度为关键词词表大小）的向量，每个元素的值取值为0或1，代表关键词是否在这个Community中。然后构建KD-tree。

每篇news通过tf-idf表示，通过kNN算法寻找其所属的topic

#### 3.2 Event聚类

构建Document Graph，每个topic中的news两两比较cosine距离，小于阈值则通过边相连，距离作为边的权重。

使用和keyword graph的Community Detection相同的算法进行聚类。每一个子图视为一个Event，每一个Event记作$\varepsilon$，取其中所有新闻关键词的并集作为事件的关键词，记为$\mathcal{C}_{\varepsilon}$。

#### 3.3 构建或扩展Story Tree

每个Story Tree记为$\mathcal{S}$，其关键词集合为$\mathcal{C}_{\mathcal{S}}$。计算$\mathcal{C}_{\varepsilon}$和$\mathcal{C}_{\mathcal{S}}$的Jaccard相似度。若与其相似度最高的tree的相似度高于阈值，则加入tree，否则创建新的Story Tree。

通过计算新加入事件与Story Tree中各个分支的compatibility, coherence, timePenalty来判断：1）新建分支，或2）扩展已有分支。

### 4、Burst Detection

将待检测事件所属的branch输入Kleinberg Burst Model 来检测突发事件，并计算突发度。

## 运行流程

### 1、清洗数据

文件：clean_data.py

输入：带有publish time, title, content的新闻数据.

输出：分好词去除停用词的content，title。publish_time不变

### 2、抽取关键词

文件：keywords_extract.py

输入：分好词的content, title

输出：每篇文档的关键词，所有文档关键词组成的词表

### 3、构建Keywords Graph

文件：build_keywords_graph.py	

输入：步骤1中分好词的content，步骤2中关键词表

输出：gml文件，内容为keywords graph

### 4、训练node2vec

文件：train_node2vec.py

输入：步骤3中构建的keywords grpah的gml文件

输出：每个keyword的embedding

### 5、关键词聚类

文件：keywords_cluster.py

输入：步骤4中输出的embedding

输出：每个topic对应的关键词

### 6、Story聚类

文件：docment_cluster.py

输入：步骤1中输出的content，publish time，步骤5中输出的每个topic的关键词，（可选：已生成的Story Tree）

输出：更新后的Story Tree，当前批次news所属的Story及其event

### 7、突发检测

文件：burst.ipynb

输入：步骤6中更新过的Story Tree

输出：检测出突发事件及突发度

