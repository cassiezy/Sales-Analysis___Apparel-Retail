# Sales Analysis - Uniqlo
### Python EDA: Analyze sales data of Uniqlo China to answer the following business questions:
#### 1. Overall sales trend through time 整体销售情况随着时间的变化是怎样的？
#### 2. Sales performance of different categories/products 不同产品的销售情况是怎样的？
#### 3. Which purchase channel is the most popular? 顾客偏爱哪一种购买方式？
#### 4. What's the relationship between sales and COG? 销售额和产品成本之间的关系怎么样？

### Import packages
```Python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
%matplotlib inline
import seaborn as sns
sns.set(style="whitegrid")
```

### Import data
```Python
uni = pd.read_csv('week1.unique.csv')
uni.shape
# add a profit column
uni['profit']=(uni['unit_price']-uni['unit_cost'])*uni['quant']
uni.head(10)
```

### EDA
```Python
# check data type
uni.info()
uni_clean=uni.copy()
# change store_id to string
uni_clean['store_id'] = uni_clean['store_id'].apply(str)
uni_clean.info()
# take a look at the columns
uni_clean.describe(include="all")

# drop records with revenue <0
drop_idx = uni_clean[uni_clean['revenue']<0].index
uni_clean = uni_clean.drop(drop_idx)
uni_clean.describe(include='all')
```


## Question 1. Overall sales trend through time
· 选择什么变量？
    
    wkd_ind, revenue, profit, uni_quant_of_order, uni_revenue_of_customer
    channel, gender_group

· 呈现怎样的数据关系？
    
    比较周中/周末不同的销售情况。从渠道、性别、销售额、利润、顾客数量、单笔订单产品数量及人均消费额等维度分析

· 选择怎样的图表？
    
    柱状图
