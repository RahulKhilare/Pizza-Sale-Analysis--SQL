Pizza Sales Analysis by using SQL

-- 1. Total orders

select count(order_id) as total_order
from orders;

-- 2. Calculate the total revenue generated from pizza sales

SELECT 
    round(SUM(p.price * od.quantity),2) AS total_revenue
FROM
    pizzas p
        JOIN
    order_details od ON p.pizza_id = od.pizza_id;


-- 3. Highest-priced pizza


SELECT 
    *
FROM
    pizzas
ORDER BY price DESC
LIMIT 1;


-- 4. Most common pizza size ordered

select size, count(order_id) as number_of_order
from pizzas p join order_details od
on p.pizza_id = od.pizza_id
group by size
order by number_of_order desc
limit 1;


-- 5. Top 5 most ordered pizza types

select 
pt.name,
sum(od.quantity) as total_order
from order_details od 
join pizzas p on od.pizza_id = p.pizza_id
join pizza_types pt on p.pizza_type_id= pt.pizza_type_id
group by pt.name
order by total_order desc
limit 5;



-- 1. Total quantity of each pizza category ordered

SELECT 
    pt.category, SUM(od.quantity) AS total_quantity
FROM
    order_details od
        JOIN
    pizzas p ON od.pizza_id = p.pizza_id
        JOIN
    pizza_types pt ON p.pizza_type_id = pt.pizza_type_id
GROUP BY pt.category
ORDER BY total_quantity DESC;

-- 2. Distribution of orders by hour

SELECT 
    CASE
        WHEN HOUR(order_time) BETWEEN 6 AND 11 THEN 'Morning'
        WHEN HOUR(order_time) BETWEEN 12 AND 17 THEN 'Afternoon'
        WHEN HOUR(order_time) BETWEEN 18 AND 22 THEN 'Evening'
        ELSE 'Night'
    END AS Time_Slot,
    count(order_id) as total_orders,
    (round(count(order_id) / (select count(order_id) from orders)*100,2)) as percentage_distribution
FROM
    orders
GROUP BY time_slot;


-- 4. Average number of pizzas ordered per day


SELECT 
    order_date, AVG(total_order) AS avg_order
FROM
    (SELECT 
        o.order_date, SUM(od.quantity) AS total_order
    FROM
        orders o
    JOIN order_details od ON o.order_id = od.order_id
    GROUP BY o.order_date) AS temp
GROUP BY order_date;


-- 5. Top 3 pizza types by revenue


SELECT 
    pt.pizza_type_id,
    SUM(od.quantity * p.price) AS total_revenue
FROM
    pizza_types pt
        JOIN
    pizzas p ON pt.pizza_type_id = p.pizza_type_id
        JOIN
    order_details od ON p.pizza_id = od.pizza_id
GROUP BY pt.pizza_type_id
ORDER BY total_revenue DESC
limit 3;


-- 1. Percentage contribution of each pizza type to total revenue

SELECT 
    pt.pizza_type_id,
    SUM(od.quantity * p.price) AS total_revenue,
    (round(SUM(od.quantity * p.price)/
    (select SUM(od.quantity * p.price) from order_details od join pizzas p on p.pizza_id = od.pizza_id)*100,2)) as percentage
FROM
    pizza_types pt
        JOIN
    pizzas p ON pt.pizza_type_id = p.pizza_type_id
        JOIN
    order_details od ON p.pizza_id = od.pizza_id
GROUP BY pt.pizza_type_id
ORDER BY total_revenue DESC;


-- 2. Cumulative revenue over time

SELECT 
    o.order_date, SUM(p.price * od.quantity) AS daily_revenue,
    sum(SUM(p.price * od.quantity)) over (order by o.order_date) as cumulative_revenue
FROM
    pizzas p
        JOIN
    order_details od ON p.pizza_id = od.pizza_id
        JOIN
    orders o ON od.order_id = o.order_id
GROUP BY o.order_date;


-- 3. Top 3 pizza types by revenue for each category


SELECT 
    pt.pizza_type_id,
    pt.category,
    SUM(p.price * od.quantity) AS revunue
FROM
    pizza_types pt
        JOIN
    pizzas p ON pt.pizza_type_id = p.pizza_type_id
        JOIN
    order_details od ON p.pizza_id = od.pizza_id
GROUP BY pt.pizza_type_id, pt.category
order by pt.pizza_type_id desc, pt.category
limit 3;






with top_3cte as (
SELECT 
    pt.pizza_type_id,
    pt.category,
    SUM(p.price * od.quantity) AS revunue,
    rank()over(partition by category order by SUM(p.price * od.quantity) desc) as rnk
FROM
    pizza_types pt
        JOIN
    pizzas p ON pt.pizza_type_id = p.pizza_type_id
        JOIN
    order_details od ON p.pizza_id = od.pizza_id
GROUP BY pt.pizza_type_id, pt.category
)
select 
pizza_type_id,
category,
revunue
from top_3cte
where rnk <=3;


-- Revenue of weekday and weekend

SELECT 
    CASE
        WHEN dayofweek(o.order_date) IN (1 , 7) THEN 'weekend'
        ELSE 'weekday'
    END AS days,
    round(SUM(od.quantity * p.price),2) AS total_revenue
FROM
    orders o
        JOIN
    order_details od ON o.order_id = od.order_id
        JOIN
    pizzas p ON od.pizza_id = p.pizza_id
GROUP BY days
ORDER BY total_revenue DESC;

