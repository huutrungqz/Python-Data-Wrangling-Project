# Data Wrangling Project

```python
import pandas as pd
import numpy as np

product_url = "https://drive.google.com/uc?id=11jvH4R2ntNmWWofuD-qebHmyWOHtG0nu"
payment_report_url = "https://drive.google.com/uc?id=1hqx4-3cFerSElpij1YWgGusWVtldvrBr"
transactions_url = "https://drive.google.com/uc?id=1c1nCpuACsJBklJ9YpoiihATj9JkiLJFM"

product = pd.read_csv(product_url)
payment_report = pd.read_csv(payment_report_url)
transactions = pd.read_csv(transactions_url)
```


Given dataset
Suppose you are a DA in an e-wallet company, and you need to analyze the following datasets:
payment_report.csv (monthly payment volume of products)
product.csv (product information)
transactions.csv (transactions information)

**Statement: Muốn hiểu về tình trạng payment hoặc transaction trong context của một ewallet.**

**Part I: EDA - Explore Data Analysis**
Do EDA task:
- Df payment_enriched (Merge payment_report.csv with product.csv)
- Df transactions
Suggestions:
1. Check each column: missing data? duplicates? incorrect data types?
2. Summarize numerical data: any incorrect values?

Sample Answers:
- Incorrect data types: column A, column B -> Next step: No action/ Delete rows/…
- Incorrect values: column A, column B -> Next step: No action/ Delete rows/…
- Missing data: x rows in column A, y rows in column B -> Next step: No action/ Delete rows/…
- Duplicates: PK? x rows? -> Next step: No action/ Delete rows/…

**Part II: Data Wrangling**

Using payment_report.csv & product.csv
1. Top 3 product_ids with the highest volume.
2. Given that 1 product_id is only owed by 1 team, are there any abnormal products against this rule?
3. Find the team has had the lowest performance (lowest volume) since Q2.2023 Find the category that contributes the least to that team.
4. Find the contribution of source_ids of refund transactions (payment_group = ‘refund’), what is the source_id with the highest contribution?

Using transactions.csv

5. Define type of transactions (‘transaction_type’) for each row, given:
- transType = 2 & merchant_id = 1205: Bank Transfer Transaction
- transType = 2 & merchant_id = 2260: Withdraw Money Transaction
- transType = 2 & merchant_id = 2270: Top Up Money Transaction
- transType = 2 & others merchant_id: Payment Transaction
- transType = 8, merchant_id = 2250: Transfer Money Transaction
- transType = 8 & others merchant_id: Split Bill Transaction
- Remained cases are invalid transactions
6. Of each transaction type (excluding invalid transactions): find the number of transactions, volume, senders and receivers.



---

**Part I: EDA - Explore Data Analysis**
Do EDA task:
- Df payment_enriched (Merge payment_report.csv with product.csv)
- Df transactions
Suggestions:
1. Check each column: missing data? duplicates? incorrect data types?
2. Summarize numerical data: any incorrect values?

Sample Answers:
- Incorrect data types: column A, column B -> Next step: No action/ Delete rows/…
- Incorrect values: column A, column B -> Next step: No action/ Delete rows/…
- Missing data: x rows in column A, y rows in column B -> Next step: No action/ Delete rows/…
- Duplicates: PK? x rows? -> Next step: No action/ Delete rows/…


```python
# Df payment_enriched (Merge payment_report.csv with product.csv)

payment_enriched = payment_report.merge(product, on = 'product_id', validate= 'many_to_one', how = 'left')
```


```python
# Df transactions Suggestions:
# Check general info

print(transactions.info())
print(transactions.head())
```


```python
transactions_remove_duplicate = transactions.drop_duplicates()  #remove duplicate rows
print(transactions_remove_duplicate.info())
```


```python
print(transactions_remove_duplicate.isnull().sum())   #Check null values
```


```python
transactions_new = transactions_remove_duplicate.drop(columns = 'extra_info')  #Remove 'extra_info' column
print(transactions_new.info())
```


```python
transactions_new[['sender_id','receiver_id']] = transactions_new[['sender_id','receiver_id']].fillna(0).astype('int') # Fillna =0 in sender_id & receive_id and change type to integer
print(transactions_new.info())
```


---

**Part II: Data Wrangling**

Using payment_report.csv & product.csv
1. Top 3 product_ids with the highest volume.
2. Given that 1 product_id is only owed by 1 team, are there any abnormal products against this rule?
3. Find the team has had the lowest performance (lowest volume) since Q2.2023 Find the category that contributes the least to that team.
4. Find the contribution of source_ids of refund transactions (payment_group = ‘refund’), what is the source_id with the highest contribution?


```python
print(payment_enriched.info())
print(payment_enriched.head())
```


```python
print(payment_enriched.groupby('product_id')['volume'].sum().sort_values(ascending = False).head(3))  # Top 3 product_ids with the highest volume.

```


```python
# Given that 1 product_id is only owed by 1 team, are there any abnormal products against this rule?

groupteam = payment_enriched.groupby('product_id')['team_own'].nunique().sort_values(ascending = False) # Check if any produt owed by more than 1 team -> no product
print(groupteam[groupteam.index.isin([1976, 429, 372])])

#-> Product_id = 1976 has highest Volume but does not have any team_own
```


```python
#Find the team has had the lowest performance (lowest volume) since Q2.2023 Find the category that contributes the least to that team
payment_enriched['report_date'] = pd.to_datetime(payment_enriched['report_month'])

print(payment_enriched[payment_enriched['report_date'] >= '2023-04-01'].groupby('team_own')['volume'].sum().sort_values(ascending = True).head(1))

# -> Team APS has the lowest volume since Q2.2023

print(payment_enriched[(payment_enriched['team_own'] == 'APS') & (payment_enriched['report_date'] >= '2023-04-01')].groupby('category')['volume'].sum().sort_values(ascending = True).head(1))

# -> category 'PXXXXXS' contributes the least to team APS
```


```python
# Find the contribution of source_ids of refund transactions (payment_group = ‘refund’), what is the source_id with the highest contribution?

print(payment_enriched[payment_enriched['payment_group'] == 'refund'].groupby('source_id')['volume'].sum().sort_values(ascending = False).head(1))

# -> source_id 38 has the highest contribution within payment_group 'refund'


```


Using transactions.csv

5. Define type of transactions (‘transaction_type’) for each row, given:
- transType = 2 & merchant_id = 1205: Bank Transfer Transaction
- transType = 2 & merchant_id = 2260: Withdraw Money Transaction
- transType = 2 & merchant_id = 2270: Top Up Money Transaction
- transType = 2 & others merchant_id: Payment Transaction
- transType = 8, merchant_id = 2250: Transfer Money Transaction
- transType = 8 & others merchant_id: Split Bill Transaction
- Remained cases are invalid transactions
6. Of each transaction type (excluding invalid transactions): find the number of transactions, volume, senders and receivers.


```python
print(transactions_new.head())
print(transactions_new.isnull().sum())
```


```python
condition = [((transactions_new['transType'] == 2) & (transactions_new['merchant_id'] == 1205)), \
             ((transactions_new['transType'] == 2) & (transactions_new['merchant_id'] == 2260)), \
             ((transactions_new['transType'] == 2) & (transactions_new['merchant_id'] == 2270)), \
             ((transactions_new['transType'] == 2) & (~transactions_new['merchant_id'].isin([1205, 2260, 2270]))), \
             ((transactions_new['transType'] == 8) & (transactions_new['merchant_id'] == 2250)), \
             ((transactions_new['transType'] == 8) & (transactions_new['merchant_id'] != 2250))]

result = ['Bank Transfer Transaction', 'Withdraw Money Transaction', 'Top Up Money Transaction', 'Payment Transaction', 'Transfer Money Transaction', 'Split Bill Transaction']
```


```python
# 5. Define type of transactions (‘transaction_type’) for each row

transactions_new['transaction_type'] = np.select(condition, result, default = 'Invalid Transaction')
print(transactions_new['transaction_type'].value_counts(ascending = False))
```


```python
# 6. Of each transaction type (excluding invalid transactions): find the number of transactions, volume, senders and receivers.
# -> count transaction_id, sum volume, count unique sender_id, count unique receiver_id

transactions_new[transactions_new['transaction_type'] != 'Invalid Transaction'].groupby('transaction_type').agg(total_transaction =('transaction_id','count'),
                                                                                                                total_volume = ('volume','sum'),
                                                                                                                total_unique_sender = ('sender_id','nunique'),
                                                                                                                total_unique_receiver = ('receiver_id','nunique'))
```


```python

```

