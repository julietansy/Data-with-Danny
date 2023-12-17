# Case Study #3: Foodie-Fi

![3](https://github.com/julietansy/Data-with-Danny/assets/151416878/ec10b7cc-08e5-4552-a08f-0959fd2a3037)

[Source Link](https://8weeksqlchallenge.com/case-study-3/)

## Business Case
Foodie-Fi, launched in 2020, is a niche streaming service dedicated solely to food-related content, similar to Netflix but exclusively for cooking shows and culinary programs. The platform offers monthly and annual subscriptions, granting unlimited on-demand access to a diverse range of exclusive food videos sourced globally.

Danny, the founder, established Foodie-Fi with a data-centric approach, intending to base all business decisions, investments, and feature introductions on data-driven insights. The platform's success relies on leveraging subscription-style digital data to address crucial business queries and optimize operations.

The available data within the foodie_fi database schema comprises multiple tables, but this case study primarily focuses on two key tables. Additionally, there's an impending challenge to create a new table tailored to the specific needs of Foodie-Fi's team, ensuring comprehensive data analysis and decision-making.

This business case centers on harnessing the data assets within Foodie-Fi's database schema to explore key business questions, drive strategic decision-making, and facilitate the platform's growth and enhancement in the competitive streaming market.

## Data Collected

![case-study-3-erd](https://github.com/julietansy/Data-with-Danny/assets/151416878/0744c303-3780-4271-87a5-b440485d663a)

**Table 1: plans**

The available data for this business case encompasses the subscription plans offered by Foodie-Fi to its customers upon sign-up. There are distinct plan options:


**Basic Plan:** Customers on this plan have limited access, allowing them to stream videos only. It's available on a monthly basis at a price of $9.90.

**Pro Plan:** This plan offers unrestricted watch time and the capability to download videos for offline viewing. The Pro Plan starts at $19.90 per month or $199 for an annual subscription.

**Trial:** Customers are offered an initial 7-day free trial, which automatically transitions into the Pro Monthly subscription plan unless the customer cancels, downgrades to the Basic Plan, or upgrades to the Pro Annual Plan during the trial period.

**Churn:** When customers opt to cancel their Foodie-Fi service, a churn plan record is generated. Although the price field is null, customers retain access to their plan until the end of the billing period.


The data available in the subscriptions table includes plan IDs, plan names, and corresponding prices associated with each subscription option offered by Foodie-Fi.


**Table 2: subscriptions**

The available subscription data for Foodie-Fi includes detailed records of customer subscriptions. These records entail:

**Customer ID:** Unique identifier for each customer.

**Plan ID:** Identifies the specific subscription plan chosen by the customer.

**Start Date:** Indicates the precise date when a particular plan becomes active for the customer.


Key subscription scenarios include:

**Downgrading or Cancellation:** If a customer downgrades from a higher plan (e.g., Pro or Annual Pro) or cancels their subscription, the higher plan remains active until the billing period ends. The start_date in the subscriptions table reflects the date of the actual plan change.

**Upgrading Accounts:** When customers upgrade from the Basic Plan to a higher-tier plan (e.g., Pro or Annual Pro), the higher plan takes immediate effect, superseding the previous plan.

**Churn Process:** In case of churn (cancellation), customers maintain access until the current billing period concludes. However, the start_date marks the day the customer initiated the cancellation process.


The subscription records include customer-specific details, plan IDs indicating the subscription tier, and the corresponding start dates indicating when each subscription plan becomes active for the customer.


## Case Study Questions

**My SQL approach: Pre-processing**

Create a temporary table by joining the subscriptions and plans table, which can be used for the subsequent queries for the case study.
```sql
SET search_path = foodie_fi;

CREATE TEMP TABLE customer_plans AS
SELECT
    s.customer_id,
    s.plan_id,
    p.plan_name,
    p.price,
    s.start_date,
    CASE
        WHEN LEAD(s.start_date) OVER (PARTITION BY s.customer_id ORDER BY s.start_date) IS NOT NULL THEN 
            LEAD(s.start_date) OVER (PARTITION BY s.customer_id ORDER BY s.start_date)
        ELSE
            NULL
    END AS end_date,
    CASE
        WHEN LEAD(s.start_date) OVER (PARTITION BY s.customer_id ORDER BY s.start_date) IS NOT NULL THEN 
            LEAD(s.start_date) OVER (PARTITION BY s.customer_id ORDER BY s.start_date) - s.start_date
        ELSE
            NULL
    END AS duration_in_days
FROM subscriptions s
LEFT JOIN plans p
    ON s.plan_id = p.plan_id;
```

### A. Customer Journey
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

Try to keep it as short as possible - you may also want to run some sort of join to make your explanations a bit easier!

![image](https://github.com/julietansy/Data-with-Danny/assets/151416878/4436e7c5-7a83-4f53-b525-73b4e0f9e782)



---
**My SQL approach**
```sql
SELECT*
FROM customer_plans
WHERE customer_id IN (1,2,11,13,15,16,18,19);
```

**Query Output and Insights**
| customer_id | plan_id | plan_name     | price  | start_date               | end_date                 | duration_in_days |
| ----------- | ------- | ------------- | ------ | ------------------------ | ------------------------ | ---------------- |
| 1           | 0       | trial         | 0.00   | 2020-08-01T00:00:00.000Z | 2020-08-08T00:00:00.000Z | 7                |
| 1           | 1       | basic monthly | 9.90   | 2020-08-08T00:00:00.000Z |                          |                  |
| 2           | 0       | trial         | 0.00   | 2020-09-20T00:00:00.000Z | 2020-09-27T00:00:00.000Z | 7                |
| 2           | 3       | pro annual    | 199.00 | 2020-09-27T00:00:00.000Z |                          |                  |
| 11          | 0       | trial         | 0.00   | 2020-11-19T00:00:00.000Z | 2020-11-26T00:00:00.000Z | 7                |
| 11          | 4       | churn         |        | 2020-11-26T00:00:00.000Z |                          |                  |
| 13          | 0       | trial         | 0.00   | 2020-12-15T00:00:00.000Z | 2020-12-22T00:00:00.000Z | 7                |
| 13          | 1       | basic monthly | 9.90   | 2020-12-22T00:00:00.000Z | 2021-03-29T00:00:00.000Z | 97               |
| 13          | 2       | pro monthly   | 19.90  | 2021-03-29T00:00:00.000Z |                          |                  |
| 15          | 0       | trial         | 0.00   | 2020-03-17T00:00:00.000Z | 2020-03-24T00:00:00.000Z | 7                |
| 15          | 2       | pro monthly   | 19.90  | 2020-03-24T00:00:00.000Z | 2020-04-29T00:00:00.000Z | 36               |
| 15          | 4       | churn         |        | 2020-04-29T00:00:00.000Z |                          |                  |
| 16          | 0       | trial         | 0.00   | 2020-05-31T00:00:00.000Z | 2020-06-07T00:00:00.000Z | 7                |
| 16          | 1       | basic monthly | 9.90   | 2020-06-07T00:00:00.000Z | 2020-10-21T00:00:00.000Z | 136              |
| 16          | 3       | pro annual    | 199.00 | 2020-10-21T00:00:00.000Z |                          |                  |
| 18          | 0       | trial         | 0.00   | 2020-07-06T00:00:00.000Z | 2020-07-13T00:00:00.000Z | 7                |
| 18          | 2       | pro monthly   | 19.90  | 2020-07-13T00:00:00.000Z |                          |                  |
| 19          | 0       | trial         | 0.00   | 2020-06-22T00:00:00.000Z | 2020-06-29T00:00:00.000Z | 7                |
| 19          | 2       | pro monthly   | 19.90  | 2020-06-29T00:00:00.000Z | 2020-08-29T00:00:00.000Z | 61               |
| 19          | 3       | pro annual    | 199.00 | 2020-08-29T00:00:00.000Z |                          |                  |

---



The query table captures the journeys of eight sample customers within Foodie-Fi. All these customers initially joined via a 7-day trial plan. After the trial period, each customer either continued with a Basic Monthly, Pro Monthly, or Pro Annual plan, except for customer ID 11, who opted to terminate their subscription in 26 November 2020, potentially finding Foodie-Fi unsuitable for their needs.

Customers like ID 13 and 16 upgraded their plans—from Basic Monthly to Pro Monthly and Pro Annual, respectively—after 97 days and 136 days, respectively. Despite the plan upgrade after the trial, customer ID 15 decided that Foodie-Fi wasn't suitable for them after 36 days of being on a paid subscription.

### A. Customer Journey

**1.  How many customers has Foodie-Fi ever had?**

```sql
SELECT 
	COUNT(DISTINCT customer_id) AS total_cust
FROM
  customer_plans;
```

**Query Output and Insights**
| total_cust |
| ---------- |
| 1000       |

Foodi-Fi had 1000 customers.

---
**2.  What is the monthly distribution of trial plan start_date values for our dataset - use the start of the month as the group by value**


```sql
SELECT 
	EXTRACT(MONTH FROM start_date) as month,
  TO_CHAR(start_date, 'Month') as month_name,
	COUNT(plan_name) as trial_count
FROM
  customer_plans
WHERE
  plan_name = 'trial'
GROUP BY
  month,
  month_name;
```
| month | month_name | trial_count |
| ----- | ---------- | ----------- |
| 1     | January    | 88          |
| 2     | February   | 68          |
| 3     | March      | 94          |
| 4     | April      | 81          |
| 5     | May        | 88          |
| 6     | June       | 79          |
| 7     | July       | 89          |
| 8     | August     | 88          |
| 9     | September  | 87          |
| 10    | October    | 79          |
| 11    | November   | 75          |
| 12    | December   | 84          |

The monthly count of trial plan sign-ups varies between 68 and 94. March witnessed the highest number of trial subscriptions, whereas February recorded the lowest count of trial plan sign-ups.

This variance suggests potential seasonal trends or periods of higher or lower customer interest in trying out the service. For instance, there might be increased interest in March and July, possibly due to seasonal changes or marketing campaigns, while November might experience a slight decline in interest. Understanding these fluctuations can aid in strategizing marketing efforts or tailoring promotions during specific months to maximize trial conversions.

---
**3.  What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name**


```sql
SELECT
  a.plan_name,
  b.cust_count
FROM
  customer_plans a
LEFT JOIN
  (SELECT
      plan_name,
      COUNT(customer_id) AS cust_count
   FROM
      customer_plans
   WHERE
      EXTRACT(YEAR FROM start_date) > 2020
   GROUP BY
      plan_name,
      plan_id
   ORDER BY
      plan_id) b
ON
  a.plan_name = b.plan_name
GROUP BY
  a.plan_id,
  a.plan_name,
  b.cust_count
ORDER BY
  a.plan_id;
```
After analyzing the dataset, the breakdown of plan start_date values occurring after the year 2020 is as follows:
| plan_name     | cust_count |
| ------------- | ---------- |
| trial         | null           |
| basic monthly | 8          |
| pro monthly   | 60         |
| pro annual    | 63         |
| churn         | 71         |

There's ongoing engagement and activity even after 2020 across all plan types—Basic Monthly, Pro Monthly, Pro Annual, and even Churn. The presence of activity for Basic Monthly, Pro Monthly, and Pro Annual plans after 2020 indicates that customers are actively subscribing or transitioning to these plans post-2020.

 The absence of trial plan subscriptions after 2020, as indicated in the dataset, implies that Foodie-Fi might have altered its subscription offerings or promotional strategies post-2020. It could suggest a strategic decision to discontinue offering trials or a shift in the marketing approach focusing more on other subscription plans.

---
**4.  What is the customer count and percentage of customers who have churned rounded to 1 decimal place?**
```sql
WITH churn AS (
    SELECT 
        COUNT(CASE WHEN plan_name = 'churn' THEN customer_id END) AS total_churn,
        COUNT(DISTINCT customer_id) AS total_cust
    FROM customer_plans
)

SELECT 
  total_churn AS churn_count,
  ROUND((ROUND((total_churn * 100.0),1) / total_cust),1) AS churn_percentage
FROM churn;
```

| churn_count | churn_percentage |
| ----------- | ---------------- |
| 307         | 30.7             |

This indicates that 307 customers have churned, constituting approximately 30.7% of the total customer base in the dataset. The high churn rate signals potential issues with customer satisfaction, content relevance, or service quality. 

Gathering feedback from churned customers could provide invaluable insights into their reasons for leaving. This data could be used to address pain points and improve service offerings, potentially preventing future churn.C On top of that, comparing churn rates with similar industry standards or competitors might offer perspectives on Foodie-Fi's competitive position and indentifying area of improvements.

---
-- 5.  How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```sql
SELECT 
	COUNT(DISTINCT b.customer_id) as churn_trial_count,
  ROUND((ROUND((COUNT(DISTINCT b.customer_id) * 100),0) / COUNT(DISTINCT a.customer_id)),0) AS churn_trial_percent
FROM
  customer_plans a
LEFT JOIN
  (SELECT customer_id
   FROM customer_plans
   WHERE
      customer_id NOT IN (SELECT customer_id
                          FROM customer_plans
                          WHERE plan_name = 'trial'
   INTERSECT
   SELECT customer_id
   FROM customer_plans
   WHERE plan_name = 'basic monthly'
			OR plan_name = 'pro monthly'
			OR plan_name = 'pro annual')) b
ON a.customer_id = b.customer_id;
```

| churn_trial_count | churn_trial_percent |
| ----------------- | ------------------- |
| 92                | 9                   |


After examining the data, it appears that 92 customers churned straight after their initial free trial. This represents approximately 9% of the total customers, rounded to the nearest whole number, who chose to end their subscription immediately after the trial period.

The immediate churn post-trial might signify a mismatch between what customers expected and what the service offered. Assessing customer feedback or preferences during the trial could aid in aligning content more accurately with user.

---
**6.  What is the number and percentage of customer plans after their initial free trial?**
```sql
SELECT 
	COUNT(DISTINCT b.customer_id) as upgrade_count,
  ROUND((ROUND((COUNT(DISTINCT b.customer_id) * 100),0) / COUNT(DISTINCT a.customer_id)),0) AS upgrade_percent
FROM customer_plans a
LEFT JOIN (SELECT customer_id
           FROM customer_plans
           WHERE customer_id IN (SELECT customer_id FROM customer_plans WHERE plan_name = 'trial'
                                 INTERSECT
                                 SELECT customer_id FROM customer_plans WHERE plan_name = 'basic monthly'
                                    										OR plan_name = 'pro monthly'
                                    										OR plan_name = 'pro annual')) b
ON a.customer_id = b.customer_id;
```
| upgrade_count | upgrade_percent |
| ------------- | --------------- |
| 908           | 91              |

Upon analyzing the data, it indicates that 908 customers proceeded with a subscription plan after completing their initial free trial. This signifies approximately 91% of the total customers who transitioned to a paid subscription following their trial period.

High post-trial conversion indicates that the content or service offerings during the trial phase resonate well with users. Evaluating which content or features attracted these conversions could guide future content strategies or service enhancements. Tailoring incentives or communications based on these preferences could potentially increase conversion rates further.

Focusing on strategies to retain users after their trial-to-paid transition is crucial. Providing ongoing value, personalized recommendations, or exclusive content could prolong customer retention beyond the initial conversion.

---
**7.  What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?**
```sql
WITH as_at_2020 AS (SELECT 
                      plan_name,
                      customer_id
                    FROM customer_plans
                    WHERE start_date <= '2020-12-31' 
                      and (end_date >= '2020-12-31' OR end_date IS NULL)) 


SELECT 
	a.plan_name, 
  COUNT(DISTINCT b.customer_id) AS cust_count,
  ROUND((ROUND((COUNT(DISTINCT b.customer_id) * 100),1) / (SELECT COUNT(customer_id) FROM as_at_2020)),1) AS cust_percent
FROM customer_plans a
LEFT JOIN (SELECT 
           	 plan_name,
           	 customer_id
           FROM as_at_2020) b
ON a.plan_name = b.plan_name
GROUP BY
  a.plan_name,
  a.plan_id
ORDER BY
  a.plan_id;
```
| plan_name     | cust_count | cust_percent |
| ------------- | ---------- | ------------ |
| trial         | 19         | 1.9          |
| basic monthly | 224        | 22.4         |
| pro monthly   | 327        | 32.7         |
| pro annual    | 195        | 19.5         |
| churn         | 236        | 23.6         |

The Pro Monthly plan appears to have the highest customer count (327) and accounts for the largest percentage (32.7%) among all plan types at the end of 2020. This suggests a popular preference for a monthly subscription among customers. This may be potentially due to its flexibility, providing unlimited access on a monthly basis, aligning with customers' preferences for shorter commitment periods

The relatively high percentage of customers in the Churn category (23.6%) might indicate potential challenges in retaining subscribers. Understanding the reasons behind churn and optimizing retention strategies could be crucial for improving customer retention.

---
**8.  How many customers have upgraded to an annual plan in 2020?**
```sql
SELECT 
	COUNT(DISTINCT customer_id) AS total_cust
FROM customer_plans
WHERE plan_name = 'pro annual' 
  AND EXTRACT(YEAR FROM start_date) = '2020';
```
| total_cust |
| ---------- |
| 195        |

195 customers upgraded to an annual plan specifically in the year 2020. The decision to upgrade to a Pro Annual plan, indicates a preference for a longer-term commitment. This choice might signify a desire for cost savings (as annual plans often come with discounts compared to monthly subscriptions) or a strong commitment to the platform's content for an extended period.

From the business standpoint, annual subscribers offer a more predictable revenue stream and stability. Knowing that a significant number of customers commit to an entire year can aid in forecasting and planning for the future. Annual subscribers are less likely to churn frequently compared to monthly subscribers, providing an opportunity for Foodie-Fi to focus on strategies to enhance retention and engagement among these long-term customers.

---
**9.  How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?**
```sql
WITH initial AS (
  SELECT 
  	customer_id, 
  	MIN(start_date) AS initial_start 
  FROM customer_plans 
  GROUP BY customer_id),
annual_convert AS (
  SELECT 
  	customer_id, 
  	start_date 
  FROM customer_plans 
  GROUP BY 
  	customer_id, 
  	start_date, 
  	plan_name 
  HAVING plan_name = 'pro annual')

SELECT 
	ROUND(AVG(ac.start_date - i.initial_start),0) AS avg_days_annual
FROM initial i
RIGHT JOIN annual_convert ac
	ON i.customer_id = ac.customer_id;
```

| avg_days_annual |
| --------------- |
| 105             |


The average duration it takes for a customer to switch to an annual plan from the day they join Foodie-Fi is approximately 105 days. This timeframe suggests that, on average, customers who eventually convert to an annual plan take about three and a half months from their initial sign-up date to upgrade their subscription to the annual tier. It implies the average duration the customers required to engage with the platform's content sufficiently and assess whether it aligns with their expectations and preferences.

Understanding this average conversion timeline can help in designing targeted engagement strategies or promotional offers to encourage quicker transitions to annual plans. This can be achieved by engaging users effectively during this critical window which can potentially enhance retention rates. Providing an excellent experience during the initial months might encourage longer-term commitments and higher customer satisfaction.

---
**10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)**
```sql
WITH initial AS (
  SELECT 
  	customer_id, 
  	MIN(start_date) AS initial_start 
  FROM customer_plans 
  GROUP BY customer_id),
annual_convert AS (
  SELECT 
  	customer_id, 
  	start_date 
  FROM customer_plans 
  GROUP BY 
  	customer_id, 
  	start_date, 
  	plan_name 
  HAVING plan_name = 'pro annual')
  
  SELECT
    CASE 
        WHEN (ac.start_date - i.initial_start) <= 30 THEN '0 - 30 days'
        WHEN (ac.start_date - i.initial_start) <= 60 THEN '31 - 60 days'
        WHEN (ac.start_date - i.initial_start) <= 90 THEN '61 - 90 days'
        WHEN (ac.start_date - i.initial_start) <= 120 THEN '91 - 120 days'
        WHEN (ac.start_date - i.initial_start) <= 150 THEN '121 - 150 days'
        WHEN (ac.start_date - i.initial_start) <= 180 THEN '151 - 180 days'
        WHEN (ac.start_date - i.initial_start) <= 210 THEN '181 - 210 days'
        WHEN (ac.start_date - i.initial_start) <= 240 THEN '211 - 240 days'
        WHEN (ac.start_date - i.initial_start) <= 270 THEN '241 - 270 days'
        WHEN (ac.start_date - i.initial_start) <= 300 THEN '271 - 300 days'
        WHEN (ac.start_date - i.initial_start) <= 330 THEN '301 - 330 days'
        WHEN (ac.start_date - i.initial_start) <= 360 THEN '331 - 360 days'
    END AS period_bucket,
    COUNT(i.customer_id) AS customer_count
FROM initial i
RIGHT JOIN annual_convert ac
	ON i.customer_id = ac.customer_id
GROUP BY
  period_bucket
ORDER BY
  MIN(ac.start_date - i.initial_start);
```

| period_bucket  | customer_count |
| -------------- | -------------- |
| 0 - 30 days    | 49             |
| 31 - 60 days   | 24             |
| 61 - 90 days   | 34             |
| 91 - 120 days  | 35             |
| 121 - 150 days | 42             |
| 151 - 180 days | 36             |
| 181 - 210 days | 26             |
| 211 - 240 days | 4              |
| 241 - 270 days | 5              |
| 271 - 300 days | 1              |
| 301 - 330 days | 1              |
| 331 - 360 days | 1              |

This breakdown provides a detailed view of how many customers transition to an annual plan within specific 30-day intervals after their initial sign-up. It can assist in identifying patterns and optimizing strategies to influence customer decisions within these critical timeframes.

A significant portion of customers (49) converted to an annual plan within the initial 30 days of joining Foodie-Fi. This suggests a subset of users who quickly recognized the platform's value and committed to a longer-term subscription.

The number of conversions gradually declines in successive 30-day periods. This trend implies that while a considerable number of customers adopt the annual plan early, a smaller but consistent flow of conversions occurs over the following months.

Understanding these conversion periods can guide targeted engagement strategies. Tailoring promotions, highlighting value, or offering incentives during these timeframes might encourage more customers to upgrade to an annual plan.

---
**11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?**
```sql
WITH pro_monthly AS (
    SELECT 
        customer_id, 
        start_date
    FROM customer_plans 
  	WHERE plan_name = 'pro monthly'
),
basic_monthly AS (
    SELECT 
        customer_id, 
        start_date
    FROM customer_plans 
  	WHERE plan_name = 'basic monthly'
)

SELECT 
    bm.customer_id
FROM basic_monthly bm
INNER JOIN pro_monthly pm
	ON bm.customer_id = pm.customer_id
WHERE bm.start_date > pm.start_date;
```

(There are no results to be displayed.)


It seems there are no customers who downgraded from a 'Pro Monthly' to a 'Basic Monthly' plan in 2020 according to the provided data. This absence of results suggests that within the specified dataset and timeframe, there were no instances where customers transitioned from the 'Pro Monthly' plan to the 'Basic Monthly' plan.
