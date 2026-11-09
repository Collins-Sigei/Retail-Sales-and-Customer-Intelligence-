# Retail Sales SQL Analysis

This is a project of Duka Smart Electronics, a growing Kenyan retailer selling phones, laptops, accessories, TVs and home appliances in ten towns that is Nairobi, Mombasa, Kisumu, Nakuru, Eldoret, Thika, Nyeri, Meru, Machakos and Kiambu. 
It has traded since January 2025 and keeps every order in a database. Entails loading a customer and order data and answers business questions with SQL. Each query returns a result that is interpreted and provides business insights

## What's in the data

| Table | Rows | What it holds |
|---|---|---|
| `customers` | 150 | Customer name, gender, city, registration date, customer segment (Retail, VIP, Corporate) |
| `orders` | 800 | Orders from Jan 2025 to Sep 2026: product, category, quantity, unit price, cost price, discount, payment method, status |

- **Currency** 
- **Order status** 
- **Discount** 
- **Cities** 
- **Product categories** 

## Project structure

```
├── README.md
├── sql/
│   ├── 01_readme.md
│   ├── 02_schema.sql
│   ├── 03_queries.sql
│   └── 04_insights.md
│   ├── screenshots/ verify_data

```
## Setup
### 1. Create the schema

```sql
create schema retail_analytics;
show search_path;
set search_path to retail_analytics;
```
### 2. Create the tables
```sql
-- creating table customers
create table customers(
    customer_id int primary key,
    customer_name Varchar(100) not null,
    gender Varchar(10) check (gender IN ('Male', 'Female')),
    city Varchar(50),
    registration_date Date,
    customer_segment Varchar(20) Check (customer_segment IN ('Retail', 'Corporate', 'VIP'))
);

-- create table orders
create table orders(
    order_id          INTEGER       PRIMARY KEY,
    customer_id       INTEGER       NOT NULL REFERENCES customers (customer_id),
    order_date        DATE          NOT NULL,
    product_name      VARCHAR(100),
    product_category  VARCHAR(50),
    quantity          INTEGER       CHECK (quantity > 0),
    unit_price        NUMERIC(12,2) CHECK (unit_price >= 0),
    cost_price        NUMERIC(12,2) CHECK (cost_price >= 0),
    discount          NUMERIC(5,2)  DEFAULT 0 CHECK (discount BETWEEN 0 AND 100),
    payment_method    VARCHAR(30)   CHECK (payment_method IN ('M-Pesa', 'Cash', 'Card', 'Bank Transfer')),
    order_status      VARCHAR(20)   CHECK (order_status IN ('Completed', 'Pending', 'Cancelled'))
);
```
### 3. Running the queries

```sql
/*Section A*/
--Q1. Retrieve all customer records 
select *from customers;

-- Q2. Display customer names, cities and segments
select customer_name, city, customer_segment
from customers;

-- Q3. Find customers living in Nairobi 
select customer_id, customer_name, city
from customers
where city = 'Nairobi';

-- Q4. Find customers registered after January 1, 2025
select customer_id, customer_name, registration_date
from customers
where registration_date >'2025-01-01';

-- Q5. Display all unique customer cities
select distinct city
from customers;

-- Q6. Retrieve the 10 most recent orders
select order_id, product_name, order_date
from orders
order by order_date desc
limit 10;

-- Q7. Find orders with unit prices above Ksh. 10,000
select order_id, product_name, unit_price
from orders
where unit_price>10000;

-- Q8. Find customers whose names starts with A
select customer_id, customer_name
from customers
where customer_name like 'A%';

-- Q9. Retrieve orders with quantities between 2 and 5
select order_id, product_name, quantity
from orders
where quantity between 2 and 5;

-- Q10. Find orders paid through Mpesa
select order_id, product_name, payment_method
from orders
where payment_method='M-Pesa';

/*Section B*/
-- Q11. Count all customers
select count(*) as total_customers
from customers;

-- Q12. Count all orders
select count(*) as total_orders
from orders;

-- Q13. Calculate completed-order revenue
select sum(quantity*unit_price*(1-discount/100)) as total_revenue
from orders
where order_status = 'Completed';

-- Q14. Calculate the average completed order value
select avg(quantity*unit_price*(1-discount/100)) as avg_order_value
from orders
where order_status = 'Completed';

-- Q15. Find the highest and lowest product prices
select max(unit_price) as highest_price, min(unit_price) as lowest_price
from orders;

-- Q16. Calculate revenue by product category
select product_category, sum(quantity*unit_price*(1-discount/100)) as total_revenue
from orders
where order_status = 'Completed'
group by product_category
order by total_revenue desc;

-- Q17. Count customers by city 
select count(customer_name) as customers, city
from customers
group by city;

-- Q18. Calculate quantity sold by product
select sum(quantity) as quantity_sold, product_name
from orders
where order_status = 'Completed'
group by product_name;

-- Q19. Find categories generating more than Ksh 100,000 in revenue
select product_category, sum(quantity*unit_price*(1-discount/100)) as revenue
from orders
where order_status = 'Completed'
group  by product_category
having sum(quantity*unit_price*(1-discount/100)) > 100000;

-- Q20. Calculate the percentage of cancelled orders
select 100.0 * count(*) filter (where order_status = 'Cancelled')/count(*) as cancelled_orders_percentage
from orders;

/*Section C*/
-- Q21. Display orders with customers names and cities 
select order_id o, customer_name c, city c
from orders o
inner join customers c
on o.customer_id=c.customer_id;

-- Q22. Calculate total spending by customer 
select c.customer_id, c.customer_name, sum(quantity*unit_price*(1-discount/100)) as total_spending
from customers c 
inner join orders o
on c.customer_id=o.customer_id
where o.order_status = 'Completed'
group by c.customer_id
order by c.customer_id asc;

-- Q23. Identify customers who have never ordered
select c.customer_id, c.customer_name, o.order_status 
from customers c 
left join orders o
on c.customer_id=o.customer_id 
where order_id is null;

-- Q24. Find the 10 highest-spending customers
select c.customer_id, c.customer_name, sum(quantity*unit_price*(1-discount/100)) as total_spending
from customers c 
inner join orders o 
on c.customer_id=o.customer_id
where o.order_status = 'Completed'
group by c.customer_id
order by sum(quantity*unit_price*(1-discount/100)) desc
limit 10;

-- Q25. Calculate revenue by customer city
select city, sum(quantity*unit_price*(1-discount/100)) as revenue
from customers c 
inner join orders o 
on c.customer_id=o.customer_id
where o.order_status='Completed'
group by c.city
order by revenue desc;

-- Q26. Identify customers with more than five orders
select c.customer_id, c.customer_name, count(o.order_id) as total_orders
from customers c
inner join orders o 
on c.customer_id = o.customer_id
group  by c.customer_id 
having count(o.order_id) > 5;

-- Q27. Calculate average customer spending by segment

select segment, count(*) as customers, avg(total_spending) as avg_customer_spending
from (
select c.customer_id, c.customer_name, c.customer_segment as segment, sum(quantity*unit_price*(1-discount/100)) as total_spending
from customers c
inner join orders o 
on c.customer_id=o.customer_id 
where o.order_status = 'Completed'
group by c.customer_id, c.customer_name, c.customer_segment
)
as per_customer
group by segment;
/*Found each cusomer's total spending by summing up the revenue across the completed orders, then averaged those customer totals within each customer segment and later grouped them by customer segment.*/

-- Q28. Identify the most popular category among VIP customers
select c.customer_segment, o.product_category, sum(o.quantity) as orders
from customers c 
inner join orders o 
on c.customer_id=o.customer_id 
where c.customer_segment = 'VIP'
and o.order_status = 'Completed'
group by c.customer_segment, o.product_category
order by orders desc
limit 1;

/*To get the most popular category, I used the quantity ordered only from the completed orders*/

/*Section D*/

-- Q29. Find customers spending above the average total spending per purchasing customer
with customer_totals as ( 
select c.customer_id, c.customer_name, sum(o.quantity*o.unit_price*(1-o.discount/100)) as total_spending
from customers c 
inner join orders o 
on c.customer_id = o.customer_id 
where o.order_status='Completed'
group by c.customer_id
)
select customer_id, customer_name, total_spending
from customer_totals 
where total_spending >(select avg(total_spending) from customer_totals)
order by total_spending desc;

-- Q30. Find products priced above the overall average unit price
select distinct o.product_name, o.unit_price
from orders o
where unit_price > (select avg(unit_price) from orders);

-- Q31. Use a CTE to identify the three highest revenue months
with monthly_revenue as (
select date_trunc('month', order_date) as month,
sum(quantity*unit_price*(1-discount/100)) as total_revenue
from orders 
where order_status = 'Completed'
group by date_trunc('month', order_date)
)
select to_char(month, 'yyyy-mm') as month, total_revenue
from monthly_revenue 
order by total_revenue desc
limit 3;

-- Q32. Find customers purchasing from at least three categories
select c.customer_id, c.customer_name, count(distinct o.product_category) as categories_bought
from customers c 
inner join orders o 
on c.customer_id=o.customer_id 
where o.order_status = 'Completed'
group by c.customer_id
having count(distinct o.product_category)>=3
order by categories_bought desc;

-- Q33. Use a CTE to calculate category revenue, cost, profit and profit margin
with category_totals as (
select product_category, sum(quantity*unit_price*(1-discount/100)) as total_revenue, sum(quantity*cost_price) as cost
from orders
where order_status = 'Completed'
group by product_category
)
select product_category, round(total_revenue::numeric,2) as category_revenue, round(cost::numeric,2) as cost,
round ((total_revenue-cost)::numeric,2) as profit, round((100*(total_revenue-cost)/total_revenue)::numeric,2) as profit_margin
from category_totals;

-- Q34. Find customers whose latest completed order is more than 90 days before the latest order date in the dataset
with latest_completed as (
select customer_id, max(order_date) as last_completed_date 
from orders
where order_status='Completed'
group by customer_id 
),
dataset_max as (
select max(order_date) as max_date 
from orders
)
select c.customer_id, c.customer_name, lc.last_completed_date, dm.max_date - lc.last_completed_date as days_before_latest
from latest_completed lc
inner join customers c
on c.customer_id=lc.customer_id
cross join dataset_max dm
where dm.max_date - lc.last_completed_date > 90
order by days_before_latest desc;

/*Section E*/
-- Q35. Rank customers by spending using 'RANK'
select rank() over (order by sum(o.quantity*o.unit_price*(1-o.discount/100))desc) as spending, c.customer_id, c.customer_name,
sum(o.quantity*o.unit_price*(1-o.discount/100)) as total_spending
from customers c
inner join orders o
on c.customer_id=o.customer_id
where o.order_status = 'Completed'
group by c.customer_id
order by spending;

-- Q36. Number each customer's orders by recency using 'ROW NUMBER()'
select customer_id, order_id, product_name, order_status,
row_number() over (partition by customer_id order by order_date desc, order_id desc) as order_recency
from orders
order by customer_id;

-- Q37. Rank products within categories using 'DENSE RANK()'
with product_revenue as (
select product_category, product_name, sum(quantity*unit_price*(1-discount/100)) as total_revenue
from orders 
where order_status = 'Completed'
group by product_category, product_name 
)
select product_category, product_name, round(total_revenue::numeric, 2) as total_revenue,
dense_rank() over(partition by product_category order by total_revenue desc) as category_rank
from product_revenue
order by product_category, category_rank;

-- Q38. Calculate cumulative monthly revenue
with monthly_revenue as (
select date_trunc('month', order_date) as month, sum(quantity*unit_price*(1-discount/100)) as total_revenue
from orders
where order_status = 'Completed'
group by date_trunc('month', order_date)
)
select to_char(month, 'yyyy-mm') as month, round(total_revenue::numeric,2) as monthly_revenue, round(sum(total_revenue)
over (order by month)::numeric,2) as cumulative_revenue
from monthly_revenue;

-- Q39. Compare monthly revenue using 'LAG()'
with monthly_revenue as (
select date_trunc('month', order_date) as month, sum(quantity*unit_price*(1-discount/100)) as total_revenue
from orders
where order_status = 'Completed'
group by date_trunc('month', order_date)
)
select to_char(month, 'yyyy-mm') as month, round(total_revenue::numeric,2) as total_revenue,
round(lag(total_revenue)over(order by month)::numeric,2) as previous_month, round((total_revenue-lag(total_revenue) over(order by month))::numeric,2) as change,
round ((100 *(total_revenue-lag(total_revenue) over (order by month))/lag(total_revenue) over (order by month))::numeric,2) as percent_change
from monthly_revenue;

-- Q40. Calculate month-over-month revenue growth %
with monthly_revenue as (
select date_trunc('month',order_date) as month, sum(quantity*unit_price*(1-discount/100)) as total_revenue
from orders
where order_status = 'Completed'
group by date_trunc('month', order_date)
),
with_previous as (
select month, total_revenue, lag(total_revenue)over(order by month) as prev_revenue
from monthly_revenue
)
select to_char(month, 'yyyy-mm') as month, round(total_revenue::numeric,2) as total_revenue, round((100* (total_revenue-prev_revenue)/nullif(prev_revenue,0))::numeric,2) as mom_growth_percent
from with_previous;

-- Q41. Divide customers into four spending groups using 'NTILE(4)'
with customer_totals as (
select c.customer_id, c.customer_name, sum(o.quantity*o.unit_price*(1-o.discount/100)) as total_spending
from customers c 
inner join orders o 
on c.customer_id = o.customer_id
where o.order_status = 'Completed'
group by c.customer_id
)
select customer_id, customer_name, round(total_spending::numeric,2) as total_spending, NTILE(4) over (order by total_spending desc) spending_group
from customer_totals;

-- Q42. Calculate each category's share of total revenue using window functions
select product_category, round(sum(quantity*unit_price*(1-discount/100))::numeric,2) as total_revenue, 
round((100*sum(quantity*unit_price*(1-discount/100))/sum(sum(quantity*unit_price*(1-discount/100)))over())::numeric,2) as percent_of_total
from orders
where order_status = 'Completed'
group by product_category;

/*Section F*/
-- Q43. Create a 'monthly_sales_summary' view (month, orders, revenue, cost, profit). Then `SELECT` from it.
create or replace view monthly_sales_summary as 
select date_trunc('month', order_date)::date as month, count(*) as orders, round(sum(quantity*unit_price*(1-discount/100))::numeric,2) as revenue,
round(sum(quantity*cost_price)::numeric,2) as cost, round((sum(quantity*unit_price*(1-discount/100)) - sum(quantity*cost_price))::numeric,2) as profit
from orders
where order_status = 'Completed'
group by date_trunc('month', order_date);
select *
from monthly_sales_summary;

-- Q44. Segment customers into High, Medium and Low Value. **Explain why you chose your thresholds** 
-- for example percentiles, or round Ksh amounts based on what the data looks like.
with customer_totals as (
select c.customer_id, c.customer_name, coalesce(sum(quantity*unit_price*(1-discount/100)), 0) as total_spending
from customers c
left join orders o 
on c.customer_id = o.customer_id 
and o.order_status = 'Completed'
group by c.customer_id,c.customer_name 
)
select customer_id,customer_name, round(total_spending::numeric,2) as total_spending, case
	when total_spending >= 250000 then 'High Value'
	when total_spending >= 50000 then 'Medium Value'
	else 'Low Value'
	end as value_segment
from customer_totals 
order by customer_totals.total_spending desc; 
/*I sorted the customers by how much they’ve spent
 * I set high value to be Ksh 250,000 or more, medium value to be Ksh below 250000 to 50000 and low value to be under Ksh 50000 
 * The 75th percentile of customer spending is Ksh 252,835, so I rounded down to KSh 250,000 to be High Value
 *  */

-- Q46. Produce a management report: the top five customers, top five products, the most profitable
-- category, and the lowest-revenue month
/*1. Top five customers;
 * - Brian Chebet
 * - Peter Ndungu 
 * - Lucy Njoroge
 * - Nancy Korir
 * - Purity Ochieng 
 * 
 * 2. Top five products by reveue 
 * - Iphone 13
 * - HP 250 G9 Laptop
 * - Hotpoint Refrigerator
 * - Lenovo IdeaPad 3
 * - Samsung 55in Smart TV
 * 
 * 3. The most profitable category
 * - Mobile phones
 * 
 * 4. Lowest-revenue month 
 * - 2026, January */

```
### 4. Insights
#### 1. Mobile Phones brings the most revenue _Q.16_
- Mobile phones bring in the most revenue in the business followed by computing as a product category, while accessories bring the least revenue. 
- The revenue from the mobile phones bring the highest revenue at about 28% and therefore any problem with the mobile phones category would affect the business. 
- Recommendation: To keep re-stocking the mobile phones to avoid any supply issues and ensure the best sellers of in the mobile phones category are always available. 

#### 2. Vip customers spend the most each but retail bring in most money _(Q.27)_
- Vip customers spend the most each about Ksh 253,000 then Corporate at about Ksh 198,000 and Retail at about Ksh 164,000. Retail still brings in the most money overall at about 60% of the total. 
- Corporate spends more than retail customers and therefore, since there are more retail customers even a small rise in what each one spends will make a big difference to the business
- Recommendation: Encourage retail customers to spend a bit more each by selling items together at a small discount or rewarding repeat buyers with shopping points that are redeemable each time they buy. For Corporate clients, provide a better price for big orders. 

#### 3. Sales are growing but fall every January _(Q.40)_
- Sales rise and fall from month to month. In  December 2025 sales were about Ksh 2.0 million while in January 2026 they fell to around Ksh 322,000, and then more than doubled in February of the same year. September 2026 was more than double September 2025.
- December high followed by a quiet January means money and stock can gets small at the start of the year. January 2026 was even lower than January 2025. 
- Recommendation: To plan stock and cash around the December peak so January doesn’t catch unprepared, then run an offer in January aimed at customers who bought in December, to keep them buying. 

#### 4. Nairobi brings in more revenue while Machakos brings in the least _(Q.25)_
- Nairobi brings in Ksh 9.5M, about 40% of all completed-order revenue. Machakos brings in less at Ksh 745,099.
- The business depends more in terms of revenue from two cities that is Nairobi and Mombasa, so a stock-out delivery problem or competitor move in Nairobi would hit the whole business.
- Recommendation: Protect Nairobi and Mombasa first by prioritising stock and delivery speed there to ensure customer retention. In Machakos, try targeted promotions and look into what they buy and how often. 

#### 5. Growth is steady and continues over time _(Q.38)_
- The average month in 2025 brought in KES 944K, while the average month in 2026 brought in Ksh 1.43M. 
- The business is growing steadily from 2025 to 2026 but revenue is seasonal, with a festive peak in December followed by a sharp January fall. That affects cash flow, because costs such as rent and salaries continue through the slow months.
- Recommendation: To plan stock and cash around the December peak and the January fall, ensuring enough reserves in Nov-Dec to cover the peak. Run a January promotion like back-to-school or new-year offers to further improve that month's revenue.
- 
