# Anomaly Detection

## 1. 算法流程：

- 训练阶段：
  1. data_process.ipynb 处理所有数据，对每个公司单独生成对应的CSV和文章标题txt。
  2. Feature_engineer.ipynb 在所有标题语料上训练w2v-tfidf模型，导出公司标题语料的句向量矩阵。
  3. EnsembleAD.ipynb 训练各公司的异常检测器。

- 测试阶段：
  1. test_new.ipynb 检测新数据是否为异常。

## 2. 目录结构

```
.
+-- data_process.ipynb               #数据处理
+-- feature_engineer.ipynb					 #文本向量化模型	
+-- EnsembleAD.ipynb						     #异常检测模型
+-- parallel_lscp.py						     #并行检测模块
+-- data									           #数据文件夹
|   +-- resources							       #依赖文件
|   	+-- sgns.financial.char.bz2		 #金融语料word2vec预训练模型
|		+-- news_em_same_company_1year.json	   #语料位置检索json
|   	+-- news_em_1year.csv				   #1年东方财富网语料(RAW)
|   	+-- stop_words.txt					   #停用词表
|   +-- models								       #模型
|   	+-- w2v_model.pkl					     #训练完的w2v模型
|   	+-- tfidf_model.pkl					   #训练后的tfidf模型
|   +-- company_csv							     #公司csv数据文件夹
|   	+-- east.csv
|   	+-- 华为.csv
|   +-- company_title						     #公司新闻标题文件夹
|   	+-- title_east.csv
|   	+-- title_华为.csv
|   +-- company_vec							     #公司新闻标题向量文件夹
|   	+-- title_华为_vec.np
|   +-- company_model						     #公司异常检测模型
|   	+-- 华为.pkl
|   +-- company_test						     #检测新数据
|   	+-- title_华为_new.txt
```

## 3. 算法运行

### 3.1 data_process.ipynb

- 输入：

  公司名称，如“华为”

- 输出：

  储存该公司的数据csv和新闻标题txt



顺序运行所有的cell，会读入所有东方财富网新闻1年的信息和各公司新闻在表中的位置信息。

在最后的cell中，以华为公司举例，运行 `gen_doc("华为")`，会在`/data/company_csv`文件夹下生成华为公司的数据 `华为.csv`；在`/data/company_title`文件夹下生成华为公司的标题文件 `title_华为.txt`，如上目录结构所示。其他公司同理，调用`gen_doc`函数生成相应文档。

### 3.2 Feature_engineer.ipynb

- 输入：

  公司名称：如”华为“

- 输出：

  储存该公司的新闻标题向量矩阵



顺序运行所有cell，会在所有标题新闻语料上对预训练的金融文本word2vec模型进行迁移训练。同时训练tf-idf模型。对一个句子中的所有词用tf-idf权重对word2vec向量加权得到向量表示。

在最后的cell中，以华为公司举例，运行`company2vec("华为")`，会在`/data/company_vec`文件夹下生成华为公司的新闻标题向量矩阵`title_华为_vec.np`, 如上目录结构所示。其他公司同理，用`company2vec`函数生成相应文档。

### 3.3 EnsembleAD.ipynb

- 输入：

  公司名称：如“华为”

- 输出：

  训练文档中的异常文档和异常值、储存该公司异常检测模型。



顺序运行所有cell，在最后的cell中，以华为公司举例，运行`huawei=EnsembleAD("华为", threshold=0.5)`,

会对该公司训练一个异常检测模型，按异常程度由重到轻输出结果。`threshold` 为异常置信阈值，即认为大于百分之多少可能性的异常为异常。同时，保存异常检测模至`/data/company_model/华为.pkl`

### 3.4 test_new.ipynb

- 输入：

  公司名称：如“华为”

- 输出：

  预测新文档为异常的概率。



顺序运行所有cell，在最后的cell中，以华为公司举例，运行`test_new("华为")`, 将读取`/data/company_test/title_华为_new.txt`，即新的需要检测的华为的新闻标题。程序将加载保存的w2v模型和tfidf权重，将新的文档转换为向量矩阵，加载训练好的华为公司的异常检测器，最后按异常程度输出每条新标题的异常值。

