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
**1. The trend of total transaction volume by day** 
```python
# Convert timeStamp to datetime
transactions['timeStamp'] = pd.to_datetime(transactions['timeStamp'], unit='ms')

# Aggregate transactions by date to observe trends
transactions['date'] = transactions['timeStamp'].dt.date

# Visualize the trend of total transaction volume by day
plt.figure(figsize=(12, 5))
daily_trend = transactions.groupby('date')['volume'].sum()
sns.lineplot(x=daily_trend.index, y=daily_trend.values, marker='o', color='b')

plt.xticks(rotation=45)
plt.xlabel("Date")
plt.ylabel("Total Transaction Volume")
plt.title("Transaction Volume Trend Over Time")
plt.grid(True)
plt.show()
```
![Image](https://github.com/user-attachments/assets/ee3d72ce-d1c7-470b-80f6-59c8433e067d)

Insight:
- Transactions surged on May 1st and 2nd, 2023, then showed a downward trend from May 3rd onwards.
- A significant drop occurred between May 3rd and 4th, possibly due to an anomaly or system change.
- Towards the end of the observed period (May 6th - 7th, 2023), transaction volume continued to decline slightly.

Recommendation:
- Investigate the reason behind the sharp increase on May 1st - 2nd: It could be due to a promotional campaign or an event that attracted more transactions.
- Analyze the cause of the sharp decline from May 3rd: Verify if there were system issues, transaction errors, or other factors affecting e-wallet usage.
- Strengthen transaction-stimulating campaigns over the weekends: If a decline is observed towards the end of the period, consider offering incentives to maintain stable transaction volumes.

Summary: 
- Transactions spiked at the beginning of May due to unidentified factors, followed by a sharp decline. Further investigation is needed, along with potential incentives to sustain stable transaction volumes.

**2. Transaction count by hour and day of the week** 
```python
# Convert problematic columns to numeric, forcing errors to NaN
transactions['sender_id'] = pd.to_numeric(transactions['sender_id'], errors='coerce')
transactions['receiver_id'] = pd.to_numeric(transactions['receiver_id'], errors='coerce')

# Heatmap of the correlation matrix without displaying numbers
plt.figure(figsize=(10, 8))
numeric_cols = transactions.select_dtypes(include=['number'])
corr_matrix = numeric_cols.corr()

# Heatmap of transaction count by hour and day of the week without displaying numbers
heatmap_data = transactions.pivot_table(index='day_of_week', columns='hour', values='transaction_id', aggfunc='count')

# Reorder days of the week
heatmap_data = heatmap_data.reindex(days_order)

# Plot heatmap
plt.figure(figsize=(12, 6))
sns.heatmap(heatmap_data, cmap="YlGnBu", linewidths=0.5, annot=False)
plt.xlabel("Hour of the Day")
plt.ylabel("Day of the Week")
plt.title("Heatmap of Transactions by Hour and Day of the Week")
plt.show()
```
![Image](https://github.com/user-attachments/assets/0dadeddc-ee67-4ca9-9c17-28a991f6dd3f)

Insights:
1. Transactions peak at night and early morning (12 AM - 6 AM)
- The highest transaction volumes occur between 12 AM - 6 AM, especially on Monday to Wednesday.
- This could be due to automated system transactions or users preferring to conduct online transactions at night.
2. Sharp decline in transactions in the evening and late night
- From around 5 PM - 11 PM, transaction volume significantly drops, particularly midweek and on weekends.
- Wednesday and Thursday show a noticeable drop after 4 PM.
3. More balanced transaction activity on weekends
- Transactions remain relatively stable throughout Saturday and Sunday without any significant peaks.
- This could indicate that customers have more free time on weekends to conduct transactions.

Recommendations:
1. Optimize system performance at night
- Since transaction volume is high between 12 AM - 6 AM, ensure the system has sufficient resources to handle transactions efficiently and avoid network congestion or system errors.
- Consider scheduling system maintenance during low-transaction periods, such as 5 PM - 11 PM.
2. Increase marketing campaigns in the evening
- Since transaction volume drops from 5 PM - 11 PM, introduce promotional campaigns or discounts during this period to attract more users.
- Example: “10% transaction fee discount from 6 PM - 10 PM” or “Cashback promotions for evening transactions.”
3. Focus on weekend transaction strategies
- Since weekend transactions are more evenly distributed, businesses can introduce exclusive weekend promotions to further boost activity.
- Identify the key customer segments transacting on weekends to offer tailored promotions.

Summary:
- This heatmap highlights peak and low transaction hours, helping optimize system performance, improve marketing strategies, and enhance customer experience.
---
