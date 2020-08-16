# Griffin的使用

### 简介

```
Apache Griffin（以下简称Griffin）是一个开源的大数据数据质量解决方案，它支持批处理和流模式两种数据质量检测方式，可以从不同维度（比如离线任务执行完毕后检查源端和目标端的数据数量是否一致、源表的数据空值数量等）度量数据资产，从而提升数据的准确度、可信度。
```

### 架构

 ![img](https://img-blog.csdnimg.cn/20190406181211343.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3pnX2hvdmVy,size_16,color_FFFFFF,t_70) 

#### 组件：

- Define：主要负责定义数据质量统计的维度，比如数据质量统计的时间跨度、统计的目标（源端和目标端的数据数量是否一致，数据源里某一字段的非空的数量、不重复值的数量、最大值、最小值、top5的值数量等）
- Measure：主要负责执行统计任务，生成统计结果
- Analyze：主要负责保存与展示统计结果

#### 功能：

- 从以上介绍可以看出，griffin的数据源可以是hadoop，rdbms，kafka。而抽象架构中的流数据源支持主要是指对kafka的支持，离线数据源主要是指对hadoop的支持。
- griffin可以定义对数据的：精确度(Accuracy)，合法性(validity)，一致性(consistency)，时间序列(timeliness)，完整性(completeness)等进行检测。
- griffin的检测任务是运行在spark基础上的，也就是说，先定义检测的标准，根据标准生成spark任务。