I. Import the dataset and do usual exploratory analysis steps like checking the
structure & characteristics of the dataset.

A. Data type of all columns in the “customers” table.

select column_name, data_type
from business_case_study.INFORMATION_SCHEMA.COLUMNS
where table_name = 'customers'



B.Get the time range between which the orders were placed.


select min(order_purchase_timestamp) as first_order, max(order_purchase_timestamp) as last_order
from `scalar-my-dsml-sql.business_case_study.orders`



C.Count the Cities & States of customers who ordered during the given period.

select distinct ord.order_purchase_timestamp, concat(cus.customer_city, ' ', cus.customer_state) as cities_state, count(ord.order_id)
from `scalar-my-dsml-sql.business_case_study.orders` ord join
`scalar-my-dsml-sql.business_case_study.customers` cus
on ord.customer_id = cus.customer_id 
where order_purchase_timestamp between '2016-09-04 21:15:19 UTC' and '2018-10-17 17:30:18 UTC'
group by 1 ,2
order by 3 desc




II. In-depth Exploration:

A. Is there a growing trend in the no. of orders placed over the past years?

select distinct format_datetime('%m', order_purchase_timestamp) as order_month, count(order_id) as order_count 
from `scalar-my-dsml-sql.business_case_study.orders`
group by 1
order by order_count, order_month 






C.)During what time of the day, do the Brazilian customers mostly place their
orders? (Dawn, Morning, Afternoon or Night)

● 0-6 hrs : Dawn
● 7-12 hrs : Mornings
● 13-18 hrs : Afternoon
● 19-23 hrs : Night


select interval_of_the_day, count(interval_of_the_day)
from (select order_purchase_timestamp, format_datetime('%H', order_purchase_timestamp) ordered_hour,
case
when format_datetime('%H', order_purchase_timestamp) between '00' and '06'
then 'Dawn'
when format_datetime('%H', order_purchase_timestamp) between '06' and '12'
then 'morning'
when format_datetime('%H', order_purchase_timestamp) between '13' and '18'
then 'afternoon'
when format_datetime('%H', order_purchase_timestamp) between '19' and '23'
then 'evening'
end as interval_of_the_day
from `scalar-my-dsml-sql.business_case_study.orders`
order by ordered_hour) 
group by 1
order by 2






3.Evolution of E-commerce orders in the Brazil region:

A.Get the month on month no. of orders placed in each state.

select count( cus.customer_id) order_count,extract(month from order_purchase_timestamp) as purchase_month, cus.customer_state
from `scalar-my-dsml-sql.business_case_study.orders` ord join `scalar-my-dsml-sql.business_case_study.customers` cus
on ord.customer_id = cus.customer_id
group by customer_state, purchase_month
order by 2 






--B.) How are the customers distributed across all the states?

select distinct count(ord.customer_id) as no_of_orders, cust.customer_state, cust.customer_city
from `scalar-my-dsml-sql.business_case_study.customers` as cust join
`scalar-my-dsml-sql.business_case_study.orders` as ord
on cust.customer_id = ord.customer_id
group by 2, 3
order by 1 desc



IV. Impact on Economy: Analyze the money movement by e-commerce by looking at
order prices, freight and others.

A. Get the % increase in the cost of orders from year 2017 to 2018 (include
months between Jan to Aug only).

SELECT
  EXTRACT(MONTH FROM o.order_purchase_timestamp) AS month,
  (
    (
      SUM(CASE WHEN EXTRACT(YEAR FROM o.order_purchase_timestamp) = 2018 AND
      EXTRACT(MONTH FROM o.order_purchase_timestamp) BETWEEN 1 AND 8 THEN
      p.payment_value END)
      -
      SUM(CASE WHEN EXTRACT(YEAR FROM o.order_purchase_timestamp) = 2017 AND
      EXTRACT(MONTH FROM o.order_purchase_timestamp) BETWEEN 1 AND 8 THEN
      p.payment_value END)
    )
    /
    SUM(CASE WHEN EXTRACT(YEAR FROM o.order_purchase_timestamp) = 2017 AND
    EXTRACT(MONTH FROM o.order_purchase_timestamp) BETWEEN 1 AND 8 THEN
    p.payment_value END)
  )*100 AS percent_increase
FROM
  `scalar-my-dsml-sql.business_case_study.orders` o
JOIN
  `scalar-my-dsml-sql.business_case_study.payments` p ON o.order_id = p.order_id
WHERE
  EXTRACT(YEAR FROM o.order_purchase_timestamp) IN (2017, 2018) AND
  EXTRACT(MONTH FROM o.order_purchase_timestamp) BETWEEN 1 AND 8
GROUP BY 1
ORDER BY 1;





B.Calculate the Total & Average value of order price for each state.

select
  cus.customer_state,
  round(avg(ite.price), 2) AS avg_price_value,
  round(sum(ite.price), 2) AS total_price_value
from
  `scalar-my-dsml-sql.business_case_study.orders` as ord
join
  `scalar-my-dsml-sql.business_case_study.order_items` ite ON ord.order_id = ite.order_id
join
  `scalar-my-dsml-sql.business_case_study.customers` cus ON ord.customer_id = cus.customer_id
group by
  cus.customer_state;




C. Calculate the Total & Average value of order freight for each state.

select
  cus.customer_state,
  round(avg(ite.freight_value), 2) AS avg_freight_value,
  round(sum(ite.freight_value), 2) AS total_freight_value
from
  `scalar-my-dsml-sql.business_case_study.orders` as ord
join
  `scalar-my-dsml-sql.business_case_study.order_items` ite ON ord.order_id = ite.order_id
join
  `scalar-my-dsml-sql.business_case_study.customers` cus ON ord.customer_id = cus.customer_id
group by
  cus.customer_state;







V. Analysis based on sales, freight and delivery time.

A. Find the no. of days taken to deliver each order from the order’s purchase date
as delivery time.

Also, calculate the difference (in days) between the estimated & actual delivery
date of an order.
Do this in a single query.


select
  order_id,
  date_diff(order_delivered_customer_date, order_purchase_timestamp, DAY) 
  as order_delivered_in_days,
  date_diff(order_estimated_delivery_date, order_delivered_customer_date, DAY) 
  as diff_estimated_delivery
from `scalar-my-dsml-sql.business_case_study.orders`
where date_diff(order_estimated_delivery_date, order_delivered_customer_date, DAY)  is not NULL
order by 2



B.Find out the top 5 states with the highest & lowest average freight value.

select cust.customer_state, round(avg(ord_it.freight_value), 2) as top_5_avg_freight_value
from `scalar-my-dsml-sql.business_case_study.customers` cust join
`scalar-my-dsml-sql.business_case_study.orders` ord 
on cust.customer_id = ord.customer_id
join
`scalar-my-dsml-sql.business_case_study.order_items` ord_it
on ord.order_id = ord_it.order_id
group by cust.customer_state
order by top_5_avg_freight_value desc
limit 5


select cust.customer_state, round(avg(ord_it.freight_value), 2) as bottom_5_avg_freight_value
from `scalar-my-dsml-sql.business_case_study.customers` cust join
`scalar-my-dsml-sql.business_case_study.orders` ord 
on cust.customer_id = ord.customer_id
join
`scalar-my-dsml-sql.business_case_study.order_items` ord_it
on ord.order_id = ord_it.order_id
group by cust.customer_state
order by bottom_5_avg_freight_value
limit 5


C.) Find out the top 5 states with the highest & lowest average delivery time.

SELECT
  cust.customer_state,
  ROUND(AVG(DATE_DIFF(order_delivered_customer_date, order_purchase_timestamp, DAY)), 2) 
  AS highest_avg_delivery_time,
FROM
  `scalar-my-dsml-sql.business_case_study.customers` cust
JOIN
  `scalar-my-dsml-sql.business_case_study.orders` ord ON ord.customer_id = cust.customer_id
WHERE
  DATE_DIFF(order_purchase_timestamp, order_delivered_customer_date, DAY) IS NOT NULL
  
GROUP BY cust.customer_state
ORDER BY 2 desc
limit 5
        



SELECT
  cust.customer_state,
  ROUND(AVG(DATE_DIFF(order_delivered_customer_date, order_purchase_timestamp, DAY)), 2) 
  AS lowest_avg_delivery_time,
FROM
  `scalar-my-dsml-sql.business_case_study.customers` cust
JOIN
  `scalar-my-dsml-sql.business_case_study.orders` ord ON ord.customer_id = cust.customer_id
WHERE
  DATE_DIFF(order_purchase_timestamp, order_delivered_customer_date, DAY) IS NOT NULL
  
GROUP BY cust.customer_state
ORDER BY 2 
limit 5
        


D.) Find out the top 5 states where the order delivery is really fast as compared to
the estimated date of delivery.
SELECT
  cust.customer_state,
  ROUND(AVG(DATE_DIFF(order_delivered_customer_date, order_purchase_timestamp, DAY)), 2) 
  AS avg_delivery,
  ROUND(AVG(DATE_DIFF(order_estimated_delivery_date, order_delivered_customer_date, DAY)), 2) 
  AS avg_actual_delivery
FROM
  `scalar-my-dsml-sql.business_case_study.customers` cust
JOIN
  `scalar-my-dsml-sql.business_case_study.orders` ord ON ord.customer_id = cust.customer_id
WHERE
  DATE_DIFF(order_purchase_timestamp, order_delivered_customer_date, DAY) IS NOT NULL
  AND
  DATE_DIFF(order_estimated_delivery_date, order_delivered_customer_date, DAY) IS NOT NULL
GROUP BY
  cust.customer_state
ORDER BY 3 desc
limit 5
        





VI. Analysis based on the payments:
A. Find the month on month no. of orders placed using different payment types.

SELECT
  pay.payment_type,
  EXTRACT(MONTH FROM ord.order_purchase_timestamp) AS month,
  COUNT(DISTINCT ord.order_id) AS order_count
FROM
  `scalar-my-dsml-sql.business_case_study.orders` ord
JOIN
  `scalar-my-dsml-sql.business_case_study.payments` pay
ON
  ord.order_id = pay.order_id
GROUP BY
  1, 2
ORDER BY
  1, 2;



 
B. Find the no. of orders placed on the basis of the payment installments that have
been paid.

SELECT
  paye.payment_installments,
  COUNT(ord.order_id) AS order_count
FROM
  `scalar-my-dsml-sql.business_case_study.orders`ord
  JOIN
  `scalar-my-dsml-sql.business_case_study.payments` paye
ON
  ord.order_id = paye.order_id
WHERE
  ord.order_status != 'paid'and paye.payment_installments >= 1
GROUP BY
  1
ORDER BY 1

