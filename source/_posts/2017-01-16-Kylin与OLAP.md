---
title: Kylin使用小结
tags:
  - big data
  - Kylin
date: 2017-01-16 20:16:21
categories:
---


# 认识Kylin
之前的BI报表开发，都是基于关系型数据库，数据仓库工程师把数据放到了大宽表里。但是关系型数据库无法满足
日益增长的查询需求，而且性能受到了挑战，关系型数据仓库需要向大数据数据仓库转型。
我是从Web开发走过来的，不了解数据仓库的设计逻辑，在网上查了下资料，了解了数据仓库的星型模型和星座模型，
比如[这篇](http://shopperplus.github.io/blog/2015/04/12/data-warehouse-schema-desgin.html)和[星型模型wiki](https://en.wikipedia.org/wiki/Star_schema)。
如果更复杂的可以用雪花模型snowflake schema [雪花模型wiki](https://en.wikipedia.org/wiki/Snowflake_schema)  
大数据的数据仓库，用Hive的比较多，Kylin是大数据的数据仓库OLAP引擎。

# 使用中的问题
新手最重要的是看懂Learn Kylin里面的example，把每一个细节都看懂，这样就可以设计一个好的Cube.
在这里我先跳过安装的步骤，直接总结使用Kylin两周以来遇到的问题：

## 设计Model
- Kylin只支持星型模型，这一点非常重要，如果已经有了Hive表但不是星型模型，可以通过创建Hive View来使用Kylin。
- Model Diemnsions 需要把所有要用到的Dimensions都选上,如果遗漏了Column，在设计Cube时就找不到了。
- Measures也是同样，不要遗漏
- settings中的过滤功能可以过滤掉不需要的数据

## 设计Cube
- 设计Cube dimension，可以直接用Auto generation功能生成所需的维度
- 维度有两种类型：有外键对应的维度表中的column，称为derived维度，意思是由外键派生出来的；其他的都是normal维度。
  接下来在advnced settings中，mandatory 维度意思是每个cube都包含的；Hierarchy意思是分层的维度，例如国家-省份-城市关系
- 在advanced settings中，rowkeys是自动生成的，但是如果column的值太长，比如ID这种字段，Encoding如果用默认的dict会报错  
`Caused by: java.lang.IllegalArgumentException: Too high cardinality is not suitable for dictionary`  
改成integer后报错消失，但目前我对rowkeys还是不太理解，我只试过integer。
- 如果Cube已经build过，对他修改前需要先Purge

## 查询
Kylin的SQL语法使用了Apache Calcite, 功能还是很强大的，但是使用中也有一些限制：
- 不支持case when，使用case when的场景就说明聚合时还要对cube的measure结果进行重新分割，不符合Kylin的设计。可以把case when的场景就说明聚合时还要对cube的measure结果进行重新分割，不符合Kylin的设计，
可以把case when条件写入到列中，或者把条件放到一个lookup table中；如果Hive表改动代价比较大，可以使用Hive视图，性能是不会受到损失的。

