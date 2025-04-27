<img src="https://www.pngplay.com/wp-content/uploads/3/Amazon-Web-Services-AWS-Logo-PNG-HD-Quality.png">


## 📊 Project Title: Amazon USA Sales Analysis Project


Author:  Loc Huynh .

Date: Nov. 14, 2024 .

Tools Used: SQL .


## 📑 Table of Contents  
1. [📌 Background & Overview](#-background--overview)  
2. [📂 Dataset Description & Data Structure](#-dataset-description--data-structure)  
3. [🧠 Problem Solving Process](#-problem-solving-process)  
4. [📊 Explore the Dataset & Generate Insights](#-explore-the-dataset--generate_insights)  
5. [🔎 Final Conclusion & Recommendations](#-final-conclusion--recommendations)


## 📌 Background & Overview  

### Objective:
### 📖 What is this project about? 

I have worked on analyzing a dataset of over 20,000 sales records from an Amazon-like e-commerce platform. This project involves extensive querying of customer behavior, product performance, and sales trends using PostgreSQL. Through this project, I have tackled various SQL problems, including revenue analysis, customer segmentation, and inventory management.

The project also focuses on data cleaning, handling null values, and solving real-world business problems using structured queries.
### 👤 Who is this project for? 

➡️ **Marketing Manager** who want to understand the customer segmentation. 

###  ❓Business Questions:  

1.How can we analyze customer behavior, sales trends, and traffic performance to develop an effective new marketing strategy?
2. Low product availability due to inconsistent restocking.
3. High return rates for specific product categories.
4. Significant delays in shipments and inconsistencies in delivery times.
5 High customer acquisition costs with a low customer retention rate.


### 🎯Project Outcome:  

- **Key Insights**: Analysis reveals monthly trends, high traffic from Amazon all highlighting user engagement and behavior. Idenity the reason shippment late, Customer Life Time , AOV
- **Strategic Focus**: These findings emphasize the importance of data-driven strategies to enhance customer engagement, optimize eCommerce operations, and drive revenue growth.

---
## 📂 Dataset Description & Data Structure  

### 📌 Data Source  
- Source: Kaggles.
- Size: Over 5000 rows 
- Format: CSV


---

## **Database Setup & Design**

### **Schema Structure**

```sql
CREATE TABLE category
(
  category_id	INT PRIMARY KEY,
  category_name VARCHAR(20)
);

-- customers TABLE
CREATE TABLE customers
(
  customer_id INT PRIMARY KEY,	
  first_name	VARCHAR(20),
  last_name	VARCHAR(20),
  state VARCHAR(20),
  address VARCHAR(5) DEFAULT ('xxxx')
);

-- sellers TABLE
CREATE TABLE sellers
(
  seller_id INT PRIMARY KEY,
  seller_name	VARCHAR(25),
  origin VARCHAR(15)
);

-- products table
  CREATE TABLE products
  (
  product_id INT PRIMARY KEY,	
  product_name VARCHAR(50),	
  price	FLOAT,
  cogs	FLOAT,
  category_id INT, -- FK 
  CONSTRAINT product_fk_category FOREIGN KEY(category_id) REFERENCES category(category_id)
);

-- orders
CREATE TABLE orders
(
  order_id INT PRIMARY KEY, 	
  order_date	DATE,
  customer_id	INT, -- FK
  seller_id INT, -- FK 
  order_status VARCHAR(15),
  CONSTRAINT orders_fk_customers FOREIGN KEY (customer_id) REFERENCES customers(customer_id),
  CONSTRAINT orders_fk_sellers FOREIGN KEY (seller_id) REFERENCES sellers(seller_id)
);

CREATE TABLE order_items
(
  order_item_id INT PRIMARY KEY,
  order_id INT,	-- FK 
  product_id INT, -- FK
  quantity INT,	
  price_per_unit FLOAT,
  CONSTRAINT order_items_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id),
  CONSTRAINT order_items_fk_products FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- payment TABLE
CREATE TABLE payments
(
  payment_id	
  INT PRIMARY KEY,
  order_id INT, -- FK 	
  payment_date DATE,
  payment_status VARCHAR(20),
  CONSTRAINT payments_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

CREATE TABLE shippings
(
  shipping_id	INT PRIMARY KEY,
  order_id	INT, -- FK
  shipping_date DATE,	
  return_date	 DATE,
  shipping_providers	VARCHAR(15),
  delivery_status VARCHAR(15),
  CONSTRAINT shippings_fk_orders FOREIGN KEY (order_id) REFERENCES orders(order_id)
);

CREATE TABLE inventory
(
  inventory_id INT PRIMARY KEY,
  product_id INT, -- FK
  stock INT,
  warehouse_id INT,
  last_stock_date DATE,
  CONSTRAINT inventory_fk_products FOREIGN KEY (product_id) REFERENCES products(product_id)
  );
```
![image](https://github.com/user-attachments/assets/1052e054-3c77-4b12-8c2d-ee5cc735a459)

---

## 🧠 Problem Solving Process  

| 1️⃣ Understand Problem | 2️⃣ Break it down into smaller pieces | 3️⃣ Ideate | 4️⃣ Implement and Review 
|-|-|-|-
| Which to be calculated (sum, count, ratio, etc.) and grouped? <br> Does it needs any filter (time, conditions, etc.)? | Which tables have data that I want to get? <br> Which columns have data corresponded to the problem? <br> Can I get the data I by one step, if not, break down even smaller? | How many step did I need to get the final result? <br> Is it optimized? | _We'll go through this step in the order of each query listed below_ <br> ⬇️


---

## **Task: Data Cleaning**

I cleaned the dataset by:
- **Removing duplicates**: Duplicates in the customer and order tables were identified and removed.
- **Handling missing values**: Null values in critical fields (e.g., customer address, payment status) were either filled with default values or handled using appropriate methods.

---

## **Handling Null Values**

Null values were handled based on their context:
- **Customer addresses**: Missing addresses were assigned default placeholder values.
- **Payment statuses**: Orders with null payment statuses were categorized as “Pending.”
- **Shipping information**: Null return dates were left as is, as not all shipments are returned.

---

## **Solving Business Problems**

### Solutions Implemented:
#### Query 1️⃣: Query the top 10 products by total sales value / Include product name, total quantity sold, and total sales value.


```sql
SELECT 
	oi.product_id,
	p.product_name,
	SUM(oi.total_sale) as total_sale,
	COUNT(o.order_id)  as total_orders
FROM orders as o
JOIN
order_items as oi
ON oi.order_id = o.order_id
JOIN 
products as p
ON p.product_id = oi.product_id
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 10
```
![image](https://github.com/user-attachments/assets/b6207dc0-1177-4e91-b7ee-7cee4463c000)

💡 Apple's iMac 24-inch and iMac 27-inch Retina sell the most units. However, the Apple iMac Pro generates the highest revenue, suggesting it has a much higher price. Apple products, in general, seem to be strong performers. 

#### Query 2️⃣: Revenue by Category Calculate total revenue generated by each product category. / Challenge: Include the percentage contribution of each category to total revenue.


```sql
SELECT 
	p.category_id,
	c.category_name,
	SUM(oi.total_sale) as total_sale,
	SUM(oi.total_sale)/
					(SELECT SUM(total_sale) FROM order_items) 
					* 100
	as contribution
FROM order_items as oi
JOIN
products as p
ON p.product_id = oi.product_id
LEFT JOIN category as c
ON c.category_id = p.category_id
GROUP BY 1, 2
ORDER BY 3 DESC
```

![image](https://github.com/user-attachments/assets/7f54065c-1d21-41cf-93bc-e6e8da96b96f)

💡 The primary insight is the massive revenue contribution from the electronics category, dwarfing all other categories. This highlights the current business focus and potential areas for strategic consideration regarding diversification and resource allocation.

#### Query 3️⃣:  Average Order Value (AOV) Compute the average order value for each customer./Include only customers with more than 5 orders.



```sql
SELECT 
	c.customer_id,
	CONCAT(c.first_name, ' ',  c.last_name) as full_name,
	SUM(total_sale)/COUNT(o.order_id) as AOV,
	COUNT(o.order_id) as total_orders --- filter
FROM orders as o
JOIN 
customers as c
ON c.customer_id = o.customer_id
JOIN 
order_items as oi
ON oi.order_id = o.order_id
GROUP BY 1, 2
HAVING  COUNT(o.order_id) > 5
```
![image](https://github.com/user-attachments/assets/7da284a9-be21-4839-8059-77dd8d3cc52d)


💡 Focusing on customers with more than 5 orders reveals a range of spending behaviors. While some frequent customers consistently make high-value purchases, others place many lower-value orders. Understanding these nuances can help tailor marketing efforts and optimize customer retention strategies.

#### Query 4️⃣:  Monthly Sales Trend Query monthly total sales over the past year. / Challenge: Display the sales trend, grouping by month, return current_month sale, last month sale!


```sql
SELECT 
	year,
	month,
	total_sale as current_month_sale,
	LAG(total_sale, 1) OVER(ORDER BY year, month) as last_month_sale
FROM ---
(
SELECT 
	EXTRACT(MONTH FROM o.order_date) as month,
	EXTRACT(YEAR FROM o.order_date) as year,
	ROUND(
			SUM(oi.total_sale::numeric)
			,2) as total_sale
FROM orders as o
JOIN
order_items as oi
ON oi.order_id = o.order_id
WHERE o.order_date >= CURRENT_DATE - INTERVAL '1 year'
GROUP BY 1, 2
ORDER BY year, month
) as t1
```
![image](https://github.com/user-attachments/assets/a34f664e-c7e4-4fbe-ac72-47b935cf6646)

💡 The data highlights significant volatility in monthly sales, with a particularly concerning drop in the most recent month (March 2025). Further investigation is needed to understand the drivers behind these fluctuations and to identify any underlying trends or seasonality.

#### Query 5️⃣: Customers with No Purchases Find customers who have registered but never placed an order. / Challenge: List customer details and the time since their registration.



```sql
Approach 1
SELECT *
	-- reg_date - CURRENT_DATE
FROM customers
WHERE customer_id NOT IN (SELECT 
					DISTINCT customer_id
				FROM orders
				);
```

![image](https://github.com/user-attachments/assets/d5c882be-f95c-4ec7-a2a0-e78543f4b595)

💡 Develop a strategy to regain customer attention by calling them or sending an email to understand the reasons why they've become disengaged.

#### Query 6️⃣: Least-Selling Categories by State Identify the least-selling product category for each state. / Challenge: Include the total sales for that category within each state.

```sql
WITH ranking_table
AS
(
SELECT 
	c.state,
	cat.category_name,
	SUM(oi.total_sale) as total_sale,
	RANK() OVER(PARTITION BY c.state ORDER BY SUM(oi.total_sale) ASC) as rank
FROM orders as o
JOIN 
customers as c
ON o.customer_id = c.customer_id
JOIN
order_items as oi
ON o.order_id = oi. order_id
JOIN 
products as p
ON oi.product_id = p.product_id
JOIN
category as cat
ON cat.category_id = p.category_id
GROUP BY 1, 2
)
SELECT 
*
FROM ranking_table
WHERE rank = 1
```
![image](https://github.com/user-attachments/assets/057c2db3-b7ac-4053-8085-7158d76ff580)

💡 Interesting Alabama only sell Electronic Need to takle this point. In addition, we can get to know more each category has sold in each state
#### Query 7️⃣:  Customer Lifetime Value (CLTV) Calculate the total value of orders placed by each customer over their lifetime./ Challenge: Rank customers based on their CLTV.



```sql
SELECT 
	c.customer_id,
	CONCAT(c.first_name, ' ',  c.last_name) as full_name,
	SUM(total_sale) as CLTV,
	DENSE_RANK() OVER( ORDER BY SUM(total_sale) DESC) as cx_ranking
FROM orders as o
JOIN 
customers as c
ON c.customer_id = o.customer_id
JOIN 
order_items as oi
ON oi.order_id = o.order_id
GROUP BY 1, 2
```
![image](https://github.com/user-attachments/assets/26a53ceb-5481-4ca4-8e0f-29f515cbbf6f)

💡 The CLTV ranking provides a clear understanding of the long-term value of individual customers. Focusing on retaining and nurturing high-CLTV customers while exploring strategies to increase the value of other segments can contribute significantly to the business's success.

#### Query 8️⃣:  8. Inventory Stock Alerts Query products with stock levels below a certain threshold (e.g., less than 10 units). / Challenge: Include last restock date and warehouse information.

```sql
SELECT 
	i.inventory_id,
	p.product_name,
	i.stock as current_stock_left,
	i.last_stock_date,
	i.warehouse_id
FROM inventory as i
join 
products as p
ON p.product_id = i.product_id
WHERE stock < 10
```
![image](https://github.com/user-attachments/assets/6a93e044-8718-4e34-85af-dfb9c4e0c659)
💡 the data signals a critical need for inventory replenishment across the board in warehouse 1. Understanding the reasons behind these low stock levels and optimizing inventory management processes are essential to prevent stockouts and ensure customer demand can be met.

#### Query 9️⃣:  9. Shipping Delays Identify orders where the shipping date is later than 3 days after the order date. /Challenge: Include customer, order details, and delivery provider.



```sql
SELECT 
	c.*,
	o.*,
	s.shipping_providers,
s.shipping_date - o.order_date as days_took_to_ship
FROM orders as o
JOIN
customers as c
ON c.customer_id = o.customer_id
JOIN 
shippings as s
ON o.order_id = s.order_id
WHERE s.shipping_date - o.order_date > 3
```
![image](https://github.com/user-attachments/assets/a48b863d-a247-4f4c-a48b-49ff2f898297)

💡 Need to work on delivery provider to know the reason behind. Why the product has been shipped lately. In addtion, we can know the reason why some product has been returned

#### Query 🔟:   Payment Success Rate  Calculate the percentage of successful payments across all orders. / Challenge: Include breakdowns by payment status (e.g., failed, pending).



```sql
SELECT 
	p.payment_status,
	COUNT(*) as total_cnt,
	COUNT(*)::numeric/(SELECT COUNT(*) FROM payments)::numeric * 100
FROM orders as o
JOIN
payments as p
ON o.order_id = p.order_id
GROUP BY 1
```

![image](https://github.com/user-attachments/assets/2db3554e-780c-45b6-89d9-4834b28bfbed)

💡 the data highlights a strong payment success rate but also points to a significant issue with refunds that needs to be addressed. Understanding the drivers of these refunds is key to improving profitability and customer satisfaction. The low payment failure rate suggests a well-functioning payment processing system.

#### Query 1️⃣1️⃣ :   11. Top Performing Sellers Find the top 5 sellers based on total sales value. / Challenge: Include both successful and failed orders, and display their percentage of successful orders.



```sql
WITH top_sellers
AS
(SELECT 
	s.seller_id,
	s.seller_name,
	SUM(oi.total_sale) as total_sale
FROM orders as o
JOIN
sellers as s
ON o.seller_id = s.seller_id
JOIN 
order_items as oi
ON oi.order_id = o.order_id
GROUP BY 1, 2
ORDER BY 3 DESC
LIMIT 5
),

sellers_reports
AS
(SELECT 
	o.seller_id,
	ts.seller_name,
	o.order_status,
	COUNT(*) as total_orders
FROM orders as o
JOIN 
top_sellers as ts
ON ts.seller_id = o.seller_id
WHERE 
	o.order_status NOT IN ('Inprogress', 'Returned')
	
GROUP BY 1, 2, 3
)
SELECT 
	seller_id,
	seller_name,
	SUM(CASE WHEN order_status = 'Completed' THEN total_orders ELSE 0 END) as Completed_orders,
	SUM(CASE WHEN order_status = 'Cancelled' THEN total_orders ELSE 0 END) as Cancelled_orders,
	SUM(total_orders) as total_orders,
	SUM(CASE WHEN order_status = 'Completed' THEN total_orders ELSE 0 END)::numeric/
	SUM(total_orders)::numeric * 100 as successful_orders_percentage
	
FROM sellers_reports
GROUP BY 1, 2
```
![image](https://github.com/user-attachments/assets/3923b03a-e571-449c-acde-bc0d57976d65)
💡 We should offer a compliment or a reward to the top five sellers who have worked diligently to achieve their targets. Additionally, we should invite them to share a story or key to their success to help other sellers reach their goals


#### Query 1️⃣2️⃣:  Product Profit Margin Calculate the profit margin for each product (difference between price and cost of goods sold). / Challenge: Rank products by their profit margin, showing highest to lowest.


```sql
SELECT 
	product_id,
	product_name,
	profit_margin,
	DENSE_RANK() OVER( ORDER BY profit_margin DESC) as product_ranking
FROM
(SELECT 
	p.product_id,
	p.product_name,
	-- SUM(total_sale - (p.cogs * oi.quantity)) as profit,
	SUM(total_sale - (p.cogs * oi.quantity))/sum(total_sale) * 100 as profit_margin
FROM order_items as oi
JOIN 
products as p
ON oi.product_id = p.product_id
GROUP BY 1, 2
) as t1
```
![image](https://github.com/user-attachments/assets/a8f1fd66-641d-4481-9b11-f8d0bcc2ede6)

💡To gain deeper insights into the business, it's imperative to ascertain which product exhibits the most substantial profit margin and which yields the scantest return. This granular understanding will enable the formulation of more efficacious and tailored strategies for each product within our portfolio.

#### Query 1️⃣3️⃣:   Most Returned Products Query the top 10 products by the number of returns. / Challenge: Display the return rate as a percentage of total units sold for each product.
```sql
SELECT 
	p.product_id,
	p.product_name,
	COUNT(*) as total_unit_sold,
	SUM(CASE WHEN o.order_status = 'Returned' THEN 1 ELSE 0 END) as total_returned,
	SUM(CASE WHEN o.order_status = 'Returned' THEN 1 ELSE 0 END)::numeric/COUNT(*)::numeric * 100 as return_percentage
FROM order_items as oi
JOIN 
products as p
ON oi.product_id = p.product_id
JOIN orders as o
ON o.order_id = oi.order_id
GROUP BY 1, 2
ORDER BY 5 DESC
```
![image](https://github.com/user-attachments/assets/7ebc3e7c-66ed-4663-bbae-cae8d67bfc36)
💡 Understanding which product has the best and worst profit margins is key to creating smarter plans for each one.




#### Query 1️⃣4️⃣:  if the customer has done more than 5 return categorize them as returning otherwise new / Challenge: List customers id, name, total orders, total returns

```sql
SELECT
    c.first_name + ' ' + c.last_name AS customers,
    COUNT(o.order_id) AS total_orders,
    SUM(CASE WHEN o.order_status = 'Returned' THEN 1 ELSE 0 END) AS total_return,
    CASE
        WHEN SUM(CASE WHEN o.order_status = 'Returned' THEN 1 ELSE 0 END) > 5 THEN 'Returning_customers' ELSE 'New'
    END AS cx_category
FROM
    Orders AS o
JOIN
    Customers AS c ON c.customer_id = o.customer_id
JOIN
    Order_Items AS oi ON oi.order_id = o.order_id
GROUP BY
    c.first_name, c.last_name
```

💡 Have a fit stragegy for each segment customer. For example give "New Customer" High-quality products/services/ High content useful. and for "Return Customer" sending more voucher, give them more Personalize the experience, Special welcome experience,Act on feedback

## **Conclusion**

This advanced SQL project successfully demonstrates my ability to solve real-world e-commerce problems using structured queries. From improving customer retention to optimizing inventory and logistics, the project provides valuable insights into operational challenges and solutions.

By completing this project, I have gained a deeper understanding of how SQL can be used to tackle complex data problems and drive business decision-making.

---


