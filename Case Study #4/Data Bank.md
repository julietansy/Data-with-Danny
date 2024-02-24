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
        GROUP BY
               r.region_name;
        ORDER BY
               r.region_name;

| region_name | nodes_count |
| ----------- | ----------- |
| Africa      | 714         |
| America     | 735         |
| Asia        | 665         |
| Australia   | 770         |
| Europe      | 616         |

3. How many customers are allocated to each region?

        SELECT 
        	r.region_name,
               COUNT(DISTINCT cn.customer_id) AS customer_count
        FROM 
        	customer_nodes cn
        LEFT JOIN
        	regions r ON cn.region_id = r.region_id
        GROUP BY
               r.region_name
        ORDER BY
               r.region_name;
   

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
                    r.region_name,
                    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY cn.end_date - cn.start_date) AS median_relocation_days,
                    PERCENTILE_CONT(0.8) WITHIN GROUP (ORDER BY cn.end_date - cn.start_date) AS percentile_80_relocation_days,
                    PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY cn.end_date - cn.start_date) AS percentile_95_relocation_days
           FROM 
                    customer_nodes cn
           LEFT JOIN
                    regions r ON cn.region_id = r.region_id
           WHERE
                   cn.end_date != '9999-12-31'
           GROUP BY
                    r.region_name
           ORDER BY
                    r.region_name;

| region_name | median_relocation_days | percentile_80_relocation_days | percentile_95_relocation_days |
| ----------- | ---------------------- | ----------------------------- | ----------------------------- |
| Africa      | 15                     | 24                            | 28                            |
| America     | 15                     | 23                            | 28                            |
| Asia        | 15                     | 23                            | 28                            |
| Australia   | 15                     | 23                            | 28                            |
| Europe      | 15                     | 24                            | 28                            |

## B. Customer Transactions

1. What is the unique count and total amount for each transaction type?

        SELECT 
               txn_type,
               COUNT(txn_type) AS transaction_count,
               SUM(txn_amount) AS transaction_amt
        FROM
               customer_transactions
        GROUP BY
               txn_type;

| txn_type   | transaction_count | transaction_amt |
| ---------- | ----------------- | --------------- |
| purchase   | 1617              | 806537          |
| deposit    | 2671              | 1359168         |
| withdrawal | 1580              | 793003          |


2. What is the average total historical deposit counts and amounts for all customers?

        SELECT
            ROUND(AVG(total_txn)) AS avg_txn,
            ROUND(AVG(avg_amt),2) AS avg_amt
        FROM
           (
            SELECT 
               ct.customer_id,
               COUNT(ct.customer_id) AS total_txn,
               AVG(ct.txn_amount) AS avg_amt
            FROM customer_transactions ct
            WHERE ct.txn_type = 'deposit'
            GROUP BY ct.customer_id
           ) as txn_details;

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
        WHERE
               deposit >= 1 AND (purchase >= 1 OR withdrawal >= 1)
        GROUP BY
                txn_month
        ORDER BY
                txn_month;

| txn_month | count |
| --------- | ----- |
| 1         | 295   |
| 2         | 298   |
| 3         | 329   |
| 4         | 138   |

4. What is the closing balance for each customer at the end of the month?

Showing first 5.

    WITH all_customer_months AS (
        SELECT DISTINCT
            customer_id,
            txn_month
        FROM
            customer_transactions
        CROSS JOIN (
            SELECT
                generate_series(
                    CAST(MIN(EXTRACT(MONTH FROM txn_date)) AS INT),
                    CAST(MAX(EXTRACT(MONTH FROM txn_date)) AS INT),
                    1
                ) AS txn_month
            FROM
                customer_transactions
        ) AS all_months
    ),
    cus_balance AS (
        SELECT
            ct.customer_id,
            acm.txn_month,
            COALESCE(
                SUM(
                    (CASE WHEN ct.txn_type = 'deposit' THEN ct.txn_amount ELSE 0 END) -
                    (CASE WHEN ct.txn_type = 'purchase' THEN ct.txn_amount ELSE 0 END) -
                    (CASE WHEN ct.txn_type = 'withdrawal' THEN ct.txn_amount ELSE 0 END)
                ),
                0
            ) AS monthly_txn
        FROM
            all_customer_months acm
        LEFT JOIN
            customer_transactions ct ON acm.customer_id = ct.customer_id
                                        AND acm.txn_month = EXTRACT(MONTH FROM ct.txn_date)
        GROUP BY
            ct.customer_id,
            acm.txn_month
        ORDER BY
            ct.customer_id,
            acm.txn_month
    ),
    closing_balance AS (
        SELECT
            acm.customer_id,
            acm.txn_month,
            SUM(cb.monthly_txn) OVER (PARTITION BY acm.customer_id ORDER BY acm.txn_month) AS closing_balance
        FROM
            all_customer_months acm
        LEFT JOIN
            cus_balance cb ON acm.customer_id = cb.customer_id AND acm.txn_month = cb.txn_month
    )
    SELECT *
    FROM closing_balance;

| customer_id | txn_month | closing_balance |
| ----------- | --------- | --------------- |
| 1           | 1         | 312             |
| 1           | 2         | 312             |
| 1           | 3         | -640            |
| 1           | 4         | -640            |
| 2           | 1         | 549             |
| 2           | 2         | 549             |
| 2           | 3         | 610             |
| 2           | 4         | 610             |
| 3           | 1         | 144             |
| 3           | 2         | -821            |
| 3           | 3         | -1222           |
| 3           | 4         | -729            |
| 4           | 1         | 848             |
| 4           | 2         | 848             |
| 4           | 3         | 655             |
| 4           | 4         | 655             |
| 5           | 1         | 954             |
| 5           | 2         | 954             |
| 5           | 3         | -1923           |
| 5           | 4         | -2413           |


5. What is the percentage of customers who increase their closing balance by more than 5%?


        WITH all_customer_months AS (
            SELECT DISTINCT
                customer_id,
                txn_month
            FROM
                customer_transactions
            CROSS JOIN (
                SELECT
                    generate_series(
                        CAST(MIN(EXTRACT(MONTH FROM txn_date)) AS INT),
                        CAST(MAX(EXTRACT(MONTH FROM txn_date)) AS INT),
                        1
                    ) AS txn_month
                FROM
                    customer_transactions
            ) AS all_months
        ),
        cus_monthly_txn AS (
            SELECT 
                ct.customer_id,
                acm.txn_month,
                COALESCE(SUM((CASE WHEN ct.txn_type = 'deposit' THEN ct.txn_amount ELSE 0 END) - 
                              (CASE WHEN ct.txn_type = 'purchase' THEN ct.txn_amount ELSE 0 END) - 
                              (CASE WHEN ct.txn_type = 'withdrawal' THEN ct.txn_amount ELSE 0 END)), 0) AS monthly_txn
            FROM 
                all_customer_months acm
            LEFT JOIN
                customer_transactions ct on acm.customer_id = ct.customer_id AND acm.txn_month = EXTRACT(MONTH FROM ct.txn_date)
            GROUP BY 
                ct.customer_id, 
                acm.txn_month
            ORDER BY
                ct.customer_id, 
                acm.txn_month
        ),
        closing_balance AS (
            SELECT
                acm.customer_id,
                acm.txn_month,
                SUM(cmt.monthly_txn) OVER (PARTITION BY acm.customer_id ORDER BY acm.txn_month) AS closing_balance
            FROM 
                all_customer_months acm
            LEFT JOIN
                cus_monthly_txn cmt on acm.customer_id = cmt.customer_id AND acm.txn_month = cmt.txn_month	
        ),
        cus_growth AS (
            SELECT 
                *,
                COALESCE((LAG(closing_balance) OVER (PARTITION BY customer_id ORDER BY txn_month)), 0) AS prev_closing_balance
            FROM closing_balance
        ),
        balance_summary AS ( 
            SELECT 
                c.customer_id, 
                (
                    SELECT p.closing_balance
                    FROM cus_growth p 
                    WHERE p.customer_id = c.customer_id AND p.txn_month = 1
                    LIMIT 1
                ) AS opening_balance,
                c.closing_balance
            FROM 
                cus_growth c
            WHERE 
                c.txn_month = 3
        )
        
        SELECT 
            ROUND(((SELECT COUNT(customer_id) * 100.00
                    FROM balance_summary
                    WHERE (closing_balance * 100.00 / opening_balance) > 105.00) / COUNT(customer_id)), 2) AS growth_percentage_5
        FROM balance_summary;

| growth_percentage_5 |
| ------------------- |
| 45.00               |

## C. Data Allocation Challenge
To test out a few different hypotheses - the Data Bank team wants to run an experiment where different groups of customers would be allocated data using 3 different options:

Option 1: data is allocated based off the amount of money at the end of the previous month

Option 2: data is allocated on the average amount of money kept in the account in the previous 30 days

Option 3: data is updated real-time

For this multi-part challenge question - you have been requested to generate the following data elements to help the Data Bank team estimate how much data will need to be provisioned for each option:

- running customer balance column that includes the impact each transaction

            SELECT customer_id,
                   txn_date,
                   txn_type,
                   txn_amount,
                   SUM(CASE WHEN txn_type = 'deposit' THEN txn_amount
            		WHEN txn_type = 'withdrawal' THEN -txn_amount
            		WHEN txn_type = 'purchase' THEN -txn_amount
            		ELSE 0
            	   END) OVER(PARTITION BY customer_id ORDER BY txn_date) AS running_balance
            FROM customer_transactions;

| customer_id | txn_date                 | txn_type   | txn_amount | running_balance |
| ----------- | ------------------------ | ---------- | ---------- | --------------- |
| 1           | 2020-01-02T00:00:00.000Z | deposit    | 312        | 312             |
| 1           | 2020-03-05T00:00:00.000Z | purchase   | 612        | -300            |
| 1           | 2020-03-17T00:00:00.000Z | deposit    | 324        | 24              |
| 1           | 2020-03-19T00:00:00.000Z | purchase   | 664        | -640            |
| 2           | 2020-01-03T00:00:00.000Z | deposit    | 549        | 549             |
| 2           | 2020-03-24T00:00:00.000Z | deposit    | 61         | 610             |
| 3           | 2020-01-27T00:00:00.000Z | deposit    | 144        | 144             |
| 3           | 2020-02-22T00:00:00.000Z | purchase   | 965        | -821            |
| 3           | 2020-03-05T00:00:00.000Z | withdrawal | 213        | -1034           |
| 3           | 2020-03-19T00:00:00.000Z | withdrawal | 188        | -1222           |
| 3           | 2020-04-12T00:00:00.000Z | deposit    | 493        | -729            |
| 4           | 2020-01-07T00:00:00.000Z | deposit    | 458        | 458             |
| 4           | 2020-01-21T00:00:00.000Z | deposit    | 390        | 848             |
| 4           | 2020-03-25T00:00:00.000Z | purchase   | 193        | 655             |


- customer balance at the end of each month
- minimum, average and maximum values of the running balance for each customer

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
