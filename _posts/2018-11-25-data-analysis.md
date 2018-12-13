---
title: "【数据库】数据分析行业调研"
layout: post
tags:
  - Database
  - Python
categories: 解问题
---

「数据分析」这一概念每个人都多少有所接触，并且在日常生活中经常用到。无论是观察市场行情进行合理投资，还是处理一份excel报表，或者是电商大促时观察各种活动的折扣力度。这些工作有时借助了计算设备，有时是经过一阵思考，有时甚至直接发生在潜意识里。随着信息化程度的日渐提高，世界上每时每刻都有新的数据产生和传播。对于企业而言，利用好内部数据可以优化资源配置，充分借助外部数据可以完善产品，制定方略。因此，拥有一个优秀的数据分析团队对于企业来说无疑是至关重要的，互联网企业更是如此。

硬件计算能力和网络传输速度的提升为数据的获取提供了良好条件，受此启发，本文作为课程第二次调研报告，参考文章[1]的思路和做法，尝试从第一手数据出发，观察并分析企业中「数据分析」这一岗位体现出的具体需求，并将结果与旧数据进行对比，从一些特定角度刻画国内数据分析领域一年半内的发展状况。

<!-- more -->

## 工具和库

为了更加便利地获取、处理和分析数据，我们借助下表中的工具和库来完成调研。

|       项目       |   工具/库 名称    |  版本号  |
| :--------------: | :---------------: | :------: |
|   **数据获取**   |   Gooseeker [2]   |          |
|   **编程语言**   |      Python       |   3.6    |
| **集成开发环境** | PyCharm Community | 2017.3.4 |
|  **科学计算库**  |     Numpy [3]     |  1.15.0  |
| **数据分析工具** |    Pandas [4]     |  0.23.4  |
|   **分词工具**   |     Jieba [5]     |   0.39   |
| **词云生成工具** |   Wordcloud [6]   |  1.5.0   |
|   **制图工具**   |  Matplotlib [7]   |  3.0.2   |

```python
import numpy as np
import pandas as pd
import jieba
import jieba.analyse
import random
from wordcloud import WordCloud
from matplotlib import pyplot as plt
from matplotlib.font_manager import FontProperties
import matplotlib as mpl
```

## 数据获取

进行数据分析前需要构建数据集，我们从拥有海量互联网职业数据的招聘网站拉勾网[10]获取数据，并使用数据采集平台Gooseeker完成数据集的构建excel数据表。具体思路大致如下（图1）：

- 在拉勾网上以「数据分析」作为关键词进行搜索；

- 使用Gooseeker提供的网络爬虫对搜索结果列表进行抓取，得到数据集「拉勾网列表数据」；

- 对列表数据中的每一条记录，再单独开启一个脚本，逐一获取岗位的详细信息；

- 将上述信息合并，得到对应的数据集「拉勾网详情数据」；

- 用Excel处理数据集文件，删除不必要的列，并给予每一列适当的命名；

- 用数据分析工具pandas读取excel，并清除重复的记录（将同公司、同名称、同描述的职位视为同一记录）。

  ```python
  data = pd.read_excel('Data_lagou_edit.xlsx')
  data.head()
  
  # 将同公司、同名称、同描述的职位视为同一记录
  clean_data = data.drop_duplicates(['company','month_salary','description','title'])
  print(clean_data.info())
  ```

![](https://github.com/HusterHope/blogimage/raw/master/20181125-1.jpg)

[^图1]: 数据获取大致过程

本阶段我们共获取417条与「数据分析」岗位相关的数据，去除重复数据后，剩余412条有效数据。

## 数据整理和分析

### 地域分布情况

首先是「数据分析」岗位的地域分布，核心源码如下，我们统计数据中的`city`列，并按照降序排列，将[1]中的结果共同放入图2。

```python
count_by_city = clean_data.groupby(['city'])['title'].count().sort_values(ascending=False)

fig1 = plt.figure(figsize=(18,10))
ax1 = plt.subplot(111)
rect1 = ax1.bar(np.arange(len(count_by_city)),count_by_city.values, width = 0.5)

# x轴刻度标签
def auto_xtricks(rects,xticks):
    ...

auto_xtricks(rect1,count_by_city.index)
ax1.set_xticklabels(count_by_city.index, fontproperties = font_small)

# 设置数据标签
def auto_tag(rects, data = None, offset = [0,0]):
    ...

auto_tag(rect1,offset=[-1,0])
ax1.set_title(u'不同城市需求量', fontproperties = font_big)
```

![](https://github.com/HusterHope/blogimage/raw/master/20181125-2.jpg)

[^图2]: 不同城市的需求变化，其中下方图片为本次统计数据

可以看出，在拉勾网上共有27个城市的企业有数据分析岗位的需求，其中超过三分之一的需求来自北京， 近8成（79.4%）的需求来源于头部四大城市：北京、上海、杭州、深圳。对比2017年3月份的数据，我们看出杭州的需求量在这一年半的时间内有了明显提升，从2017年3月与广州不相上下，到2018年11月明显超过深圳。这也从侧面体现出了大企业能够推动城市内同行业的发展。

### 职位薪酬分布情况

拉勾网上的薪酬数据形式为「aK-bK」，相比城市数据而言并不直观。我们分别取最高值、平均值和最低值制图（以下代码计算平均值），所得结果如图3所示。

```python
def avg_salary(salary):
    try:
        s_list = salary.split('-')
        s_min = int(s_list[0][:-1])
        s_max = int(s_list[1][:-1])
        s_avg = float(s_min + s_max)/2
    except UnicodeEncodeError:
        s_list = salary.split('k')
        s_avg = float(int(s_list[0][:-1]))
    return s_avg
```

![](https://github.com/HusterHope/blogimage/raw/master/20181125-3.jpg)
[^图3]: 薪酬分布图，其中(d)为2017年3月的平均薪酬分布

对比图3中的(b)和(d)，我们看出数据分析师这一行业的平均薪酬呈现上涨趋势（最高峰从5-10k来到了10-15k），超50%的从业者月薪在15k-25k之间。对比(a)和(c)，不难发现企业给数据分析师的薪酬波动较大，这一现象在25k以上的区间段表现尤为明显。

### 城市薪酬分布情况

接下来我们按城市分组，对比不同城市的具体薪酬分布情况。由于27座城市中，第7位以后的城市职位数据均未超过十条，样本容量较小，因此只对前6名的城市进行统计（北京、上海、杭州、深圳、广州、成都）。所得箱线图结果如图4所示。

```python
small_data_by_city = count_by_city[0:6]
df = []
for group in small_data_by_city.index:
    v = count_by_city_salary.get_group(group).values
    df.append(v)
```

![](https://github.com/HusterHope/blogimage/raw/master/20181125-4.png)
[^图4]: 按城市划分的薪酬分布图

从图上看，这六大城市的薪酬分布情况大致分为两个梯队，北京、上海、深圳薪酬分布中位数在20k左右，属第一梯队，其中北京的浮动范围最大，而杭州、广州、成都薪酬分布中位数在15k左右，属第二梯队，其中杭州的上升空间较大。这和我们前面分析的全国职位总体需求情况相符，杭州有明显后来居上的趋势。同样，这两个梯队中各个城市的物价和消费水平差异，同样在薪酬上有所体现（北京、上海、深圳的消费水平从主观上要高于杭州、广州和成都）。

### 工作经验需求情况

从宏观上了解了各个城市的需求情况后，我们接下来更加深入地了解这个行业本身的需求。首先按照工作经验的需求进行分组。

注意到原始数据中’Experience’一栏有“经验不限”，“经验应届毕业生”和“经验1年以下”等3类近似条件，可以在处理时将它们视作同一类，这里统一归为“经验1年以下”。所得结果如图5所示。


```python
# 将这几类要求合并
for j in range(len(clean_data['experience'])):
    if clean_data['experience'].iloc[j] in [u'不限 /',u'应届毕业生 /']:
        clean_data['experience'].iloc[j] = u'1年以下 /'
        # print("Cleaned!")
...
# 按照经验需求排序
mappings = {u'1年以下 /':1 ,u'1-3年 /':2 ,u'3-5年 /':3 ,u'5-10年 /':4 ,u'10年以上 /':5}
sort_by_exp['sortby'] = sort_by_exp['exp'].map(mappings)
sort_by_exp.sort_values('sortby', inplace = True)
```

![](https://github.com/HusterHope/blogimage/raw/master/20181125-5.jpg)

[^图5]: 工作经验需求分布 (b)为2017年3月的工作经验分布

前面我们提到了平均薪酬相比一年半以前有明显上涨的现状，而对比图5中(a)和(b)的结果后不难看出，企业对工作经验的要求也有了明显提升。超7成的职位需要有1年以上的工作经验，其中又有近6成需要3年以上的工作经验。

随着工作经验的提升，薪酬是否也会逐年上涨？我们接下来尝试分析这两者的关系，并作出薪酬随工作经验增长的变化情况，如图6所示。

![](https://github.com/HusterHope/blogimage/raw/master/20181125-6.png)

[^图6]: 不同工作经验下的薪酬分布

随着工作经验的提升，薪酬的中位数也在稳定上涨。大约分别对应10k/月，15k/月，20k/月和30k/月（10年以上的情况由于只有一个样本，此处略去）。我们特别注意3-5年与5-10年的薪酬差距，后者大约比前者提高了50%，同时最低和最高工资线都有了明显提升。这也就意味着富有经验的数据分析师对于企业而言更具价值，与我们的预期基本相符。

### 职业技能关键词

在本节最后，我们从每个企业对求职者的具体要求出发，借助分词工具进行关键词提取和统计，直接分析企业的需求。

```python
# 关键词抽取
def key_words(text):
    key_words = jieba.analyse.extract_tags(text, topK=20, withWeight=False, allowPOS=())
    return key_words

clean_data['key_words'] = clean_data['description'].apply(key_words)

# 创建一个文本，将关键词列表全部写入该文本
def write_to_text(word_list):
    f = open("word_list_text.txt","a")
    for word in word_list:
        f.write(word+u',')
    f.close()
```

![](https://github.com/HusterHope/blogimage/raw/master/20181125-7-1.png)

![](https://github.com/HusterHope/blogimage/raw/master/20181125-7-2.png)
[^图7]: 关键词提取

如图7所示，我们将关键词分为中文和英文分别统计。中文部分除掉「数据」、「数据分析」这类宽泛的名词外，主要是突出工作者的职业素养，一些被强调的关键词包括：「建模」、「负责」、「用户」、「团队」、「运营」等。而英文部分更多是核心技能要求，如「Python」、「SQL」、「Excel」、「SPSS」、「Hiva」「Matlab」、「Spark」、「Hadoop」等。对比2017年3月的数据（图8），能够看出企业对使用Python的需求有了较大提升。

![](https://github.com/HusterHope/blogimage/raw/master/20181125-8.png)
[^图8]: 2017年3月关键词数据

## 总结

数据分析并不是特别新鲜的概念，近年来随着社会信息化程度的日益提升，数据分析这一职位发展势头迅猛。

从城市层面看，北京和上海依旧是资源聚集地，坐稳了国内前两位。杭州市在龙头企业（如阿里巴巴）的推动下走上了快车道，人才需求量猛增。人才需求量前6的城市提供的平均薪酬形成了由明显差距的两个薪酬梯队，分别为20k/月和15k/月。

从职业需求看，目前市场更倾向于有一定经验的数据分析师，3年左右的工作经验具有较强议价能力，5年以上的工作经验能够带来一个不小的收入跃升。行业所需的核心技能是以数据库操作和可视化工具为主的若干软件或编程语言。此外，从业者应充分了解行业最新需求，丰富自身技能，紧跟数据分析的时代潮流。

## 参考资料

[1] 《数据分析师挣多少钱？“黑”了招聘网站告诉你！》<https://zhuanlan.zhihu.com/p/25704059>

[2] 集搜客 - 网页抓取和整理 <https://www.gooseeker.com/>

[3] Numpy -  Fundamental package for scientific computing with Python <http://www.numpy.org/> 

[4] Pandas - Data structures and data analysis tools for the Python programming language <https://pandas.pydata.org/>

[5] Jieba - 结巴中文分词 <https://github.com/fxsjy/jieba>

[6] Wordcloud - A little word cloud generator in Python <https://github.com/amueller/word_cloud>.

[7] Matplotlib - Python 2D plotting library <https://matplotlib.org/>

[8] 拉勾网 - 互联网招聘平台 <https://www.lagou.com>



ps. 本文所有数据均于2018年11月23日从拉勾网[8]获得，部分数据来源于[1]，仅作学习使用。