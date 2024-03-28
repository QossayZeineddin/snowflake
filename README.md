# snowflake simple sql querys 

### the database schema is 

LOCATION_ID	|   DATE_TIME                |	PRODUCT_NAME                | QUANTITY | PRICE  \
3766365	    |  2024-02-01 20:00:00.000	 | Blueberry Chiffon Cake Slice | 1        | 6.590 \
3766365     |  2024-02-01 20:00:00.000	 | Cheese Tart	                | 1	       | 3.490 \
6561974	    |  2024-02-01 18:00:00.000	 | 10 Boneless Wings	        | 1        | 13.990 \

## 1 - Top 10, product soled per location id 


    WITH RankedProducts AS (
      SELECT
        location_id,
        product_name,
        AVG(quantity) AS avg_quantity,
        RANK() OVER(PARTITION BY location_id ORDER BY AVG(quantity) DESC) AS rank_desc,
        RANK() OVER(PARTITION BY location_id ORDER BY AVG(quantity)) AS rank_asc
      FROM product
      GROUP BY location_id, product_name
    )
    SELECT *
    FROM RankedProducts
    WHERE rank_asc <= 3 
    ORDER BY location_id, rank_desc;
    
sample snapshot  

  <img width="1106" alt="Screenshot 2024-03-28 at 10 30 52 AM" src="https://github.com/QossayZeineddin/snowflake/assets/103140839/436ffb72-cef9-44ff-bf36-38267b18c7b7">



### lowest 3, product soled per location id 


    WITH RankedProducts AS (
      SELECT
        location_id,
        product_name,
        AVG(quantity) AS avg_quantity,
        RANK() OVER(PARTITION BY location_id ORDER BY AVG(quantity) DESC) AS rank_desc,
        RANK() OVER(PARTITION BY location_id ORDER BY AVG(quantity)) AS rank_asc
      FROM product
      GROUP BY location_id, product_name
    )
    
    SELECT *
    FROM RankedProducts
    WHERE rank_desc <= 10 
    ORDER BY location_id, rank_desc;
    
sample snapshot

  <img width="1111" alt="Screenshot 2024-03-28 at 10 29 44 AM" src="https://github.com/QossayZeineddin/snowflake/assets/103140839/cd55c71e-94e8-4154-9e39-9a5dabcc60ef">



## 2 Best Location sales


    SELECT
      location_id,
      SUM(quantity * price) AS total_sales
    FROM product
    GROUP BY location_id
    ORDER BY total_sales DESC
    LIMIT 1;
    
  sample snapshot

  <img width="1117" alt="Screenshot 2024-03-28 at 10 32 59 AM" src="https://github.com/QossayZeineddin/snowflake/assets/103140839/d49ab2e3-4d41-4070-8c37-5b0e3521eb67">



## 3 best time for sale per location


    WITH SalesRank AS (
      SELECT
        location_id,
        date_time,
        SUM(quantity * price) AS total_sales,
        RANK() OVER(PARTITION BY location_id ORDER BY SUM(quantity * price) DESC) AS sales_rank
      FROM product
      GROUP BY location_id, date_time
    )
    SELECT
      location_id,
      date_time,
      total_sales,
      sales_rank
    FROM SalesRank
    WHERE sales_rank = 1;

  sample snapshot

 <img width="1115" alt="Screenshot 2024-03-28 at 10 34 58 AM" src="https://github.com/QossayZeineddin/snowflake/assets/103140839/14dd5eaf-5208-44d8-962b-1f35653e5c80">



## 4 quantity per day per location



    select
      location_id,
      DATE(date_time) as sale_date,
      SUM(quantity) as total_quantity
    from product
    group by location_id, DATE(date_time)
    order by location_id, sale_date;

  sample snapshot
  
   <img width="1111" alt="Screenshot 2024-03-28 at 10 36 48 AM" src="https://github.com/QossayZeineddin/snowflake/assets/103140839/1beb113c-39ef-425a-81f9-21539b4d9474">



## 5 Month to Date sales per location

    
    WITH DailySales AS (
      SELECT
        location_id,
        DATE(date_time) AS sale_date,
        SUM(quantity * price) AS daily_sales
      FROM product
      //WHERE date_time BETWEEN DATE_TRUNC('MONTH', CURRENT_DATE()) AND CURRENT_DATE()
      GROUP BY location_id, DATE(date_time)
    )
    SELECT
      location_id,
      sale_date,
      daily_sales,
      SUM(daily_sales) OVER (
        PARTITION BY location_id, EXTRACT(YEAR FROM sale_date), EXTRACT(MONTH FROM sale_date)
        ORDER BY sale_date
      ) AS mtd_sales
    FROM DailySales
    ORDER BY location_id, sale_date;

  sample snapshot

 <img width="1112" alt="Screenshot 2024-03-28 at 10 38 17 AM" src="https://github.com/QossayZeineddin/snowflake/assets/103140839/0f25a86d-e791-47b3-89d1-55e29e058b20">

