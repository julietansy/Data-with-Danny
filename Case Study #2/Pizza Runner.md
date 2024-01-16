# Case Study #2 - Pizza Runner

![2](https://github.com/julietansy/Data-with-Danny/assets/151416878/2dc85a0d-ad38-4123-af9a-b0142316e4b0)

[Source Link](https://8weeksqlchallenge.com/case-study-2/)

## Business Case

Pizza Runner, a culinary venture inspired by the timeless appeal of pizza and the efficiency of the gig economy. Conceived by Danny, this innovative endeavor seamlessly blends 80s retro styling with the convenience of modern delivery services. More than just a pizzeria, Pizza Runner transforms pizza delivery through a fleet of dedicated "runners" and a user-friendly mobile app developed by freelance developers. Join us in exploring the strategic vision, operational prowess, and growth potential that define Pizza Runner's quest to revolutionize the global pizza delivery experience.

## Data Collected

![image](https://github.com/julietansy/Data-with-Danny/assets/151416878/320b9ef7-ff09-4ebc-8a42-1c25d4d9b10f)


**Table 1: runners**

The runners table shows the registration_date for each new runner.

**Table 2: customer_orders**


Orders from customers are recorded in the "customer_orders" table, with each row representing an individual pizza within an order. The "pizza_id" indicates the type of pizza ordered. In the context of these orders, "exclusions" are ingredient_id values meant to be removed from the pizza, while "extras" are ingredient_id values intended to be added.

It's important to note that a single order can comprise multiple pizzas, and these pizzas can be of the same type but with varying exclusions and extras.

Before incorporating the "exclusions" and "extras" columns into queries, it's necessary to clean up and preprocess these columns for accurate and effective use.

**Table 3: runner_orders**


Following the receipt of each order through the system, a designated runner is assigned. It's important to note that not all orders are successfully completed and may be canceled by either the restaurant or the customer.

The "pickup_time" serves as the timestamp indicating when the assigned runner arrives at Pizza Runner headquarters to collect the freshly prepared pizzas. Additionally, the "distance" and "duration" fields provide information about the distance traveled and the time taken by the runner for the delivery.

It's crucial to exercise caution when utilizing this table due to known data issues. Ensure careful scrutiny of the data types for each column as specified in the schema SQL when incorporating this information into queries.

**Table 4: pizza_names**

Presently, Pizza Runner offers a limited selection of two pizzas: the Meat Lovers and the Vegetarian.

**Table 5: pizza_recipes**


Every pizza_id is associated with a predefined set of toppings, constituting the standard ingredients used in the pizza recipe.
 
**Table 6: pizza_toppings**

The table encompasses all topping_name values paired with their respective topping_id values.

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
        

### A. Pizza Metrics

1. How many pizzas were ordered?
   
       SELECT
           COUNT(pizza_id) AS pizza_order
       FROM customer_orders;

| pizza_order |
| ----------- |
| 14          |

There are 14 pizza orders being placed.

2. How many unique customer orders were made?

        SELECT
           COUNT(DISTINCT order_id) AS unique_order
        FROM customer_orders;

| unique_order |
| ------------ |
| 10           |

There are 10 unique customer order being placed.

3. How many successful orders were delivered by each runner?

        SELECT 
          runner_id,
          COUNT(duration) AS successful_delivery
        FROM 
          runner_orders
        WHERE 
          duration != ''
        GROUP BY 
          runner_id
        ORDER BY 
          runner_id;


| runner_id | successful_delivery |
| --------- | ------------------- |
| 1         | 4                   |
| 2         | 3                   |
| 3         | 1                   |

The table shows the count of successful deliveries for each runner. Runner 1 completed 4 successful deliveries, Runner 2 completed 3, and Runner 3 completed 1.


4. How many of each type of pizza was delivered?

        SELECT 
          cus.pizza_id,
          pn.pizza_name,
          COUNT(del.duration) AS successful_delivery
        FROM 
          customer_orders cus
        LEFT JOIN 
          runner_orders del
        ON 
          del.order_id = cus.order_id
        LEFT JOIN 
          pizza_names pn
        ON 
          cus.pizza_id = pn.pizza_id
        WHERE 
          del.duration != ''
        GROUP BY 
          cus.pizza_id, pn.pizza_name
        ORDER BY 
          cus.pizza_id;


| pizza_id | pizza_name | successful_delivery |
| -------- | ---------- | ------------------- |
| 1        | Meatlovers | 9                   |
| 2        | Vegetarian | 3                   |

Meatlovers pizza has been delivered 9 times, while Vegetarian pizza has been delivered 3 times. This insight can be valuable for understanding customer preferences and demand for different pizza types

5. How many Vegetarian and Meatlovers were ordered by each customer?

        SELECT 
          pn.pizza_name,
          cus.customer_id,
          COUNT(pn.pizza_name) AS pizza_ordered
        FROM 
          customer_orders cus
        LEFT JOIN 
          pizza_names pn
        ON 
          cus.pizza_id = pn.pizza_id
        GROUP BY 
          cus.customer_id, pn.pizza_name
        ORDER BY 
          pn.pizza_name, cus.customer_id;


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

The customer base shows diversity in pizza preferences and Meatlovers emerges as a potentially more popular choice.

6. What was the maximum number of pizzas delivered in a single order?

        SELECT 
          order_id,
          COUNT(pizza_id) AS order_qty
        FROM 
          customer_orders
        GROUP BY 
          order_id
        ORDER BY 
          order_qty DESC
        LIMIT 1;


| order_id | order_qty |
| -------- | --------- |
| 4        | 3         |


The query output indicates that the maximum number of pizzas delivered in a single order was 3 and that was for order_id 4.

7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

        SELECT 
          cus.customer_id,
          COUNT(cus.pizza_id) AS pizza_changed
        FROM 
          customer_orders cus
        LEFT JOIN 
          runner_orders del
        ON 
          cus.order_id = del.order_id
        WHERE 
          del.duration != '' 
          AND (cus.exclusions != '' OR cus.extras != '')
        GROUP BY 
          cus.customer_id;


| customer_id | pizza_changed |
| ----------- | ------------- |
| 103         | 3             |
| 104         | 2             |
| 105         | 1             |

        SELECT 
          cus.customer_id,
          COUNT(cus.pizza_id) AS pizza_unchanged
        FROM 
          customer_orders cus
        LEFT JOIN 
          runner_orders del
        ON 
          cus.order_id = del.order_id
        WHERE 
          del.duration != '' 
          AND (cus.exclusions = '' AND cus.extras = '')
        GROUP BY 
          cus.customer_id;


| customer_id | pizza_unchanged |
| ----------- | --------------- |
| 101         | 2               |
| 102         | 3               |
| 104         | 1               |

The two queries identifiied the number of delivered pizzas for each customer based on whether there were changes (additions or exclusions) or no changes made to the pizza orders. Customer_id 103 and 105 seems to have specific preferences over the pizza ordered as they had pizza changed for every order that was made.

8. How many pizzas were delivered that had both exclusions and extras?

        SELECT 
          cus.customer_id,
          COUNT(cus.pizza_id) AS pizza_exc_ext
        FROM 
          customer_orders cus
        LEFT JOIN 
          runner_orders del
        ON 
          cus.order_id = del.order_id
        WHERE 
          del.duration != '' 
          AND (cus.exclusions != '' AND cus.extras != '')
        GROUP BY 
          cus.customer_id;


| customer_id | pizza_exc_ext |
| ----------- | ------------- |
| 104         | 1             |

Customer 104 had one pizza delivered with both exclusions and extras.

9. What was the total volume of pizzas ordered for each hour of the day?

        SELECT 
          EXTRACT(HOUR FROM order_time) AS hour_of_day,
          COUNT(pizza_id) AS pizza_order
        FROM 
          customer_orders
        GROUP BY 
          hour_of_day
        ORDER BY 
          hour_of_day;


| hour_of_day | pizza_order |
| ----------- | ----------- |
| 11          | 1           |
| 13          | 3           |
| 18          | 3           |
| 19          | 1           |
| 21          | 3           |
| 23          | 3           |

It was being observed that the higher volume of pizza orders during the following hours of the day:

- 13:00 (1:00 PM), 18:00 (6:00 PM), 21:00 (9:00 PM) and 23:00 (11:00 PM):

The concentration of higher pizza orders during the meal hours, such as lunch (13:00), dinner (18:00), supper (21:00), and late-night snacks (23:00), suggests a clear pattern of peak demand during traditional meal times.

The insight underscores the importance for Pizza Runner to strategically manage operations, allocate resources efficiently, and possibly consider offering promotions or incentives during these peak hours to optimize sales and customer satisfaction.

10. What was the volume of orders for each day of the week?

        SELECT 
          EXTRACT(DOW FROM order_time) AS day_of_week,
          COUNT(pizza_id) AS pizza_order
        FROM 
          customer_orders
        GROUP BY 
          day_of_week
        ORDER BY 
          day_of_week;


| day_of_week | pizza_order |
| ----------- | ----------- |
| 3           | 5           |
| 4           | 3           |
| 5           | 1           |
| 6           | 5           |

These results show the distribution of pizza orders across different days of the week, with Wednesday (3), Thursday (4), Friday (5), and Saturday (6).

It appears that Saturday has the highest volume of pizza orders, and Wednesday also has a significant number of orders.

### B. Runner and Customer Experience

1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

        SELECT 
          EXTRACT(WEEK FROM registration_date) AS week_no,
          COUNT(runner_id) AS runner_count,
          registration_date
        FROM 
          runners
        GROUP BY 
          week_no, registration_date
        ORDER BY 
          week_no;


| week_no | runner_count | registration_date        |
| ------- | ------------ | ------------------------ |
| 1       | 1            | 2021-01-08T00:00:00.000Z |
| 2       | 1            | 2021-01-15T00:00:00.000Z |
| 53      | 1            | 2021-01-03T00:00:00.000Z |
| 53      | 1            | 2021-01-01T00:00:00.000Z |

_Note: The week numbering is determined according to the ISO 8601 standard, which specifies that a week starts on a Monday and that the first week of the year is the week that contains at least four days of the new year. 01 January 2021, was a Friday, and since it doesn't meet the requirement of having at least four days of the new year, it is considered part of the last week of the previous year (Week 53 of the previous year). This is why 01 and 03 January are part of Week 53 instead of Week 1._


2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
        
        WITH time_diff AS (
          SELECT 
            del.runner_id,
            EXTRACT(MINUTE FROM del.pickup_time::timestamp - cus.order_time::timestamp) AS minutes,
            EXTRACT(SECOND FROM del.pickup_time::timestamp - cus.order_time::timestamp) AS seconds
          FROM 
            customer_orders cus
          LEFT JOIN 
            runner_orders del ON cus.order_id = del.order_id
          WHERE 
            del.duration != ''
        )
        SELECT
          runner_id,
          CAST(AVG(minutes) AS DECIMAL(10, 2)) AS avg_minutes
        FROM 
          time_diff
        GROUP BY 
          runner_id
        ORDER BY 
          runner_id;


| runner_id | avg_minutes |
| --------- | ----------- |
| 1         | 15.33       |
| 2         | 23.40       |
| 3         | 10.00       |
   
3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

            WITH order_prep AS (
              SELECT
              	cus.order_id,
              	COUNT(cus.pizza_id) AS pizza_count,
              	(del.pickup_time::timestamp - cus.order_time::timestamp) AS prep_time
              FROM customer_orders cus
              LEFT JOIN runner_orders del ON cus.order_id = del.order_id
              WHERE del.duration != ''
              GROUP BY
              	cus.order_id,
              	del.pickup_time,
              	cus.order_time
            )
            
            SELECT 
            	pizza_count, 
                EXTRACT(MINUTE FROM AVG(prep_time)) AS avg_minutes,
                CAST(EXTRACT(SECOND FROM AVG(prep_time)) AS DECIMAL(2, 0)) AS avg_seconds
            FROM order_prep
            GROUP BY pizza_count
            ORDER BY pizza_count;

| pizza_count | avg_minutes | avg_seconds |
| ----------- | ----------- | ----------- |
| 1           | 12          | 21          |
| 2           | 18          | 23          |
| 3           | 29          | 17          |

These results show the average preparation time, in both minutes and seconds, for orders with different pizza counts. As the pizza count increases, the average preparation time also tends to increase.

4. What was the average distance travelled for each customer?

        WITH dist_travel AS (
          SELECT 
            del.order_id,
            cus.customer_id,
            CAST(REGEXP_REPLACE(distance, '[^0-9.]', '', 'g') AS DECIMAL) AS distance
          FROM 
            runner_orders del
          LEFT JOIN 
            customer_orders cus ON del.order_id = cus.order_id
          WHERE 
            del.duration != ''
        )
        
        SELECT 
          customer_id,
          ROUND(AVG(distance), 2) AS avg_dist
        FROM 
          dist_travel
        GROUP BY 
          customer_id
        ORDER BY 
          customer_id;


| customer_id | avg_dist |
| ----------- | -------- |
| 101         | 20.00    |
| 102         | 16.73    |
| 103         | 23.40    |
| 104         | 10.00    |
| 105         | 25.00    |

5. What was the difference between the longest and shortest delivery times for all orders?

        WITH time_diff AS (
          SELECT 
            del.runner_id,
            del.pickup_time,
            cus.order_time,
            RANK() OVER (
              ORDER BY 
                EXTRACT(MINUTE FROM del.pickup_time::timestamp - cus.order_time::timestamp), 
                EXTRACT(SECOND FROM del.pickup_time::timestamp - cus.order_time::timestamp)
            ) AS rank_no
          FROM 
            customer_orders cus
          LEFT JOIN 
            runner_orders del ON cus.order_id = del.order_id
          WHERE 
            del.duration != ''
        ),
        max_del_time AS (
          SELECT 
            pickup_time,
            order_time
          FROM 
            time_diff
          WHERE 
            rank_no = (SELECT MAX(rank_no) FROM time_diff)
          LIMIT 1
        ),
        min_del_time AS (
          SELECT 
            pickup_time,
            order_time
          FROM 
            time_diff
          WHERE 
            rank_no = (SELECT MIN(rank_no) FROM time_diff)
          LIMIT 1
        )
        
        SELECT 
          EXTRACT(MINUTE FROM (max_del_time.pickup_time::timestamp - max_del_time.order_time::timestamp)
                              - (min_del_time.pickup_time::timestamp - min_del_time.order_time::timestamp)) AS minutes,
          
          EXTRACT(SECOND FROM (max_del_time.pickup_time::timestamp - max_del_time.order_time::timestamp)
                              - (min_del_time.pickup_time::timestamp - min_del_time.order_time::timestamp)) AS seconds;


| minutes | seconds |
| ------- | ------- |
| 19      | 15      |

6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

        WITH del_travel AS (
          SELECT 
            del.runner_id,
            del.order_id,
            CAST(REGEXP_REPLACE(distance, '[^0-9.]', '', 'g') AS DECIMAL) AS distance,
            CAST(REGEXP_REPLACE(duration, '[^0-9.]', '', 'g') AS DECIMAL) AS duration,
            ROUND(CAST(REGEXP_REPLACE(distance, '[^0-9.]', '', 'g') AS DECIMAL) / (CAST(REGEXP_REPLACE(duration, '[^0-9.]', '', 'g') AS DECIMAL) / 60), 2) AS speed_kmh
          FROM 
            runner_orders del
            LEFT JOIN customer_orders cus ON del.order_id = cus.order_id
          WHERE 
            del.duration != ''
        )
        
        SELECT
          runner_id,
          ROUND(AVG(speed_kmh), 2) AS avg_speed
        FROM 
          del_travel
        GROUP BY 
          runner_id
        ORDER BY 
          runner_id;



| runner_id | avg_speed |
| --------- | --------- |
| 1         | 47.06     |
| 2         | 51.78     |
| 3         | 40.00     |

The trend suggests that there are variations in the average speed among the runners, with Runner 2 being the fastest and Runner 3 being the slowest. It might be worth further investigation to understand the factors influencing these speed differences, such as distance traveled and delivery time.

7. What is the successful delivery percentage for each runner?
        
        WITH complete_order AS (
          SELECT
            runner_id,
            COUNT(order_id) AS com_order
          FROM 
            runner_orders
          WHERE 
            duration != ''
          GROUP BY 
            runner_id
        )
        
        SELECT 
          del.runner_id,
          ROUND(cdel.com_order * 1.0 / COUNT(del.order_id) * 100) AS del_percent
        FROM 
          runner_orders del
        LEFT JOIN 
          complete_order cdel ON del.runner_id = cdel.runner_id
        GROUP BY 
          del.runner_id, cdel.com_order;


| runner_id | del_percent |
| --------- | ----------- |
| 1         | 100         |
| 3         | 50          |
| 2         | 75          |

### C. Ingredient Optimisation

1. What are the standard ingredients for each pizza?

        SELECT
          pr.pizza_id,
          pn.pizza_name,
          STRING_AGG(topping_name, ', ') AS standard_ingredients_list
        FROM
          pizza_recipes pr
        LEFT JOIN
          pizza_toppings ON topping_id = ANY(STRING_TO_ARRAY(toppings, ',')::int[])
        LEFT JOIN
          pizza_names pn ON pr.pizza_id = pn.pizza_id
        GROUP BY 
          pr.pizza_id, 
          pn.pizza_name
        ORDER BY 
          pr.pizza_id;


| pizza_id | pizza_name | standard_ingredients_list                                             |
| -------- | ---------- | --------------------------------------------------------------------- |
| 1        | Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 2        | Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomatoes, Tomato Sauce            |

2. What was the most commonly added extra?

        WITH extras_top AS (
          SELECT
            unnest(STRING_TO_ARRAY(extras, ',')::int[]) AS topping_id,
            COUNT(pizza_id) AS count
          FROM
            customer_orders
          GROUP BY topping_id
          ORDER BY count
        )
        
        SELECT
          et.topping_id,
          pt.topping_name,
          et.count
        FROM extras_top et
        LEFT JOIN pizza_toppings pt ON et.topping_id = pt.topping_id;


| topping_id | topping_name | count |
| ---------- | ------------ | ----- |
| 1          | Bacon        | 4     |
| 4          | Cheese       | 1     |
| 5          | Chicken      | 1     |

 "Bacon" (topping_id: 1) is the most commonly added extra topping, appearing in 4 pizzas. Additionally, "Cheese" and "Chicken" each appear once as added extras.
 
3. What was the most common exclusion?

        WITH exclu_top AS (
          SELECT
            unnest(STRING_TO_ARRAY(exclusions, ',')::int[]) AS exclu_topping_id,
            COUNT(pizza_id) AS count
          FROM
            customer_orders
          GROUP BY exclu_topping_id
          ORDER BY count
        )
        
        SELECT
          et.exclu_topping_id,
          pt.topping_name,
          et.count
        FROM exclu_top et
        LEFT JOIN pizza_toppings pt ON et.exclu_topping_id = pt.topping_id
        ORDER BY count DESC;


| exclu_topping_id | topping_name | count |
| ---------------- | ------------ | ----- |
| 4                | Cheese       | 4     |
| 2                | BBQ Sauce    | 1     |
| 6                | Mushrooms    | 1     |

"Cheese" (topping_id: 4) is the most commonly excluded topping, appearing in 4 pizzas. Additionally, "BBQ Sauce" and "Mushrooms" each appear once as excluded toppings.

4. Generate an order item for each record in the customers_orders table in the format of one of the following:
    - Meat Lovers
    - Meat Lovers - Exclude Beef
    - Meat Lovers - Extra Bacon
    - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

                SELECT
                  order_id,
                  pizza_id,
                  CASE WHEN pizza_id = 2 THEN 'Yes' ELSE '-' END AS Meat_Lovers,
                  CASE WHEN STRING_AGG(exclusions, ',') LIKE '%3%' THEN 'Yes' ELSE '-' END AS Meat_Lovers_Exclude_Beef,
                  CASE WHEN STRING_AGG(extras, ',') LIKE '%1%' THEN 'Yes' ELSE '-' END AS Meat_Lovers_Extra_Bacon,
                  CASE WHEN STRING_AGG(exclusions, ',') LIKE '%1%' 
                       OR STRING_AGG(exclusions, ',') LIKE '%3%' 
                       OR STRING_AGG(exclusions, ',') LIKE '%6%' 
                       OR STRING_AGG(exclusions, ',') LIKE '%9%' 
                       THEN 'Yes' ELSE '-' END AS Meat_Lovers_Exclude_CBMP
                FROM customer_orders
                GROUP BY order_id, pizza_id
                ORDER BY order_id;

| order_id | pizza_id | meat_lovers | meat_lovers_exclude_beef | meat_lovers_extra_bacon | meat_lovers_exclude_cbmp |
| -------- | -------- | ----------- | ------------------------ | ----------------------- | ------------------------ |
| 1        | 1        | -           | -                        | -                       | -                        |
| 2        | 1        | -           | -                        | -                       | -                        |
| 3        | 1        | -           | -                        | -                       | -                        |
| 3        | 2        | Yes         | -                        | -                       | -                        |
| 4        | 1        | -           | -                        | -                       | -                        |
| 4        | 2        | Yes         | -                        | -                       | -                        |
| 5        | 1        | -           | -                        | Yes                     | -                        |
| 6        | 2        | Yes         | -                        | -                       | -                        |
| 7        | 2        | Yes         | -                        | Yes                     | -                        |
| 8        | 1        | -           | -                        | -                       | -                        |
| 9        | 1        | -           | -                        | Yes                     | -                        |
| 10       | 1        | -           | -                        | Yes                     | Yes                      |

5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
    - For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

            WITH pizza_ingredients AS (
              SELECT
                cus.sn,
                cus.order_id,
                cus.pizza_id,
                cus.exclusions,
                cus.extras,
                pn.pizza_name,
                STRING_AGG(topping_name, ', ') AS excluded_ingredient
              FROM 
                (SELECT ROW_NUMBER() OVER (ORDER BY order_id) AS sn, * FROM customer_orders) AS cus
              LEFT JOIN 
                pizza_toppings ON topping_id = ANY(STRING_TO_ARRAY(exclusions, ',')::int[])
              LEFT JOIN 
                pizza_names pn ON cus.pizza_id = pn.pizza_id
              GROUP BY
                cus.sn,
                cus.order_id,
                cus.pizza_id,
                cus.exclusions,
                cus.extras,
                pn.pizza_name
              ORDER BY
                cus.order_id
            ), 
            pizza_ingredients_2 AS (
              SELECT 
                pi.*,
                STRING_AGG(topping_name, ', ') AS extras_ingredient
              FROM 
                pizza_ingredients pi
              LEFT JOIN 
                pizza_toppings ON topping_id = ANY(STRING_TO_ARRAY(extras, ', ')::int[])
              GROUP BY
                pi.sn,
                pi.order_id,
                pi.pizza_id,
                pi.exclusions,
                pi.extras,
                pi.pizza_name,
                pi.excluded_ingredient
            )
            
            SELECT 
              pi2.sn,
              pi2.order_id,
              pi2.pizza_name,
              STRING_AGG(
                CASE
                  WHEN pt.topping_name = ANY(STRING_TO_ARRAY(pi2.extras_ingredient, ', ')) THEN '2x ' || pt.topping_name
                  WHEN pt.topping_name = ANY(STRING_TO_ARRAY(pi2.excluded_ingredient, ', ')) THEN NULL
                  ELSE pt.topping_name
                END, ', ' ORDER BY LOWER(pt.topping_name)) AS standard_ingredient
            FROM 
              pizza_ingredients_2 pi2
            LEFT JOIN 
              pizza_recipes pr ON pr.pizza_id = pi2.pizza_id
            LEFT JOIN 
              pizza_toppings pt ON pt.topping_id = ANY(STRING_TO_ARRAY(pr.toppings, ',')::int[])
            GROUP BY
              pi2.sn,
              pi2.order_id,
              pi2.pizza_name;

| sn  | order_id | pizza_name | standard_ingredient                                                      |
| --- | -------- | ---------- | ------------------------------------------------------------------------ |
| 1   | 1        | Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 2   | 2        | Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 3   | 3        | Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 4   | 3        | Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes               |
| 5   | 4        | Meatlovers | Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami            |
| 6   | 4        | Meatlovers | Bacon, BBQ Sauce, Beef, Chicken, Mushrooms, Pepperoni, Salami            |
| 7   | 4        | Vegetarian | Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes                       |
| 8   | 5        | Meatlovers | 2x Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami |
| 9   | 6        | Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes               |
| 10  | 7        | Vegetarian | Cheese, Mushrooms, Onions, Peppers, Tomato Sauce, Tomatoes               |
| 11  | 8        | Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 12  | 9        | Meatlovers | 2x Bacon, BBQ Sauce, Beef, 2x Chicken, Mushrooms, Pepperoni, Salami      |
| 13  | 10       | Meatlovers | Bacon, BBQ Sauce, Beef, Cheese, Chicken, Mushrooms, Pepperoni, Salami    |
| 14  | 10       | Meatlovers | 2x Bacon, Beef, 2x Cheese, Chicken, Pepperoni, Salami                    |

6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?

            WITH standard AS (
                SELECT
                    UNNEST(STRING_TO_ARRAY(pr.toppings, ', '))::integer AS standard,
                    COUNT(*) AS standard_count
                FROM
                    customer_orders cus
                LEFT JOIN
                    runner_orders del ON cus.order_id = del.order_id
                LEFT JOIN
                    pizza_recipes pr ON pr.pizza_id = cus.pizza_id
                WHERE del.duration != ''
                GROUP BY standard
            ),
            excluded AS (
                SELECT
                    UNNEST(STRING_TO_ARRAY(cus.exclusions, ', '))::integer AS removal,
                    COUNT(*) AS excluded_count
                FROM
                    customer_orders cus
                LEFT JOIN
                    runner_orders del ON cus.order_id = del.order_id
                LEFT JOIN
                    pizza_recipes pr ON pr.pizza_id = cus.pizza_id
                WHERE del.duration != ''
                GROUP BY removal
            ),
            additional AS (
                SELECT
                    UNNEST(STRING_TO_ARRAY(cus.extras, ', '))::integer AS addition,
                    COUNT(*) AS addition_count
                FROM
                    customer_orders cus
                LEFT JOIN
                    runner_orders del ON cus.order_id = del.order_id
                LEFT JOIN
                    pizza_recipes pr ON pr.pizza_id = cus.pizza_id
                WHERE del.duration != ''
                GROUP BY addition
            )
            
            SELECT 
                pt.topping_name,
                (s.standard_count - COALESCE(e.excluded_count, 0) + COALESCE(a.addition_count, 0)) AS final_count
            FROM 
                standard s
            LEFT JOIN
                pizza_toppings pt ON pt.topping_id = s.standard
            LEFT JOIN 
                excluded e ON s.standard = e.removal
            LEFT JOIN 
                additional a ON s.standard = a.addition
            ORDER BY
                final_count DESC;

| topping_name | final_count |
| ------------ | ----------- |
| Bacon        | 12          |
| Mushrooms    | 11          |
| Cheese       | 10          |
| Pepperoni    | 9           |
| Salami       | 9           |
| Chicken      | 9           |
| Beef         | 9           |
| BBQ Sauce    | 8           |
| Tomato Sauce | 3           |
| Onions       | 3           |
| Peppers      | 3           |
| Tomatoes     | 3           |
