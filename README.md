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


#### *Notice: There are five weekdays and only two weekends，therefore, add two columnsL: revenue_per_day, profit_per_day*
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
- **Offline average daily sales and profits are much higher than online**
- **Offline: sales and profits on weekends are almost twice as high as mid-week**
- **But there is little difference in terms of the number of products per order and the amount spent per person**


### B. Performance of difference gender (weekend vs. weekday)
Using metrics: sales, profit, number of customers, per capita consumption
```Python
# groupby gender and wkd_ind to see the 4 metrics
uni_gen=uni_clean.groupby(['gender_group','wkd_ind']).agg({'revenue':np.sum, 'profit': np.sum, 'customer':np.sum, 'uni_revenue_of_customer':np.mean}).reset_index()
uni_gen
```
|gender_group |wkd_ind	|revenue  	|profit	|uni_quant_of_order	|uni_revenue_of_customer|
| --------|----------|-----------|--------|--------------------|---------------------- |
|Female	|Weekday	|1541781.47	|725916	|15278	|95.322304|
|Female	|Weekend	|985978.53	|474065	|9884	|93.745690|
|Male	|Weekday	|545806.31	|246549	|5961	|90.266569|
|Male	|Weekend	|472764.34	|219686	|5077	|91.730576|
|Unkown	|Weekday	|6360.86	|2686	|79	|79.708704|
|Unkown	|Weekend	|3742.00	|1744	|46	|81.347826|

#### *Notice: There are five weekdays and only two weekends，therefore, add three columnsL: revenue_per_day, profit_per_day, customer_per_day*
```python
uni_gen.loc[uni_gen['wkd_ind'] =='Weekday', 'revenue_per_day'] = uni_gen['revenue']/5
uni_gen.loc[uni_gen['wkd_ind'] =='Weekend', 'revenue_per_day'] = uni_gen['revenue']/2

uni_gen.loc[uni_gen['wkd_ind'] =='Weekday', 'profit_per_day'] = uni_gen['profit']/5
uni_gen.loc[uni_gen['wkd_ind'] =='Weekend', 'profit_per_day'] = uni_gen['profit']/2

uni_gen.loc[uni_gen['wkd_ind'] =='Weekday', 'customer_per_day'] = uni_gen['customer']/5
uni_gen.loc[uni_gen['wkd_ind'] =='Weekend', 'customer_per_day'] = uni_gen['customer']/2

uni_gen
```
|gender_group |wkd_ind	|revenue  	|profit	|uni_quant_of_order	|uni_revenue_of_customer|revenue_per_day|	profit_per_day	|customer_per_day|
| --------|----------|-----------|--------|----------|----------|----------|-------|------ |
|Female	|Weekday	|1541781.47	|725916	|15278	|95.322304|308356.294	|145183.2	|3055.6|
|Female	|Weekend	|985978.53	|474065	|9884	|93.745690|492989.265	|237032.5	|4942.0|
|Male	|Weekday	|545806.31	|246549	|5961	|90.266569|109161.262	|49309.8	|1192.2|
|Male	|Weekend	|472764.34	|219686	|5077	|91.730576|236382.170	|109843.0	|2538.5|
|Unkown	|Weekday	|6360.86	|2686	|79	|79.708704|1272.172	|537.2	|15.8|
|Unkown	|Weekend	|3742.00	|1744	|46	|81.347826|1871.000	|872.0	|23.0|

#### Visualization
```python
# sales
sns.barplot(x = 'gender_group', y = 'revenue_per_day', hue = 'wkd_ind', data = uni_gen)
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/1.5.png)
```python
# profit
sns.barplot(x = 'gender_group', y = 'profit_per_day', hue = 'wkd_ind', data = uni_gen)
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/1.6.png)

```python
# number of customers
sns.barplot(x = 'gender_group', y = 'customer_per_day', hue = 'wkd_ind', data = uni_gen)
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/1.7.png)

```python
# per capita consumption
sns.barplot(x = 'gender_group', y = 'uni_revenue_of_customer', hue = 'wkd_ind', data = uni_gen)
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/1.8.png)

## Conclusion
- **Female customers bring in far more daily sales and profits than male customers**
- **For male customers: weekend daily sales/profits are almost twice as much as weekday daily sales/profit**
- **The number of customers on weekends was much higher than that in the middle of the week, regardless of the gender of customers, but there was no significant difference in terms of per capita consumption.**



## Question 2. Sales performance of different categories/products 
+ Variables to use?
   - revenue, profit, quant, product

+ What data relationships are presented?    
   - Compare sales performance of different products
   - Analyze from sales, profit, volume and other dimensions
   - Loss-making product analysis

+ What kind of charts?    
   - Bar chart, pie chart

### A. Performance of difference products
```
uni_product=uni_clean.groupby(['product']).agg({'revenue':np.sum, 'profit': np.sum, 'quant':np.sum}).reset_index().sort_values('revenue',ascending=False)
uni_product
```
| |product	|revenue	|profit	|quant|
|--|--------|--------|--------|-----|
|0	|T-shirt	|1538744.84	|636130	|18425|
|1	|Season	|590664.88	|275915	|5338|
|8	|Accessories	|444685.15	|310695	|4622|
|3	|Jeans	|246127.48	|78328	|2432|
|2	|Sweater	|245630.80	|111427	|1356||
|6	|Dress	|137302.78	|78617	|995|
|5	|Socks	|127731.36	|95154	|3639|
|7	|Sports	|118060.34	|30299	|1792|
|4	|Shorts	|107485.88	|54081	|2821|

#### Visualization
```python
# volume
result = uni_clean.groupby(["product"])['quant'].agg(np.sum).reset_index().sort_values('quant',ascending=False)
sns.barplot(x = 'product', y = 'quant', data = uni_product, order=result['product'])
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/2.1.png)
```python
# sales & profit
# set width of bar
barWidth = 0.25
 
# Set position of bar on X axis
r1 = np.arange(len(uni_product['revenue']))
r2 = [x + barWidth for x in r1]
 
# Make the plot
plt.bar(r1, uni_product['revenue'], color='b', width=barWidth, edgecolor='white', label='revenue')
plt.bar(r2, uni_product['profit'], color='r', width=barWidth, edgecolor='white', label='profit')
 
# Add xticks on the middle of the group bars
plt.xlabel('group', fontweight='bold')
plt.xticks([r + barWidth for r in range(len(uni_product['revenue']))], uni_product['product'])
 
# Create legend & Show graphic
plt.legend()
plt.show()
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/2.2.png)

## Conclusion
- **In terms of volumes, sales and profits, T-shirts performed much better than other products, followed by Seasonal products and Accessories**

## *Therefore, let's take a deeper look at T-shirts.*
```
# Sales volume by gender
uni_t_gender = uni_clean[uni_clean['product']=='T恤'].groupby(['product','gender_group'])['quant'].sum().reset_index()
uni_t_gender
```

|product|	gender_group|	quant|
| --------|----------|---------|
|T恤	|Female	|13109|
|T恤	|Male	|5253|
|T恤	|Unkown	|63|

```
colors = ['lightcoral', 'lightskyblue', 'yellowgreen']
explode = (0.1, 0, 0)  # explode 1st slice

# Plot a pie chart
plt.pie(uni_t_gender['quant'], explode=explode, labels=uni_t_gender['gender_group'], colors=colors,
autopct='%1.1f%%', shadow=True, startangle=140)

plt.axis('equal')
plt.show()
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/2.3.png)

## Conclusion
- **Women account for 70 percent of T-shirt sales volume.**



### B. Loss-making product analysis
```
plt.rcParams['axes.unicode_minus']=False
```
```
# histogram of product profit
uni_clean['uni_profit_of_product'] = uni_clean['profit']/uni_clean['quant']
sns.distplot(uni_clean['uni_profit_of_product'], bins=20)
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/2.4.png)
```
# products that are lossing money
uni_loss = uni_clean[uni_clean['uni_profit_of_product']<0]
sort = uni_loss.groupby(["product"])['profit'].agg(np.sum).reset_index().sort_values('profit')
plt.figure(figsize=(10,5))
sns.barplot(x = 'product', y = 'profit',data = sort, order=sort['product'],palette="pastel")
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/2.5.png)

#### Take a deeper look into Jeans
```
# groupby gender
uni_jean = uni_clean[(uni_clean['product']=='牛仔裤') & (uni_clean['profit']<0) ].groupby(['gender_group'])['profit'].sum().reset_index()
uni_jean
```
|gender_group	|profit|
|-----|-----|
|Female	|-22975|
|Male	|-5137|
|Unkown	|-180|
```
colors = ['lightcoral', 'lightskyblue', 'yellowgreen']
explode = (0.1, 0, 0)  # explode 1st slice

# Plot a pie chart
plt.pie(-1*uni_jean['profit'], explode=explode, labels=uni_jean['gender_group'], colors=colors,
autopct='%1.1f%%', shadow=True, startangle=140)

plt.axis('equal')
plt.show()
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/2.6.png)

## Conclusion
- **The most loss-making product was jeans; followed by T-shirts, Sports and Seasonal products. 80% of the sales loss came from women.**


## Question 3. Which purchase channel is the most popular? 
+ Variables to use?
   - channel, gender_group, age_group, city

+ What data relationships are presented?    
   - Analyze channel preference among people of different gender, age and from different cities.

+ What kind of charts?    
   - Bar chart
   
#### seaborn color palette: https://seaborn.pydata.org/tutorial/color_palettes.html   
```
uni_channel_gender = uni_clean.groupby(['gender_group','channel'])['customer'].sum().reset_index().sort_values('customer',ascending=False)
uni_channel_gender
sns.barplot(x='gender_group',y='customer',hue='channel',data=uni_channel_gender, palette="Paired")
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/3.1.png)
```
age_order = ['<20','20-24','25-29',  '30-34','35-39',  '40-44','45-49', '50-54','55-59','>=60', 'Unkown']
uni_channel_age = uni_clean.groupby(['age_group','channel'])['customer'].sum().reset_index()
uni_channel_age
sns.barplot(x='age_group',y='customer',hue='channel',data=uni_channel_age, order=age_order, palette="Paired")
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/3.2.png)
```
uni_channel_city = uni_clean.groupby(['city','channel'])['customer'].sum().reset_index().sort_values('customer',ascending=False)
uni_channel_city
sns.barplot(x='city',y='customer',hue='channel',data=uni_channel_city, palette="Paired")
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/3.3.png)

## Conclusion
- **Customers of different genders prefer offline purchase.**
- **The majority of the customers are between 20 and 40 years old, and they all prefer to purchase Uniqlo offline.**
- **Only customers in Wuhan, Guangzhou, Shanghai, Xi'an and Chongqing will buy online. Customers in Guangzhou prefer to buy online, while customers in Wuhan have no obvious preference. Customers in other cities prefer to buy offline.**


## Question 4. What's the relationship between sales and COG?
+ Variables to use?
   - revenue，cost

+ What data relationships are presented?    
   - The correlation between sales and costs.

+ What kind of charts?    
   - Thermal diagram/scatter plot

```
uni_clean['uni_revenue_of_product'] = uni_clean['revenue']/uni_clean['quant']
uni_clean['uni_profit_of_product'] = uni_clean['profit']/uni_clean['quant']
cor = uni_clean[['uni_revenue_of_product','uni_profit_of_product','unit_cost','unit_price']].corr()
cor
```

|  |uni_revenue_of_product	|uni_profit_of_product	|unit_cost	|unit_price|
|--|---------------------|------------------------|-----------|---------|
|uni_revenue_of_product	|1.000000	|0.911672	|0.502248	|0.999996|
|uni_profit_of_product	|0.911672	|1.000000	|0.102566	|0.911735|
|unit_cost	|0.502248	|0.102566	|1.000000	|0.502125|
|unit_price	|0.999996	|0.911735	|0.502125	|1.000000|
```
sns.heatmap(cor)
```
![image](https://github.com/cassiezy/Sales_Analysis_Uniqlo/blob/master/pic/4.1.png)

## Conclusion
- **There is a positive correlation between unit sales and cost, with a correlation coefficient of 0.5.**
- **The sales is strongly positively correlated with the profit, and the correlation coefficient is 0.91.**
- **The correlation between the cost of goods and profit is very low, only 0.1; However, the price of goods and profits are highly correlated, with a correlation coefficient of 0.91.**


# Final takeways and recommendations
### 1. Offline average daily sales and profits are much higher than online, but there is no big difference in terms of the number of products per order and per capita consumption.
### 2. Female customers bring in far more daily sales and profits than male customers.
### 3. Sales and profits on weekends are much higher than those at weekdays.
### 4. The number of customers on weekends was much higher than that in the middle of the week, regardless of the gender of customers, but there was no significant difference in terms of per capita consumption.
   *__Recommendation:__ Try more appealing in-store visual merchandising and piece matching to attract more female customers at weekends.*
### 5. Product performance:
   - The best-selling item was T-shirts, with 70 percent of sales coming from women. The biggest loss was in jeans, with 80 per cent coming from women.
### 6. Channel preference:
   - The majority of the customers are between 20 and 40 years old, and both genders prefer to purchase Uniqlo offline.
   - Only customers in Wuhan, Guangzhou, Shanghai, Xi'an and Chongqing purchase online. Customers in Guangzhou prefer to buy online, while customers in other cities prefer to buy offline. 
   - *__Recommendation:__ Different online marketing strategies could be adopted for Guangzhou and Wuhan.*
