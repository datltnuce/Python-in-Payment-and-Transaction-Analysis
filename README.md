# 📊 Payment and Transaction Analysis
Author: Dat Le Tien  
Date: 2025-02-15  
Tools Used: Python  

---

## 📑 Table of Contents  
1. [📌 Background & Overview](#-background--overview)  
2. [📂 Dataset](#-dataset)
3. [⚒️ Main Process](#-main-process)
4. [📊 Key Insights & Visualizations](#-key-insights--visualizations)  

---

## 📌 Background & Overview  

### 📖 What is this project about? 
To analyze payment statuses and transaction trends within an e-wallet system to identify performance patterns, detect anomalies, and uncover opportunities for optimization, ultimately enhancing efficiency, security, and user experience.  

### 👤 Who is this project for?  
✔️ Finance & Risk Management Team  
✔️ Operations & Product Team  
✔️ Marketing & Business Strategy Team

###  ❓Business Questions:  
✔️ Which products, teams, or categories are performing well or underperforming?<br>
✔️ Are there any unusual transaction patterns or anomalies that need investigation?<br> 
✔️ How do refund transactions impact overall revenue and user behavior?<br>
✔️ What are the statistics on transaction volume, number of senders, and receivers?

### 🎯Project Outcome:  

✔️ Identification of high and low-performing products, teams, or categories.<br>
✔️ Detection of anomalies and fraudulent activities to enhance security.<br>
✔️ Insights into refund trends and their impact on revenue.<br>
✔️ Improved transaction efficiency and user experience through data-driven decisions. 

---

## 📂 Dataset   
1. payment_report.csv (monthly payment volume of products)
2. product.csv (product information)
3. transactions.csv (transactions information)

Type of transaction:
- transType = 2 & merchant_id = 1205: Bank Transfer Transaction
- transType = 2 & merchant_id = 2260: Withdraw Money Transaction
- transType = 2 & merchant_id = 2270: Top Up Money Transaction
- transType = 2 & others merchant_id: Payment Transaction
- transType = 8, merchant_id = 2250: Transfer Money Transaction
- transType = 8 & others merchant_id: Split Bill Transaction
- Remained cases are invalid transactions

---

## ⚒️ Main Process
1️⃣ Exploratory Data Analysis<br> 
Do EDA task:
- Df payment_enriched (Merge payment_report.csv with product.csv)
- Df transactions

Suggestions:
- Check each column: missing data? duplicates? incorrect data types?
    - Sample Answers:
        - Missing data: x rows in column A, y rows in column B -> Next step: No action/ Delete rows/…
        - Duplicates: PK? x rows? -> Next step: No action/ Delete rows/…
        - Incorrect data types: column A, column B -> Next step: No action/ Delete rows/…
        - Incorrect values: column A, column B -> Next step: No action/ Delete rows/…
- Summarize numerical data: any incorrect values?
```python
# View all rows of the payment_report DataFrame
payment_report.head(1000)

# View the first few of the product DataFrame
product.head(5)

# View the first few of the transactions DataFrame
transactions.head()

# Merge the payment_report and product DataFrames
payment_enriched = pd.merge(payment_report, product, on='product_id', how='left')
payment_enriched.head()

# Missing data checking:
payment_enriched.info()
transactions.info()

payment_enriched.duplicated().sum()
transactions.duplicated().sum()

payment_enriched.describe()
transactions.describe()

# Drop missing data in payment_enriched
payment_enriched.dropna(inplace=True)
```
2️⃣ Data Wrangling<br>
- Check Top 3 `product_id` with the highest transaction volume.
```python
# Top 3 product_id:
top_3_products = payment_enriched.groupby('product_id')['volume'].sum().sort_values(ascending=False).head(3)
print(top_3_products)
```
- Check if there are any product_id violating the rule that each product_id belongs to only one team_own.
```python
# Check for any product_id that is associated with more than one team_own:
abnormal_products = payment_enriched.groupby('product_id')['team_own'].nunique()
abnormal_products = abnormal_products[abnormal_products > 1]
print(abnormal_products)
```
- Identify the team with the lowest performance since Q2 2023 and determine the category that contributes the least to the team's performance.
```python
# Since Q2.2023 data:
payment_enriched['report_month'] = pd.to_datetime(payment_enriched['report_month'])
q2_2023 = payment_enriched[payment_enriched['report_month'] >= '2023-04-01']
# The team has had the lowest performance:
lowest_team = q2_2023.groupby('team_own')['volume'].sum().idxmin()
# The category that contributes the least to that team
lowest_team_data = q2_2023[q2_2023['team_own'] == lowest_team]
lowest_category = lowest_team_data.groupby('category')['volume'].sum().idxmin()
print("Lowest Team:", lowest_team)
print("Lowest Category for the Lowest Team:", lowest_category)
```
- Analyze the contribution percentage of source_id in refund transactions and find the source_id with the highest contribution.
```python
# Filter refund transactions:
refunds = payment_enriched[payment_enriched['payment_group'] == 'refund']
print(refunds.head())
# Total refund volume:
source_contribution = refunds.groupby('source_id')['volume'].sum().sort_values(ascending=False)
print(source_contribution)
# Largest source_id:
largest_source = source_contribution.idxmax()
print("Largest Source ID:", largest_source)
```
- Calculate the number of transactions, total volume, the number of senders, and the number of receivers.
```python
# Find the number of transactions, volume, senders and receivers
def define_transaction_type(row):
    if row['transType'] == 2:
        if row['merchant_id'] == 1205:
            return 'Bank Transfer Transaction'
        elif row['merchant_id'] == 2260:
            return 'Withdraw Money Transaction'
        elif row['merchant_id'] == 2270:
            return 'Top Up Money Transaction'
        else:
            return 'Payment Transaction'
    elif row['transType'] == 8:
        if row['merchant_id'] == 2250:
            return 'Transfer Money Transaction'
        else:
            return 'Split Bill Transaction'
    else:
        return 'Invalid transactions'

transactions['transaction_type'] = transactions.apply(define_transaction_type, axis=1)
valid_transactions = transactions[transactions['transaction_type'] != 'Invalid transactions']

transaction_summary = valid_transactions.groupby('transaction_type').agg({
    'transaction_id': 'count',
    'volume': 'sum',
    'sender_id': 'nunique',
    'receiver_id': 'nunique'
}).rename(columns={'transaction_id': 'total_transactions'})
print(transaction_summary)
```
---

## 📊 Key Insights & Visualizations  
**1. User profile Distribution** 
```python
# Distribution of User profile
segment_by_user_count = RFM_transactions_final[['Segment', 'CustomerID']].groupby(['Segment']).count().reset_index().rename(columns={'CustomerID': 'user_volume'})
segment_by_user_count['contribution_percent'] = round(segment_by_user_count['user_volume'] / segment_by_user_count['user_volume'].sum() * 100, 2)
segment_by_user_count['type'] = 'user contribution'

segment_by_spending = RFM_transactions_final[['Segment', 'Monetery']].groupby(['Segment']).sum().reset_index().rename(columns={'Monetery': 'spending'})
segment_by_spending['contribution_percent'] = segment_by_spending['spending'] / segment_by_spending['spending'].sum() * 100
segment_by_spending['type'] = 'spending contribution'

segment_agg = pd.concat([segment_by_user_count, segment_by_spending])

# Visualize the distribution of User profile
plt.figure(figsize=(15, 8))
sns.barplot(data=segment_agg, x='Segment', y='contribution_percent', hue='type')
plt.title('The overall distribution of user profile using RFM Model')
plt.xticks(rotation=45)
plt.show()
```
![Image](https://github.com/user-attachments/assets/a0905645-3452-4f80-aa87-bc164766a3cd)

**Data Analysis:**<br>

Group: At Risk Customer and Cannot Lose Them Customer are the two most important customer groups as they account for a large proportion of both volume and revenue. However, these customers haven't used the product for a long time, showing signs of declining interest and high risk of churning.<br>

Suggestions:<br>
- Special promotion campaigns: Target these customers with exclusive offers to motivate them to return.<br>
- Personalized notifications: Send relevant notifications, highlighting product value, or announce new features to revive interest.<br>
- Feedback surveys: Investigate reasons for declining engagement to develop appropriate solutions.

Group: Loyal, New Customer, Potential Loyalist, and Promising make up a large number of customers. However, their transaction value is low, leading to limited overall revenue contribution. This group has development potential if properly stimulated.<br>

Summary:<br>
- Cross-selling and up-selling strategies: Introduce complementary products/services to increase their transaction value.<br>
- Encourage consumption through loyalty programs: Implement point accumulation policies, discounts when reaching certain spending thresholds.<br>
- Enhanced interaction: Use email, marketing campaign notifications to build long-term relationships.

**2. Frequency Distribution** 
```python
# Distribution of Frequency
binsF = [0, 2, 5, 20, np.inf]
labelsF = ['1-2', '2-5', '5-20', '20+']
RFM_transactions['FrequencyGroup'] = pd.cut(RFM_transactions['Frequency'], bins=binsF, labels=labelsF)
fig, ax = plt.subplots(figsize=(8, 3))
sns.countplot(x='FrequencyGroup', data=RFM_transactions, ax=ax)
ax.set_title('Distribution of Frequency')
ax.yaxis.set_visible(False)
ax.spines['left'].set_visible(False)
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)

# Visualize the distribution of Frequency
for container in ax.containers:
    ax.bar_label(container, label_type='edge', padding=2)
plt.show()
```
![Image](https://github.com/user-attachments/assets/f74320fc-c323-4f92-b561-05c91d01037a)

**Data Analysis:**
- Group 1-2 times (138 customers):<br>
This is the group with the lowest number of customers.<br>
These may be customers who made trial purchases or did not return after their first purchase.

- Group 2-5 times (169 customers):<br>
The number of customers increases slightly compared to the 1-2 times group.<br>
This group may consist of potential customers who are not yet loyal or regular.

- Group 5-20 times (969 customers):<br>
This is the second-largest group, accounting for a significant proportion.<br>
Customers in this group have relatively high purchase frequency and can be considered loyal customers or those with increasing frequency trends.

- Group 20+ times (3,096 customers):<br>
This is the largest group, dominating the distribution.<br>
Customers in this group are considered loyal or bring the highest value to the business.

**Summary:** <br>
The reason for dividing the bins this way is:

- Helps identify potential customers and groups with the highest revenue potential.
- Supports marketing strategies and loyalty programs based on purchasing frequency.

The business has a large number of regular customers (20+ group) with very high purchase frequency, which is a positive sign. However, the number of customers in the 1-2 and 2-5 times groups is quite low. The business should consider strategies to convert customers with lower frequency (groups 1-2 and 2-5) into higher frequency groups (5-20 and 20+).<br>

Can implement promotional programs, customer care, or special offers to encourage more frequent shopping.

**3. Distribution throughout the time**
```python
# RFM Distribution throughout the time and visualize
transactions['YearMonth'] = transactions['InvoiceDate'].dt.to_period('M')
rfm_time = transactions.groupby(['YearMonth', 'CustomerID']).agg({
    'InvoiceDate': 'max',
    'InvoiceNo': 'count',
    'cost': 'sum'
}).reset_index()

rfm_time.columns = ['YearMonth', 'CustomerID', 'Recency', 'Frequency', 'Monetary']
rfm_time['Recency'] = (last_day - rfm_time['Recency']).dt.days

# Define the rfm_segment function
def rfm_segment(row):
    if row['Recency'] <= 30 and row['Frequency'] >= 10 and row['Monetary'] >= 1000:
        return 'Champions'
    elif row['Recency'] <= 90 and row['Frequency'] >= 5 and row['Monetary'] >= 500:
        return 'Loyal Customers'
    elif row['Recency'] <= 180 and row['Frequency'] >= 3 and row['Monetary'] >= 300:
        return 'Potential Loyalists'
    else:
        return 'Others'

# Customer segmentation over time
rfm_time['Segment'] = rfm_time.apply(rfm_segment, axis=1)

# Calculate the number of customers in each segment over time
rfm_distribution = rfm_time.groupby(['YearMonth', 'Segment']).size().reset_index(name='Count')

# Convert YearMonth to string for plotting
rfm_distribution['YearMonth'] = rfm_distribution['YearMonth'].astype(str)

# Visualize the distribution of RFM segments over time
plt.figure(figsize=(15, 8))
sns.lineplot(data=rfm_distribution, x='YearMonth', y='Count', hue='Segment')
plt.title('RFM Distribution Throughout the Time')
plt.xticks(rotation=45)
plt.show()
```
![Image](https://github.com/user-attachments/assets/14cd2735-ed1e-4de7-ab43-ca0bae1fefcb)
**Data Analysis:** 
- Others Group (Blue):<br>
Largest group in early 2011, but customer numbers decreased significantly from mid-2011.<br>
Peak occurred around January 2011, then sharply declined approaching 0 by year-end.

- Potential Loyalists Group (Orange):<br>
Started increasing from June 2011, showing successful conversion of "Others" into potentially loyal customers.<br>
However, numbers declined from October 2011 to year-end.

- Loyal Customers Group (Green):<br>
Emerged clearly from mid-2011 with slight growth from July 2011 to October 2011.<br>
Subsequently, loyal customer numbers decreased towards year-end, indicating need for retention measures.

- Champions Group (Red):<br>
Latest to emerge, from October 2011, but showed slight decline in following months.<br>
This is a crucial customer group, but relatively small in size, highlighting need for retention and development focus.

**Summary:** <br>
- Graph shows customer migration from "Others" to higher-value groups like "Potential Loyalists", "Loyal Customers", and "Champions". However, from October 2011 onwards, all customer groups showed declining trends, especially the "Others" group.<br>
- Business needs to focus on improving customer retention strategies, particularly for "Champions" and "Loyal Customers" groups, as these provide the highest value. 

---
