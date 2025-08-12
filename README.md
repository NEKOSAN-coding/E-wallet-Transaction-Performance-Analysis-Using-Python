# 📱E-wallet Transaction Performance Analysis Using Python



**Author:** Hà Minh Khuê

**Date:** 05/08/2025

**Tools Used:** Python

## 📑 Table of Contents

1. [📖 Background & Overview](#background--overview)
2. [📂 Dataset Description & Data Structure](#dataset-description--data-structure)
3. [🔎 Final Conclusion & Recommendations](#final-conclusion--recommendations)


## 📖 Background & Overview

### Objective:
This project aims to analyze transaction and payment data within an e-wallet system to:

✔️ Understand payment and transaction trends.  
✔️ Detect anomalies in product ownership and team performance.  
✔️ Identify key contributors to refunds and transaction volume.  
✔️ Categorize transactions into meaningful types for further analysis.  

### 👤 Who is this project for?

✔️ Business Analysts and Data Analysts  
✔️ Senior Manager & stakeholders in the fintech industry  
✔️ Product teams optimizing e-wallet services  

## 📂 Dataset Description & Data Structure

### 📌 Data Source
- **Source**: Internal company database (e-wallet transaction records)  
- **Size**:  
  - `product.csv`: 493 rows × 3 columns  
  - `payment_report.csv`: 920 rows × 5 columns  
  - `transactions.csv`: 1,324,002 rows × 9 columns  
- **Format**: `.csv`  

---

### 📊 Data Structure & Relationships

#### 1️⃣ Tables Used:
This project utilizes **three** datasets:

1. **product.csv** – Product details, including category and team ownership.  
2. **payment_report.csv** – Monthly payment volume and source tracking.  
3. **transactions.csv** – Comprehensive transaction records.  

**Key Relationships:**
- `product_id` links **product.csv** and **payment_report.csv**.  
- `transaction_id` is the unique key in **transactions.csv** for tracking payments.  
- `source_id` in **payment_report.csv** contributes to refund analysis.  

---

#### 2️⃣ Table Schema & Data Snapshot

##### Table 1: Products Table (`product.csv`) - *493 rows, 3 columns*

👉🏻 **Schema:**
| Column Name  | Data Type | Description |
|-------------|----------|-------------|
| product_id  | INT      | Unique identifier for each product |
| category    | TEXT     | Product category |
| team_own    | TEXT     | Team responsible for the product |

---

##### Table 2: Payment Report (`payment_report.csv`) - *920 rows, 5 columns*

👉🏻 **Schema:**
| Column Name    | Data Type | Description |
|---------------|----------|-------------|
| report_month  | DATE     | Month of the payment report |
| payment_group | TEXT     | Type of payment (e.g., refund, purchase) |
| product_id    | INT      | Associated product ID |
| source_id     | INT      | Source of the transaction |
| volume        | FLOAT    | Total payment volume |

---

##### Table 3: Transactions (`transactions.csv`) - *1,324,002 rows, 9 columns*

👉🏻 **Schema:**
| Column Name    | Data Type | Description |
|---------------|----------|-------------|
| transaction_id | INT      | Unique transaction identifier |
| merchant_id    | INT      | Merchant involved in the transaction |
| volume         | FLOAT    | Transaction amount |
| transType      | INT      | Type of transaction |
| transStatus    | TEXT     | Status of the transaction (e.g., completed, failed) |
| sender_id      | INT      | Sender of the transaction |
| receiver_id    | INT      | Receiver of the transaction |
| extra_info     | TEXT     | Additional details about the transaction |
| timeStamp      | TIMESTAMP | Time when the transaction occurred |

---

## ⚒️ Main Process

### 1️⃣ Data Cleaning & EDA 

#### Step 1. Import library and working with dataset

```python
import pandas as pd
df_payment_report = pd.read_csv('/content/drive/MyDrive/Colab Notebooks/Python project 2/payment_report.csv')
df_product = pd.read_csv('/content/drive/MyDrive/Colab Notebooks/Python project 2/product.csv')
df_transactions = pd.read_csv('/content/drive/MyDrive/Colab Notebooks/Python project 2/transactions.csv')
df_payment_enriched = df_payment_report.merge(df_product, how = 'left', on = 'product_id')
```

#### Step 2. EDA df_payment_enriched

```python
missing_data_df_payment_enriched = df_payment_enriched.isnull().sum()
print(missing_data_df_payment_enriched)

duplicate_rows_df_payment_enriched = df_payment_enriched.duplicated().sum()
print(duplicate_rows_df_payment_enriched)

df_payment_enriched.info()

df_payment_enriched.describe()
```

<img width="398" height="616" alt="image" src="https://github.com/user-attachments/assets/88692b08-6f92-420a-821a-2c076688a1fc" />

#### Step 3. EDA df_transactions

```python
missing_data_df_transactions = df_transactions.isnull().sum()
print(missing_data_df_transactions)

duplicate_rows_df_transactions = df_transactions.duplicated().sum()
print(duplicate_rows_df_transactions)

df_transactions.info()

df_transactions.describe()
```

<img width="857" height="681" alt="image" src="https://github.com/user-attachments/assets/7691090e-f2a0-484d-b631-977341f28596" />

### 2️⃣ Python Analysis


#### ✅ Find the top 3 products with the highest revenue

Find the top 3 products with the highest revenue to identify which products are generating the most income. This helps in focusing on the highest-performing products for sales and marketing strategies.

[In 1]:

```python
#1.
df_payment_enriched['report_month'] = pd.to_datetime(df_payment_enriched['report_month'])
df_transactions.drop_duplicates(subset=['transaction_id'], inplace=True)
#top3product:
top_3_products = df_payment_enriched.groupby('product_id')['volume'].sum().sort_values(ascending = False).head(3)
print(top_3_products)
```

[Out 1]:

<img width="670" height="83" alt="image" src="https://github.com/user-attachments/assets/96078db8-ba60-461f-804a-2fa8a8fc40f6" />

#### ✅ Verify whether each product is managed by only one team

Check whether each product is managed by a single team to ensure clear ownership and avoid potential conflicts in responsibility.

[In 2]:

```python
#2.
product_team_counts = df_payment_enriched.groupby('product_id')['team_own'].nunique()
abnormal_products = product_team_counts[product_team_counts > 1]

if not abnormal_products.empty:
    print("Các product_id bất thường (thuộc sở hữu của hơn 1 team):")
    print(abnormal_products)
else:
    print("Không tìm thấy product_id bất thường (mỗi product_id thuộc sở hữu của 1 team).")
```

[Out 2]:

<img width="648" height="35" alt="image" src="https://github.com/user-attachments/assets/9359c56d-8432-47ad-bb3e-702eaa08bcfe" />

#### ✅ Find the team has had the lowest performance (lowest volume) since Q2.2023. Find the category that contributes the least to that team.

Identify the team with the lowest performance in terms of total transaction volume since Q2.2023. Once the lowest-performing team is found, determine the product category that contributes the least to that team's total volume. This analysis helps pinpoint underperforming teams and categories, providing insights for potential improvements or strategic decisions.

[In 3]:

```python
#3.
lowest_performance_team = df_payment_enriched.groupby('team_own')['volume'].sum().idxmin()
print(lowest_performance_team)
q2_2023_start = pd.to_datetime('2023-04-01')
q2_2023_end = pd.to_datetime('2023-06-30')
df_q2_2023 = df_payment_enriched[(df_payment_enriched['report_month'] >= q2_2023_start) & (df_payment_enriched['report_month'] <= q2_2023_end)] #chỉ lấy dữ liệu

team_volumes = df_q2_2023.groupby('team_own')['volume'].sum()
lowest_performing_team = team_volumes.idxmin()
lowest_team_volume = team_volumes.min()
print(f"Team có hiệu suất thấp nhất kể từ Q2.2023: {lowest_performing_team} với tổng khối lượng {lowest_team_volume:,}.")

if lowest_performing_team and pd.notna(lowest_performing_team):
    df_lowest_team = df_q2_2023[df_q2_2023['team_own'] == lowest_performing_team]
    category_contributions = df_lowest_team.groupby('category')['volume'].sum()
    category_contributions = category_contributions.dropna() # Loại bỏ NaN category

    if not category_contributions.empty:
        least_contributing_category = category_contributions.idxmin()
        least_category_volume = category_contributions.min()
        print(f"Danh mục đóng góp ít nhất cho team {lowest_performing_team}: {least_contributing_category} với khối lượng {least_category_volume:,}.")
    else:
        print(f"Không tìm thấy danh mục nào cho team {lowest_performing_team} trong giai đoạn này.")
else:
    print("Không thể xác định team có hiệu suất thấp nhất.")
```

[Out 3]:

<img width="548" height="29" alt="image" src="https://github.com/user-attachments/assets/80f2ab94-3b7f-40a8-a269-6199a206c32d" />

<img width="576" height="45" alt="image" src="https://github.com/user-attachments/assets/eaac83a6-fab1-4a4b-88f9-6e25fd2153db" />

#### ✅ Determine the contribution of source_ids in refund transactions (where payment_group = 'refund') and identify the source_id with the highest contribution.

The goal is to analyze the distribution of refund transactions across different source_ids and identify which source_id has the highest contribution. This helps in understanding refund patterns and potential areas for improvement.

[In 4]:

```python
refund_transactions = df_payment_enriched[df_payment_enriched['payment_group'] == 'refund']
source_id_refund_volume = refund_transactions.groupby('source_id')['volume'].sum()
if not source_id_refund_volume.empty:
    total_refund_volume = source_id_refund_volume.sum()
    source_id_contributions = (source_id_refund_volume / total_refund_volume * 100).sort_values(ascending=False)
    print(f"Source_id có đóng góp cao nhất vào giao dịch hoàn tiền: {highest_contribution_source_id} ({highest_contribution_percentage}%)")
else:
    print("Không tìm thấy giao dịch hoàn tiền nào.")
```

[Out 4]:

<img width="598" height="40" alt="image" src="https://github.com/user-attachments/assets/cd3622c8-c590-43db-bcc6-350ffa851e84" />

#### ✅ Define the transaction type for each row based on the given conditions:

- **For transType = 2 and merchant_id = 1205:** Bank Transfer Transaction
- **For transType = 2 and merchant_id = 2260:** Withdraw Money Transaction
- **For transType = 2 and merchant_id = 2270:** Top Up Money Transaction
- **For transType = 2 and other merchant_id:** Payment Transaction
- **For transType = 8 and merchant_id = 2250:** Transfer Money Transaction
- **For transType = 8 and other merchant_id:** Split Bill Transaction
- **All remaining cases** are considered **invalid transactions**.

The purpose of this task is to categorize each transaction into a specific transaction type based on the given conditions. By doing so, we can clearly identify the type of each transaction in the dataset, which helps in analyzing and understanding transaction patterns, processing specific transactions, and detecting any anomalies or invalid transactions. 

[In 5]:

```python
#5.
def define_transaction_type(row):
    if row['transType'] == 2 and row['merchant_id'] == 1205:
        return 'Bank Transfer Transaction'
    elif row['transType'] == 2 and row['merchant_id'] == 2260:
        return 'Withdraw Money Transaction'
    elif row['transType'] == 2 and row['merchant_id'] == 2270:
        return 'Top Up Money Transaction'
    elif row['transType'] == 2: # transType = 2 & others merchant_id
        return 'Payment Transaction'
    elif row['transType'] == 8 and row['merchant_id'] == 2250:
        return 'Transfer Money Transaction'
    elif row['transType'] == 8: # transType = 8 & others merchant_id
        return 'Split Bill Transaction'
    else:
        return 'Invalid Transaction'

df_transactions['transaction_type'] = df_transactions.apply(define_transaction_type, axis=1)
```

#### ✅ Of each transaction type (excluding invalid transactions): find the number of transactions, volume, senders and receivers

This helps in understanding the distribution of transactions across different types, as well as the engagement of unique senders and receivers for each type of transaction.

[In 6]:
```python
#6.
df_valid_transactions = df_transactions[df_transactions['transaction_type'] != 'Invalid Transaction']
transaction_summary = df_valid_transactions.groupby('transaction_type').agg(
    number_of_transactions=('transaction_id', 'count'),
    total_volume=('volume', 'sum'),
    unique_senders=('sender_id', lambda x: x.dropna().nunique()),
    unique_receivers=('receiver_id', lambda x: x.dropna().nunique())
)
print(transaction_summary)
```

[Out 6]:

<img width="570" height="270" alt="image" src="https://github.com/user-attachments/assets/7175daff-add5-4a51-825d-1dae782be512" />

## 🔎 Final Conclusion & Recommendations

### 🔗 Findings:

1. **Top 3 Products with Highest Volume**:
   - **Product ID 1976**: 61,797,583,647
   - **Product ID 429**: 14,667,676,567
   - **Product ID 372**: 13,713,658,515

2. **Team Ownership of Products**:
   - No abnormal products (each product is handled by one team).

3. **Lowest Performance Team in Q2.2023**:
   - **Team APS** with volume = 51,141,753
   - **Category PXXXXXE** contributes 25,232,438 volume.

4. **Highest Contributor in Refund Transactions**:
   - **Source ID 38** contributes 59.11% of refund volume.

5. **Transaction Types**:
   - **Bank Transfer**: 37,879 transactions, volume = 50.6B
   - **Payment**: 398,677 transactions, volume = 71.85B
   - **Split Bill**: 1,376 transactions, volume = 4.9M
   - **Top Up**: 290,502 transactions, volume = 108.6B
   - **Transfer**: 341,177 transactions, volume = 37B
   - **Withdraw**: 33,725 transactions, volume = 23.42B

---

### 📈 Recommendations:

1. **Product Focus**: 
   - Prioritize **Product ID 1976** as it drives the most volume.
   - Investigate ways to boost **Product ID 429** and **Product ID 372**, which has potential to increase volume.

2. **Team APS**:
   - APS has the lowest performance in Q2.2023. Investigate and optimize their operations, especially in the **PXXXXXE category**.

3. **Refunds**:
   - **Source ID 38** is the biggest contributor to refunds. Investigate potential issues related to this source to reduce refund volume.

4. **Transaction Optimization**:
   - Focus on improving the **Payment** and **Top Up** transactions as they have the highest volume.
   - Consider increasing engagement for **Split Bill Transactions** to boost volume.

