# Case Study #4 - Data Bank

## A. Customer Nodes Exploration

1. How many unique nodes are there on the Data Bank system?

        SELECT 
        	COUNT(DISTINCT CAST(region_id AS VARCHAR) || CAST(node_id AS VARCHAR)) AS unique_nodes
        FROM 
        	customer_nodes;

| unique_nodes |
| ------------ |
| 25           |

2. What is the number of nodes per region?

        SELECT 
        	r.region_name,
            COUNT(cn.node_id) AS nodes_count
        FROM 
        	customer_nodes cn
        LEFT JOIN
        	regions r ON cn.region_id = r.region_id
        GROUP BY r.region_name;

| region_name | nodes_count |
| ----------- | ----------- |
| America     | 735         |
| Australia   | 770         |
| Africa      | 714         |
| Asia        | 665         |
| Europe      | 616         |

3. How many customers are allocated to each region?

        SELECT 
        	r.region_name,
            COUNT(DISTINCT cn.customer_id) AS customer_count
        FROM 
        	customer_nodes cn
        LEFT JOIN
        	regions r ON cn.region_id = r.region_id
        GROUP BY r.region_name;

| region_name | customer_count |
| ----------- | -------------- |
| Africa      | 102            |
| America     | 105            |
| Asia        | 95             |
| Australia   | 110            |
| Europe      | 88             |

4. How many days on average are customers reallocated to a different node?

        SELECT 
        	MAX(end_date) AS max_end_date
        FROM 
        	customer_nodes;

| max_end_date             |
| ------------------------ |
| 9999-12-31T00:00:00.000Z |



        SELECT 
        	ROUND(AVG(end_date - start_date),0) AS avg_days
        FROM 
        	customer_nodes
        WHERE
            end_date != '9999-12-31';

| avg_days |
| -------- |
| 15       |

5. What is the median, 80th and 95th percentile for this same reallocation days metric for each region?

        SELECT 
        	region_id,
            PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY end_date - start_date) AS median_relocation_days,
            PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY end_date - start_date) AS percentile_80_relocation_days,
            PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY end_date - start_date) AS percentile_95_relocation_days
        FROM 
            customer_nodes
        WHERE
            end_date != '9999-12-31'
        GROUP BY 
            region_id
        ORDER BY 
            region_id;

| region_id | median_relocation_days | percentile_80_relocation_days | percentile_95_relocation_days |
| --------- | ---------------------- | ----------------------------- | ----------------------------- |
| 1         | 15                     | 23                            | 28                            |
| 2         | 15                     | 23                            | 28                            |
| 3         | 15                     | 24                            | 28                            |
| 4         | 15                     | 23                            | 28                            |
| 5         | 15                     | 24                            | 28                            |

## B. Customer Transactions

1. What is the unique count and total amount for each transaction type?

        SELECT 
        	txn_type,
            COUNT(txn_type) AS total_txn,
            SUM(txn_amount) AS total_amt
        FROM customer_transactions
        GROUP BY txn_type;

| txn_type   | total_txn | total_amt |
| ---------- | --------- | --------- |
| purchase   | 1617      | 806537    |
| deposit    | 2671      | 1359168   |
| withdrawal | 1580      | 793003    |

2. What is the average total historical deposit counts and amounts for all customers?

        SELECT
            ROUND(AVG(total_txn)) AS avg_txn,
            ROUND(AVG(avg_amt),2) AS avg_amt
        FROM (
        SELECT 
        	ct.customer_id,
            COUNT(ct.customer_id) AS total_txn,
            AVG(ct.txn_amount) AS avg_amt
        FROM customer_transactions ct
        WHERE ct.txn_type = 'deposit'
        GROUP BY ct.customer_id) as txn_details;

| avg_txn | avg_amt |
| ------- | ------- |
| 5       | 508.61  |

3. For each month - how many Data Bank customers make more than 1 deposit and either 1 purchase or 1 withdrawal in a single month?

        WITH monthly_txn AS (
            SELECT
                customer_id,
                EXTRACT(MONTH FROM txn_date) AS txn_month,
          		SUM(CASE WHEN txn_type = 'deposit' THEN 1 ELSE 0 END) AS deposit,
          		SUM(CASE WHEN txn_type = 'purchase' THEN 1 ELSE 0 END) AS purchase,
          		SUM(CASE WHEN txn_type = 'withdrawal' THEN 1 ELSE 0 END) AS withdrawal
            FROM 
          		customer_transactions
          	GROUP BY
          		customer_id,
          		txn_month
        )
        
        SELECT
            COUNT(customer_id)
        FROM 
            monthly_txn
        WHERE deposit >= 1 AND (purchase >= 1 OR withdrawal >= 1);

| count |
| ----- |
| 1060  |

4. What is the closing balance for each customer at the end of the month?

Showing first 5.


        WITH all_customer_months AS (
            SELECT DISTINCT
                customer_id,
                generate_series(
                    CAST(MIN(EXTRACT(MONTH FROM txn_date)) AS INT),
                    CAST(MAX(EXTRACT(MONTH FROM txn_date)) AS INT),
                    1
                ) AS txn_month
            FROM 
                customer_transactions
          	GROUP BY
          		customer_id
        )
        
        , monthly_txn AS (
            SELECT
                acm.customer_id,
                acm.txn_month,
                COALESCE(SUM(CASE WHEN ct.txn_type = 'deposit' THEN ct.txn_amount ELSE 0 END), 0) AS deposit,
                COALESCE(SUM(CASE WHEN ct.txn_type = 'purchase' THEN ct.txn_amount ELSE 0 END), 0) AS purchase,
                COALESCE(SUM(CASE WHEN ct.txn_type = 'withdrawal' THEN ct.txn_amount ELSE 0 END), 0) AS withdrawal
            FROM 
                all_customer_months acm
            LEFT JOIN 
                customer_transactions ct ON acm.customer_id = ct.customer_id AND acm.txn_month = EXTRACT(MONTH FROM ct.txn_date)
            GROUP BY
                acm.customer_id,
                acm.txn_month
        )
        
        , monthly_closing_amt AS (
            SELECT
                customer_id,
                txn_month,
                deposit,
                purchase,
                withdrawal,
                (deposit - purchase - withdrawal) AS total_monthly
            FROM 
                monthly_txn
        )
        
        , monthly_closing_balance AS (
            SELECT
                customer_id,
                txn_month,
                COALESCE(LAG(CAST(total_monthly AS INT), 1, 0) OVER (PARTITION BY customer_id ORDER BY txn_month), 0) AS prev_closing_balance,
                COALESCE((LAG(CAST(total_monthly AS INT), 1, 0) OVER (PARTITION BY customer_id ORDER BY txn_month)), 0) + deposit - purchase - withdrawal AS closing_balance
            FROM 
                monthly_closing_amt
        )
        
        SELECT 
            *
        FROM 
            monthly_closing_balance;

| customer_id | txn_month | prev_closing_balance | closing_balance |
| ----------- | --------- | -------------------- | --------------- |
| 1           | 1         | 0                    | 312             |
| 1           | 2         | 312                  | 312             |
| 1           | 3         | 0                    | -952            |
| 2           | 1         | 0                    | 549             |
| 2           | 2         | 549                  | 549             |
| 2           | 3         | 0                    | 61              |
| 3           | 1         | 0                    | 144             |
| 3           | 2         | 144                  | -821            |
| 3           | 3         | -965                 | -1366           |
| 3           | 4         | -401                 | 92              |
| 4           | 1         | 0                    | 848             |
| 4           | 2         | 848                  | 848             |
| 4           | 3         | 0                    | -193            |
| 5           | 1         | 0                    | 954             |
| 5           | 2         | 954                  | 954             |
| 5           | 3         | 0                    | -2877           |
| 5           | 4         | -2877                | -3367           |

5. What is the percentage of customers who increase their closing balance by more than 5%?


## C. Data Allocation Challenge
To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

Option 1: data is allocated based off the amount of money at the end of the previous month
Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days
Option 3: data is updated real-time
For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

running customer balance column that includes the impact each transaction
customer balance at the end of each month
minimum, average and maximum values of the running balance for each customer
Using all of the data available - how much data would have been required for each option on a monthly basis?

## D. Extra Challenge
Data Bank wants to try another option which is a bit more difficult to implement - they want to calculate data growth using an interest calculation, just like in a traditional savings account you might have with a bank.

If the annual interest rate is set at 6% and the Data Bank team wants to reward its customers by increasing their data allocation based off the interest calculated on a daily basis at the end of each day, how much data would be required for this option on a monthly basis?

Special notes:

Data Bank wants an initial calculation which does not allow for compounding interest, however they may also be interested in a daily compounding interest calculation so you can try to perform this calculation if you have the stamina!

## Extension Request
The Data Bank team wants you to use the outputs generated from the above sections to create a quick Powerpoint presentation which will be used as marketing materials for both external investors who might want to buy Data Bank shares and new prospective customers who might want to bank with Data Bank.

Using the outputs generated from the customer node questions, generate a few headline insights which Data Bank might use to market it’s world-leading security features to potential investors and customers.

With the transaction analysis - prepare a 1 page presentation slide which contains all the relevant information about the various options for the data provisioning so the Data Bank management team can make an informed decision.
