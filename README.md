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
|   |store_id	|city	|channel	|gender_group	|age_group	|wkd_ind	|product	|customer	|revenue	|order	|quant	|unit_cost	|unit_price	|profit	|uni_quant_of_order	|uni_revenue_of_customer|
| ---|-----|------|----|------|-------|-----|-----------|--------|---------|--------|--------|------|-----|--------|-------|------------------------- |
|count	|22292	|22292	|22292	|22292	|22292	|22292	|22292	|22292.000000	|22292.000000	|22292.000000	|22292.000000	|22292.000000	|22292.000000	|22292.000000	|22292.000000	|22292.000000|
|unique	|64	|10	|2	|3	|11	|2	|9	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN|
|top	|207	|深圳	|Offline	|Female	|30-34	|Weekday	|T恤	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN|
|freq	|610	|4364	|18403	|14207	|4426	|12464	|10610	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN|
|mean	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|1.629508	|159.538557	|1.652028	|1.858066	|46.124529	|84.283779	|74.943747	|1.114455	|93.254588|
|std	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|1.785640	|276.258179	|1.861517	|2.347353	|19.124766	|46.311894	|179.894807	|0.443968	|76.634104|
|min	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|1.000000	|0.000000	|1.000000	|1.000000	|9.000000	|0.000000	|-650.000000	|1.000000	|0.000000|
|25%	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|1.000000	|64.000000	|1.000000	|1.000000	|49.000000	|56.000000	|18.000000	|1.000000	|59.000000|
|50%	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|1.000000	|99.000000	|1.000000	|1.000000	|49.000000	|79.000000	|42.000000	|1.000000	|79.000000|
|75%	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|2.000000	|175.000000	|2.000000	|2.000000	|49.000000	|99.000000	|87.000000	|1.000000	|99.000000|
|max	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|NaN	|58.000000	|12538.000000	|65.000000	|84.000000	|99.000000	|299.000000	|8400.000000	|22.000000	|6636.000000|



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


#### *Notice: There are five weekdays and only two weekends，therefore, add two columnsL: daily sales volume, daily profit*
```python
uni_wkd['revenue_per_day'] = uni_wkd.apply(lambda x: x['revenue']/5 if x['wkd_ind']=='Weekday' else x['revenue']/2, axis=1)
uni_wkd['profit_per_day'] = uni_wkd.apply(lambda x: x['profit']/5 if x['wkd_ind']=='Weekday' else x['profit']/2, axis=1)
uni_wkd
```
|channel	 |wkd_ind	|revenue  	|profit	|uni_quant_of_order	|uni_revenue_of_customer|revenue_per_day	|profit_per_day|
| --------|----------|-----------|--------|--------------------|-----------------------|-----------------|------------- |
|Online	 |Weekday	|395302.56	|189110	|1.108044	         |92.739192              |79060.512	      |37822.0       |
|Online	 |Weekend	|257868.58	|127754	|1.109055	         |93.502841              |128934.290	      |63877.0       |
|Offline	 |Weekday	|1698646.08	|786041	|1.122747	         |93.686322              |339729.216	      |157208.2      |
|Offline	 |Weekend	|1204616.29	|567741	|1.106883	         |92.801170              |602308.145	      |283870.5      |

#### Visualization
```python
# for windows to show Chinese characters
plt.rcParams['font.sans-serif'] = ['simhei']
```

```python
# sales
sns.barplot(x = 'channel', y = 'revenue_per_day', hue = 'wkd_ind', data = uni_wkd)
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/1.1.png)

```python
# profit
sns.barplot(x = 'channel', y = 'profit_per_day', hue = 'wkd_ind', data = uni_wkd)
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/1.2.png)

```python
# number of products per order
sns.barplot(x = 'channel', y = 'uni_quant_of_order', hue = 'wkd_ind', data = uni_wkd)
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/1.3.png)

```python
# per capita consumption
sns.barplot(x = 'channel', y = 'uni_revenue_of_customer', hue = 'wkd_ind', data = uni_wkd)
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/1.4.png)


## Conclusion
- **1. Offline average daily sales and profits are much higher than online**
- **2. Offline: sales and profits on weekends are almost twice as high as mid-week**
- **3. But there is little difference in terms of the number of products per order and the amount spent per person**

### B. Performance of difference gender (weekend vs. weekday)
