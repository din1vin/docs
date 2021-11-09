## 数据质量?

### 1. 准确率检查 /v1/measure/accuracy

准确率是指入仓的数据与数据源的一致性比较.  

accuracy = count(source == target)  / count(1)

### 2. 非空检查 /v1/measure/nullable

非空检查是指,以用户指定的表格非空字段为前提,对结果表进行非空校验,统计非空字段为空的行数.



### 3. 数据量波动检查 /v1/measure/fluctuation

数据量波动检查是指,时间分区表在新增分区之后,跟上n个分区的数据量均值对比,阈值可指定.



### 4. 重复率检查 /v1/measure/repetition

重复率检查是指



