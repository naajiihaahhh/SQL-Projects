# Foodpanda SQL Project

## 📝Context 
This project analyzes **6,000 Foodpanda Records** , integrating cstomer information, order history, payments, and ratings. The dataset provides insights into user behavior, reaturant performance, and engagement patterns, which are useful for studying customer churn, spending habits, restaurant and food popularity, and delivery efficiency. 

---

## 💹Dataset Content 
Columns : customer_id, gender, age, city, signup_date, order_id, order_date, restaurant_name, dish_name, category, quantity, price, payment_method, order_frequency, last_order_date_loyalty_points, churned, rating, rating_date, delivery_status
- **Customer Demographics**
    - Gender, age range, city, signup_date, churn status
- **Order_Level Data**
    - Restaurant names, dishes, categories, quantities, prices, delivery results
- **Engagement etrics**
    - Order frequency, last order date, loyalty points
- **Feedback & Payments**
    - Ratings, payment method
 
Together, this dataset gives a **holistic view of customer activity and bussiness performance** in a food delivery platform.

---

## 👉Business Problem Statement
Although FoodPanda collect large amounts of data, the buisness faces challenges is using it effectively to: 
- Identify reasons for **customer churn** and improve retention
- -Determine **top-performing restaurants and dishes**
- Monitor and reduce **delayed or cancelled deliveries**
- Design better **promotions and loyalty programs**

---

## 🚀Project Objectives
Use SQL to tranform raw FoodPanda data into actionabl insights that help the business improve **customer satisfaction and delivery efficiency**. 

---

## 🔍SQL Analysis Scope 
This project answers key business questions such as:
1. What are the **top 3 restaurants** by revenue and average rating?
2. What is the **customer churn rate**, and which groups are most affected?
3. What is the **average order values** by city, gender, age group?
4. Which restaurants have the **highest % if delyed deliveries**?
5. What are the **most popular dishes and cuisines**?
6. How do **monthly order trends** change over time?

---

## ✔️Key Skills and  Tools
- SQL (SELECT, WHERE, GROUP BY, CTEs, ORDER BY, Window Functions)
- Data Cleaning and Transformation
- Business Analysis and Insight Generation

---

## 👨‍🔧Data Cleaning Process 
- Create new table with a copy of the original table to do the Data Cleaning and Standardizing without affecting the original data
- Converted date into YYYY-mm-dd , from text to date format

--- 
## 🔍Exploratory Analysis
1. What are the **top 3 restaurants** by revenue and average rating?

SQL Code: 
```SQL
SELECT 
	restaurant_name, 
    ROUND(AVG(rating), 2) AS avg_rating, 
    ROUND(SUM(price), 2) AS total_revenue
FROM foodpanda_review2
WHERE delivery_status = 'Delayed' OR 'Delivered'  # Cancelled orders are not count 
GROUP BY restaurant_name
ORDER BY 2 DESC, 3 DESC
LIMIT 3;  # limit to top 3

```
Results:

<img src="https://github.com/user-attachments/assets/fe7bbea1-fc45-4a84-91e3-e20cd0f83936" width="365" height="130">


2. What is the **customer churn rate**, and which groups are most affected?

SQL Code: 
```SQL
SELECT 
	age, 
    COUNT(*) AS total_users,
    COUNT(CASE WHEN churned = 'Active' THEN 1 END) AS total_inactive_users,
    ROUND(
		(COUNT(CASE WHEN churned = 'Active' THEN 1 END) / COUNT(*) ) *100, 2
        ) AS churn_rate
FROM foodpanda_review2
GROUP BY age
ORDER BY churn_rate DESC;
```
Results:

<img src="https://github.com/user-attachments/assets/ff67064d-ad7d-443e-81c2-0824bbf241d9" width="365" height="130">


3. What is the **city with highest frequency order** ?

SQL Code:
 
```SQL
SELECT 
	city,
    CASE      # Case statement to create segments of frequency of order
    WHEN order_frequency >=20 THEN 'High Frequency'  
    WHEN order_frequency <10 THEN 'Low Frequency'
    ELSE 'Medium Frequency'
    END AS frequency_segment,
    COUNT(*) AS number_users
FROM foodpanda_review2
GROUP BY city, frequency_segment
ORDER BY frequency_segment ASC, number_users DESC
LIMIT 5;
```
Results:

<img src="https://github.com/user-attachments/assets/0ac415ad-4b3b-4ed9-a5c7-ee3a8d1a32d2" width="365" height="130">





4.  What are the ranking of restaurants with the **highest % if delayed deliveries**?

SQL Code:
```SQL
SELECT 
	restaurant_name, 
    COUNT(*) AS total_customers,
	COUNT(CASE WHEN delivery_status = 'Delivered' THEN 1 END) AS total_delayed,  # if delayed, count it
    ROUND(
    (COUNT(CASE WHEN delivery_status = 'Delivered' THEN 1 END) / COUNT(*)) *100 )AS delayed_percentage  # calculate percentage
FROM foodpanda_review2
GROUP BY restaurant_name
ORDER BY delayed_percentage DESC;
```
Results:

<img src="https://github.com/user-attachments/assets/aaa4770f-b092-49a0-a0a2-150a25df3e73" width="365" height="130">


5. What are the **most popular dishes and cuisines** for each year?

SQL Code:
```SQL

SELECT 
	YEAR(order_date) AS order_year,
    dish_name,
    SUM(quantity) AS total_orders
FROM foodpanda_review2
GROUP BY YEAR(order_date), dish_name
ORDER BY order_year, total_orders DESC;    # this gives the ranking of popular dish, but we want to find top 1 for ever year
--- Most popular dishes
SELECT order_year, dish_name, total_orders
FROM (                                         # insert subqueries above
	SELECT 
		YEAR(order_date) AS order_year,
		dish_name,
		SUM(quantity) AS total_orders,
		ROW_NUMBER() OVER (
			PARTITION BY YEAR(order_date)  # filter per year
			ORDER BY SUM(quantity) DESC
        ) AS rn
	FROM foodpanda_review2
    GROUP BY YEAR(order_date), dish_name
) ranked
WHERE rn = 1
ORDER BY order_year;
--- Most popular cuisines
SELECT order_year, category AS cuisines, total_orders    # same method but, this time by cuisines
FROM (
	SELECT 
		YEAR(order_date) AS order_year,
		category,
		SUM(quantity) AS total_orders,
		ROW_NUMBER() OVER (
			PARTITION BY YEAR(order_date)
			ORDER BY SUM(quantity) DESC
        ) AS rn
	FROM foodpanda_review2
    GROUP BY YEAR(order_date), category
) ranked
WHERE rn = 1
ORDER BY order_year;

```
Results:

<img src="https://github.com/user-attachments/assets/4d1115e4-f62b-41fd-87fe-fbc64557bb28" width="320" height="110">


<img src="https://github.com/user-attachments/assets/18704b48-8fe1-45cc-999b-e05cf4096f91" width="320" height="110">


6. How do **monthly order trends** change over time?

SQL Code:
```SQL

SELECT 
	DATE_FORMAT(rating_date, '%Y-%m' ) AS month,  # just look at months
    AVG(rating) AS avg_rating
FROM foodpanda_review2
GROUP BY DATE_FORMAT(rating_date, '%Y-%m' )
ORDER BY month;
```
Results:

<img src="https://github.com/user-attachments/assets/ce72302e-a894-4e83-b452-f5a61cb91045" width="180" height="320">

## 🔑Key Insights
1. **Subway leads in business performance** — it records the highest revenue with strong average ratings, followed by Burger King and McDonald's.
2. **Customer satisfaction drives revenue** — average ratings show a positive correlation with total revenue.
3. **Retention challenges differ by age group** — teenagers represent the largest group of inactive users, while adults have the highest churn rate.
4. **City-level loyalty patterns** — Lahore customers demonstrate the strongest repeat purchase behavior, while Islamabad shows the lowest order frequency.
5. **Operational efficiency concerns** — KFC faces the highest number of delayed deliveries.
6. **Evolving preferences** — Fast Food re-emerged as the most popular cuisine in 2025.
7. **Customer experience stability** — overall ratings remain steady, ranging between 2.9 and 3.1.


