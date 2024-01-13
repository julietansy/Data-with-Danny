# Case Study #2 - Pizza Runner

![2](https://github.com/julietansy/Data-with-Danny/assets/151416878/2dc85a0d-ad38-4123-af9a-b0142316e4b0)

[Source Link](https://8weeksqlchallenge.com/case-study-2/)

## Business Case

## Data Collected

**Table 1: runners**

**Table 2: customer_orders**

**Table 3: runner_orders**

**Table 4: pizza_names**

**Table 5: pizza_recipes**

**Table 6: pizza_toppings**


## Case Study Questions

**My SQL approach: Pre-processing**
**Data Cleaning**

        SET search_path = pizza_runner;
        
        UPDATE customer_orders
        SET extras = ''
        WHERE extras IS NULL;
        
        UPDATE customer_orders
        SET exclusions = ''
        WHERE exclusions = 'null';
        
        UPDATE customer_orders
        SET extras = ''
        WHERE extras = 'null';
        
        UPDATE runner_orders
        SET pickup_time = ''
        WHERE pickup_time = 'null';
        
        UPDATE runner_orders
        SET distance = ''
        WHERE distance = 'null';
        
        UPDATE runner_orders
        SET duration = ''
        WHERE duration = 'null';
        
        UPDATE runner_orders
        SET cancellation = ''
        WHERE cancellation IS NULL OR cancellation = 'null';
        
        SELECT *
        FROM runner_orders;
        
        SELECT *
        FROM customer_orders;

### A. Pizza Metrics

1. How many pizzas were ordered?
   
       SELECT COUNT(pizza_id) AS pizza_order
       FROM customer_orders;

| pizza_order |
| ----------- |
| 14          |

2. How many unique customer orders were made?

        SELECT COUNT(DISTINCT order_id) AS unique_order
        FROM customer_orders;

| unique_order |
| ------------ |
| 10           |


3. How many successful orders were delivered by each runner?

          SELECT runner_id, COUNT(duration) AS successful_delivery
          FROM runner_orders
          WHERE duration != ''
          GROUP BY runner_id
          ORDER BY runner_id;

| runner_id | successful_delivery |
| --------- | ------------------- |
| 1         | 4                   |
| 2         | 3                   |
| 3         | 1                   |

4. How many of each type of pizza was delivered?

        SELECT cus.pizza_id, pn.pizza_name, COUNT(del.duration) AS successful_delivery
        FROM customer_orders cus
        LEFT JOIN runner_orders del
        ON del.order_id = cus.order_id
        LEFT JOIN pizza_names pn
        ON cus.pizza_id = pn.pizza_id
        WHERE del.duration != ''
        GROUP BY cus.pizza_id, pn.pizza_name
        ORDER BY cus.pizza_id;

| pizza_id | pizza_name | successful_delivery |
| -------- | ---------- | ------------------- |
| 1        | Meatlovers | 9                   |
| 2        | Vegetarian | 3                   |

5. How many Vegetarian and Meatlovers were ordered by each customer?

        SELECT pn.pizza_name, cus.customer_id, COUNT(pn.pizza_name) AS pizza_ordered
        FROM customer_orders cus
        LEFT JOIN pizza_names pn
        ON cus.pizza_id = pn.pizza_id
        GROUP BY cus.customer_id, pn.pizza_name
        ORDER BY pn.pizza_name, cus.customer_id;

| pizza_name | customer_id | pizza_ordered |
| ---------- | ----------- | ------------- |
| Meatlovers | 101         | 2             |
| Meatlovers | 102         | 2             |
| Meatlovers | 103         | 3             |
| Meatlovers | 104         | 3             |
| Vegetarian | 101         | 1             |
| Vegetarian | 102         | 1             |
| Vegetarian | 103         | 1             |
| Vegetarian | 105         | 1             |

6. What was the maximum number of pizzas delivered in a single order?

        SELECT order_id, COUNT(pizza_id) AS order_qty
        FROM customer_orders
        GROUP BY order_id
        ORDER BY order_qty DESC
        LIMIT 1;

| order_id | order_qty |
| -------- | --------- |
| 4        | 3         |

7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

        SELECT cus.customer_id, COUNT(cus.pizza_id) AS pizza_changed
        FROM customer_orders cus
        LEFT JOIN runner_orders del
        ON cus.order_id = del.order_id
        WHERE del.duration != '' AND (cus.exclusions != '' OR cus.extras != '')
        GROUP BY cus.customer_id;

| customer_id | pizza_changed |
| ----------- | ------------- |
| 103         | 3             |
| 104         | 2             |
| 105         | 1             |

        SELECT cus.customer_id, COUNT(cus.pizza_id) AS pizza_unchanged
        FROM customer_orders cus
        LEFT JOIN runner_orders del
        ON cus.order_id = del.order_id
        WHERE del.duration != '' AND (cus.exclusions = '' AND cus.extras = '')
        GROUP BY cus.customer_id;

| customer_id | pizza_unchanged |
| ----------- | --------------- |
| 101         | 2               |
| 102         | 3               |
| 104         | 1               |

8. How many pizzas were delivered that had both exclusions and extras?

        SELECT cus.customer_id, COUNT(cus.pizza_id) AS pizza_exc_ext
        FROM customer_orders cus
        LEFT JOIN runner_orders del
        ON cus.order_id = del.order_id
        WHERE del.duration != '' AND (cus.exclusions != '' AND cus.extras != '')
        GROUP BY cus.customer_id;

| customer_id | pizza_exc_ext |
| ----------- | ------------- |
| 104         | 1             |

9. What was the total volume of pizzas ordered for each hour of the day?

        SELECT EXTRACT(HOUR FROM order_time) AS hour_of_day, COUNT(pizza_id) AS pizza_order
        FROM customer_orders
        GROUP BY hour_of_day
        ORDER BY hour_of_day;

| hour_of_day | pizza_order |
| ----------- | ----------- |
| 11          | 1           |
| 13          | 3           |
| 18          | 3           |
| 19          | 1           |
| 21          | 3           |
| 23          | 3           |

10. What was the volume of orders for each day of the week?

        SELECT EXTRACT(DOW FROM order_time) AS day_of_week, COUNT(pizza_id) AS pizza_order
        FROM customer_orders
        GROUP BY day_of_week
        ORDER BY day_of_week;

| day_of_week | pizza_order |
| ----------- | ----------- |
| 3           | 5           |
| 4           | 3           |
| 5           | 1           |
| 6           | 5           |

### B. Runner and Customer Experience

1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

        SELECT EXTRACT(WEEK FROM registration_date) AS week_no, COUNT(runner_id) AS runner_count, registration_date
        FROM runners
        GROUP BY week_no, registration_date
        ORDER BY week_no;

| week_no | runner_count | registration_date        |
| ------- | ------------ | ------------------------ |
| 1       | 1            | 2021-01-08T00:00:00.000Z |
| 2       | 1            | 2021-01-15T00:00:00.000Z |
| 53      | 1            | 2021-01-03T00:00:00.000Z |
| 53      | 1            | 2021-01-01T00:00:00.000Z |

The behavior you're observing may be related to the specific rules for week numbering in the database system you're using, and it could also be influenced by the settings of the system's calendar.

In many systems, the week numbering is determined according to the ISO 8601 standard, which specifies that a week starts on a Monday and that the first week of the year is the week that contains at least four days of the new year. If January 1st falls on a Monday to Thursday, it is part of the first week of the year; otherwise, it is part of the last week of the previous year.

If your system follows ISO 8601, then January 1, 2021, was a Friday, and since it doesn't meet the requirement of having at least four days of the new year, it is considered part of the last week of the previous year (Week 53 of the previous year). This is why you are seeing it as Week 53 instead of Week 1.

If you want to change this behavior or if your database system uses a different set of rules, you may need to refer to the documentation of your specific database system to understand how week numbering is implemented and whether there are any settings or options that can be adjusted.

2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

   
3. Is there any relationship between the number of pizzas and how long the order takes to prepare?


4. What was the average distance travelled for each customer?


5. What was the difference between the longest and shortest delivery times for all orders?


6. What was the average speed for each runner for each delivery and do you notice any trend for these values?


7. What is the successful delivery percentage for each runner?


### C. Ingredient Optimisation

1. What are the standard ingredients for each pizza?


2. What was the most commonly added extra?


3. What was the most common exclusion?


4. Generate an order item for each record in the customers_orders table in the format of one of the following:
    - Meat Lovers
    - Meat Lovers - Exclude Beef
    - Meat Lovers - Extra Bacon
    - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers


5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
    - For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"


6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
