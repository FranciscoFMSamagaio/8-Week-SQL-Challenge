

## Question and Solution

#### 1. What is the total amount each customer spent at the restaurant?

```
SELECT DISTINCT 
  	sales.customer_id, 
    SUM(menu.price)
FROM dannys_diner.sales
LEFT JOIN dannys_diner.menu
    on menu.product_id = sales.product_id
GROUP BY 
    sales.customer_id
```
##### Answer:

| customer_id | sum |
| ----------- | --- |
| C           | 36  |
| B           | 74  |
| A           | 76  |

---


#### 2. How many days has each customer visited the restaurant?

```
SELECT  
  	sales.customer_id, COUNT(DISTINCT sales.order_date)
FROM dannys_diner.sales
GROUP BY 
    sales.customer_id
```
##### Answer:

| customer_id | count |
| ----------- | ----- |
| A           | 4     |
| B           | 6     |
| C           | 2     |

---

#### 3. What was the first item from the menu purchased by each customer?

```
WITH cte AS
(
 	SELECT DISTINCT 
      sales.customer_id, 
      menu.product_name,
      DENSE_RANK() OVER (PARTITION BY sales.customer_id
                         ORDER BY sales.order_date) as rank

  FROM dannys_diner.sales
  LEFT JOIN dannys_diner.menu
      on menu.product_id = sales.product_id
  order by rank
)
SELECT 
	cte.customer_id, 
	cte.product_name
FROM cte AS cte 
where rank = 1
```

##### Answer:

| customer_id | product_name |
| ----------- | ------------ |
| A           | curry        |
| A           | sushi        |
| B           | curry        |
| C           | ramen        |

---

#### 4. What is the most purchased item on the menu and how many times was it purchased by all customers?

```
SELECT  
  	COUNT(sales.product_id), menu.product_name
FROM dannys_diner.sales AS sales
LEFT JOIN dannys_diner.menu
    on menu.product_id = sales.product_id
GROUP BY 
	menu.product_name
ORDER BY 
	COUNT(sales.product_id) DESC
LIMIT 1
```

##### Answer:

| count | product_name |
| ----- | ------------ |
| 8     | ramen        |

---

#### 5. Which item was the most popular for each customer?

```
WITH cte AS 
(
  SELECT DISTINCT 
  	sales.customer_id, 
  	menu.product_name,
  	DENSE_RANK() OVER (PARTITION BY sales.customer_id
                      ORDER BY COUNT(menu.product_name)) as rank

  FROM dannys_diner.sales
  LEFT JOIN dannys_diner.menu
  	on menu.product_id = sales.product_id
  GROUP BY
  	sales.customer_id, menu.product_name
  order by rank
  )
  
  SELECT 
 	cte.customer_id, 
  	cte.product_name
  FROM cte as cte
  WHERE 1=1
```

##### Answer:

| customer_id | product_name |
| ----------- | ------------ |
| A           | sushi        |
| B           | curry        |
| B           | ramen        |
| B           | sushi        |
| C           | ramen        |

---

#### 6. Which item was purchased first by the customer after they became a member?

```
WITH cte AS (
  SELECT 
 	members.customer_id, 
  	members.join_date,
    sales.product_id, 
    rank() OVER (PARTITION BY members.customer_id ORDER BY sales.order_date)
  FROM dannys_diner.members as members
  LEFT JOIN dannys_diner.sales as sales
  	ON sales.customer_id = members.customer_id
  WHERE 1=1
	AND sales.order_date >= members.join_date
)
SELECT 
 	cte.customer_id, 
    menu.product_name
FROM cte as cte
LEFT JOIN dannys_diner.menu as menu 
	ON menu.product_id = cte.product_id
WHERE rank = 1
```

##### Answer:

| customer_id | product_name |
| ----------- | ------------ |
| B           | sushi        |
| A           | curry        |

---

#### 7. Which item was purchased just before the customer became a member?

```
WITH cte AS (
  SELECT 
 	members.customer_id, 
  	members.join_date,
    sales.product_id, 
    row_number() OVER (PARTITION BY members.customer_id ORDER BY sales.order_date desc) as rank
  FROM dannys_diner.members as members
  LEFT JOIN dannys_diner.sales as sales
  	ON sales.customer_id = members.customer_id
  WHERE 1=1
	AND sales.order_date < members.join_date
)
SELECT 
 	cte.customer_id, 
    menu.product_name
FROM cte as cte
LEFT JOIN dannys_diner.menu as menu 
	ON menu.product_id = cte.product_id
WHERE rank = 1
```

##### Answer:

| customer_id | product_name |
| ----------- | ------------ |
| B           | sushi        |
| A           | sushi        |

---

#### 8. What is the total items and amount spent for each member before they became a member?


```
SELECT 
  sales.customer_id, 
  count( menu.product_name), 
  sum(menu.price)

  FROM dannys_diner.sales as sales
  LEFT JOIN dannys_diner.members as members
    ON members.customer_id = sales.customer_id
  LEFT JOIN dannys_diner.menu as menu
    ON menu.product_id = sales.product_id
  WHERE 1=1
  AND sales.order_date < members.join_date
  GROUP BY 
  sales.customer_id

```

##### Answer:

| customer_id | count | sum |
| ----------- | ----- | --- |
| B           | 3     | 40  |
| A           | 2     | 25  |

---

#### 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier — how many points would each customer have?

```
SELECT 
  sales.customer_id, 
    SUM(
      CASE 
        WHEN sales.product_id = 1 THEN 20 * menu.price
        ELSE 10 * menu.price
      END 
      ) AS Points
      
  FROM dannys_diner.sales as sales
  LEFT JOIN dannys_diner.members as members
    ON members.customer_id = sales.customer_id
  LEFT JOIN dannys_diner.menu as menu
    ON menu.product_id = sales.product_id
  WHERE 1=1

  GROUP BY 
  sales.customer_id
```

| customer_id | points |
| ----------- | ------ |
| B           | 940    |
| C           | 360    |
| A           | 860    |

---

#### 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi — how many points do customer A and B have at the end of January?
```
    WITH dates AS 
    (
      SELECT 
     	members.customer_id, 
    	members.join_date as first_day,
        members.join_date + 6 as last_day
         
      FROM dannys_diner.members as members
    )
    
    SELECT 
      sales.customer_id, 
        SUM(
          CASE 
            WHEN sales.product_id = 1 THEN 20 * menu.price
          	WHEN sales.order_date BETWEEN first_day AND last_day THEN 20 * menu.price
            ELSE 10 * menu.price
          END 
          ) AS Points
          
      FROM dannys_diner.sales as sales
      LEFT JOIN dannys_diner.members as members
        ON members.customer_id = sales.customer_id
      LEFT JOIN dannys_diner.menu as menu
        ON menu.product_id = sales.product_id
      LEFT JOIN dates as dates 
      	ON dates.customer_id = sales.customer_id
      
      WHERE 1=1
      	AND sales.customer_id IN ('A', 'B')
        AND sales.order_date < '2021-02-01'
    
      GROUP BY 
      sales.customer_id
```

| customer_id | points |
| ----------- | ------ |
| A           | 1370   |
| B           | 820    |

---

## BONUS QUESTIONS

#### Join All The Things

#### Recreate the table with: customer_id, order_date, product_name, price, member (Y/N)

SELECT 
	sales.customer_id, 
	sales.order_date,
    menu.product_name,
    menu.price,
    CASE
    	WHEN members.join_date <= sales.order_date THEN 'Y'
        ELSE 'N'
    END AS member
      
  FROM dannys_diner.sales as sales
  LEFT JOIN dannys_diner.members as members
    ON members.customer_id = sales.customer_id
  LEFT JOIN dannys_diner.menu as menu
    ON menu.product_id = sales.product_id
  
  WHERE 1=1

ORDER BY
	sales.customer_id, 
	sales.order_date




| customer_id | order_date | product_name | price | member |
| ----------- | ---------- | ------------ | ----- | ------ |
| A           | 2021-01-01 | sushi        | 10    | N      |
| A           | 2021-01-01 | curry        | 15    | N      |
| A           | 2021-01-07 | curry        | 15    | Y      |
| A           | 2021-01-10 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| A           | 2021-01-11 | ramen        | 12    | Y      |
| B           | 2021-01-01 | curry        | 15    | N      |
| B           | 2021-01-02 | curry        | 15    | N      |
| B           | 2021-01-04 | sushi        | 10    | N      |
| B           | 2021-01-11 | sushi        | 10    | Y      |
| B           | 2021-01-16 | ramen        | 12    | Y      |
| B           | 2021-02-01 | ramen        | 12    | Y      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-01 | ramen        | 12    | N      |
| C           | 2021-01-07 | ramen        | 12    | N      |

---

#### Rank All The Things

#### Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.


```
    WITH a AS(
    SELECT 
    	sales.customer_id, 
    	sales.order_date,
        menu.product_name,
        menu.price,
        CASE
        	WHEN members.join_date <= sales.order_date THEN 'Y'
            ELSE 'N'
        END AS member
          
      FROM dannys_diner.sales as sales
      LEFT JOIN dannys_diner.members as members
        ON members.customer_id = sales.customer_id
      LEFT JOIN dannys_diner.menu as menu
        ON menu.product_id = sales.product_id
      
      WHERE 1=1
    
    ORDER BY
    	sales.customer_id, 
    	sales.order_date
    )
    select 
    	*,
      CASE
        WHEN member = 'N' then NULL
        ELSE RANK () OVER (
          PARTITION BY customer_id, member
          ORDER BY order_date
      ) END AS ranking
    FROM a
    ORDER BY
    	a.customer_id, 
    	a.order_date

``` 
| customer_id | order_date | product_name | price | member | ranking |
| ----------- | ---------- | ------------ | ----- | ------ | ------- |
| A           | 2021-01-01 | sushi        | 10    | N      |         |
| A           | 2021-01-01 | curry        | 15    | N      |         |
| A           | 2021-01-07 | curry        | 15    | Y      | 1       |
| A           | 2021-01-10 | ramen        | 12    | Y      | 2       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| A           | 2021-01-11 | ramen        | 12    | Y      | 3       |
| B           | 2021-01-01 | curry        | 15    | N      |         |
| B           | 2021-01-02 | curry        | 15    | N      |         |
| B           | 2021-01-04 | sushi        | 10    | N      |         |
| B           | 2021-01-11 | sushi        | 10    | Y      | 1       |
| B           | 2021-01-16 | ramen        | 12    | Y      | 2       |
| B           | 2021-02-01 | ramen        | 12    | Y      | 3       |
| C           | 2021-01-01 | ramen        | 12    | N      |         |
| C           | 2021-01-01 | ramen        | 12    | N      |         |
| C           | 2021-01-07 | ramen        | 12    | N      |         |

---
