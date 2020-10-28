# Sales Analysis - Uniqlo
### Python EDA: Analyze sales data of Uniqlo China to answer the following business questions:
#### 1. Overall sales trend through time 
#### 2. Sales performance of different categories/products 
#### 3. Which purchase channel is the most popular? 
#### 4. What's the relationship between sales and COG? 

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
+ Variables to use?
   - wkd_ind, revenue, profit, uni_quant_of_order, uni_revenue_of_customer, channel, gender_group

+ What data relationships are presented?    
   - Compare sales performance of different channel/gender at weekends/weekdays
   - Analyze from sales, profit, number of customers, number of products per order, per capita consumption and other dimensions

+ What kind of charts?    
   - Bar chart

### A. Performance of difference channels (weekend vs. weekday)
Using metrics: sales, profit, number of products per order, per capita consumption
```Python
# add two columns: number of products per order, per capita consumption
uni_clean['uni_quant_of_order'] = uni_clean['quant']/uni_clean['order']
uni_clean['uni_revenue_of_customer'] = uni_clean['revenue']/uni_clean['customer']

# groupby channel and wkd_ind to see the 4 metrics
uni_wkd=uni_clean.groupby(['channel','wkd_ind']).agg({'revenue':np.sum, 'profit': np.sum, 'uni_quant_of_order':np.mean, 'uni_revenue_of_customer':np.mean}).reset_index()
uni_wkd
```
|channel	 |wkd_ind	|revenue  	|profit	|uni_quant_of_order	|uni_revenue_of_customer|
| --------|----------|-----------|--------|--------------------|---------------------- |
|Online	 |Weekday	|395302.56	|189110	|1.108044	         |92.739192 |
|Online	 |Weekend	|257868.58	|127754	|1.109055	         |93.502841 |
|Offline	 |Weekday	|1698646.08	|786041	|1.122747	         |93.686322 |
|Offline	 |Weekend	|1204616.29	|567741	|1.106883	         |92.801170 |

### B. Performance of difference gender (weekend vs. weekday)
