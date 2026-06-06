# Data Wrangling Project — E-wallet Transaction Analysis

## Overview

This project focuses on exploring, cleaning, and analyzing transaction data from an e-wallet company using Python specifically Pandas and Numpy package.

The project includes:

- Exploratory Data Analysis (EDA)
- Data Cleaning
- Data Wrangling
- Business Insights
- Transaction Classification
- Aggregation & Reporting

---

# Technologies Used

- Python
- Pandas
- NumPy
- Google Colab
- Jupyter Notebook

---

# Part I — Exploratory Data Analysis (EDA)

## Question

Perform EDA tasks on:

- `payment_enriched`
- `transactions`

Tasks include:

- Merge datasets
- Check missing values
- Check duplicates
- Check data types
- Remove unnecessary columns

---

## Solution

### Import Libraries & Load Data

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

---

### For payment_report.csv & product.csv
### Merge Dataset

```python
payment_enriched = payment_report.merge(product, on = 'product_id', validate= 'many_to_one', how = 'left')
```

---

### Check General Information

```python
print(transactions.info())
print(transactions.head())
```

---

### For transactions.csv
### Remove Duplicate Rows

```python
transactions_remove_duplicate = transactions.drop_duplicates()

print(transactions_remove_duplicate.info())
```

---

### Check Missing Values

```python
print(transactions_remove_duplicate.isnull().sum())
```

---

### Remove Unnecessary Column

```python
transactions_new = transactions_remove_duplicate.drop(columns='extra_info')

print(transactions_new.info())
```

---

### Fill Missing Values & Change Data Type

```python
transactions_new[['sender_id', 'receiver_id']] = (
    transactions_new[['sender_id', 'receiver_id']]
    .fillna(0)
    .astype('int')
)

print(transactions_new.info())
```

---

# Part II — Data Wrangling

---

# Question 1

Find the top 3 `product_id` with the highest volume.

## Solution

```python
print(
    payment_enriched
    .groupby('product_id')['volume']
    .sum()
    .sort_values(ascending=False)
    .head(3)
)
```

---

# Question 2

Given that 1 `product_id` is only owned by 1 team, identify abnormal products violating this rule.

## Solution

```python
groupteam = payment_enriched.groupby('product_id')['team_own'].nunique().sort_values(ascending = False) # Check if any produt owed by more than 1 team -> no product
print(groupteam[groupteam.index.isin([1976, 429, 372])])
```

---

# Question 3

Find the team with the lowest performance (lowest volume) since Q2 2023.

Then identify the category contributing the least to that team.

## Solution

```python
payment_enriched['report_date'] = pd.to_datetime(payment_enriched['report_month'])

print(payment_enriched[payment_enriched['report_date'] >= '2023-04-01'].groupby('team_own')['volume'].sum().sort_values(ascending = True).head(1))

# -> Team APS has the lowest volume since Q2.2023

print(payment_enriched[(payment_enriched['team_own'] == 'APS') & (payment_enriched['report_date'] >= '2023-04-01')].groupby('category')['volume'].sum().sort_values(ascending = True).head(1))

# -> category 'PXXXXXS' contributes the least to team APS
```

---

# Question 4

Find the contribution of `source_id` in refund transactions (`payment_group = 'refund'`).

Which `source_id` contributes the highest volume?

## Solution

```python
print(payment_enriched[payment_enriched['payment_group'] == 'refund'].groupby('source_id')['volume'].sum().sort_values(ascending = False).head(1))

# -> source_id 38 has the highest contribution within payment_group 'refund'

```

---

# Question 5: 

Define transaction types (`transaction_type`) using the following conditions:

| Condition | Transaction Type |
|---|---|
| `transType = 2` & `merchant_id = 1205` | Bank Transfer Transaction |
| `transType = 2` & `merchant_id = 2260` | Withdraw Money Transaction |
| `transType = 2` & `merchant_id = 2270` | Top Up Money Transaction |
| `transType = 1` & `merchant_id = 2250` | Payment Transaction |
| `transType = 4` | Split Bill Transaction |
| Remaining rows | Invalid Transaction |
---

## Solution

```python
condition = [
    (
        (transactions_new['transType'] == 2) &
        (transactions_new['merchant_id'] == 1205)
    ),

    (
        (transactions_new['transType'] == 2) &
        (transactions_new['merchant_id'] == 2260)
    ),

    (
        (transactions_new['transType'] == 2) &
        (transactions_new['merchant_id'] == 2270)
    ),

    (
        (transactions_new['transType'] == 1) &
        (transactions_new['merchant_id'] == 2250)
    ),

    (
        transactions_new['transType'] == 4
    )
]

result = [
    'Bank Transfer Transaction',
    'Withdraw Money Transaction',
    'Top Up Money Transaction',
    'Payment Transaction',
    'Split Bill Transaction'
]
```

---

### Create `transaction_type`

```python
transactions_new['transaction_type'] = np.select(condition, result, default = 'Invalid Transaction')
print(transactions_new['transaction_type'].value_counts(ascending = False))
```

---

# Question 6

For each transaction type (excluding invalid transactions), find:

- Number of transactions
- Total volume
- Number of unique senders
- Number of unique receivers

---

## Solution

```python
transactions_new[transactions_new['transaction_type'] != 'Invalid Transaction']
                          .groupby('transaction_type').agg(total_transaction =('transaction_id','count'),
                                                            total_volume = ('volume','sum'),
                                                            total_unique_sender = ('sender_id','nunique'),
                                                            total_unique_receiver = ('receiver_id','nunique'))
```

---

# Skills Practiced

- Data Cleaning
- Exploratory Data Analysis
- Data Wrangling
- GroupBy Aggregation
- Conditional Logic
- Transaction Classification
- Business Analysis
- Pandas Data Manipulation

---

# Learning Outcomes

Through this project, I practiced:

- Cleaning raw datasets
- Handling missing values
- Removing duplicates
- Merging datasets
- Building business metrics
- Creating transaction classification logic
- Performing analytical reporting

---

# How to Run

1. Open the notebook in Google Colab

2. Upload the `.ipynb` file

3. Run all cells sequentially

---
