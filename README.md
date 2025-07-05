
# E-commerce Revenue Optimization Dashboard

 ![image](https://github.com/user-attachments/assets/14ad64d7-4cf4-4692-9823-db91c7b7a730)

## Problem Statement

The Planning Department set a revenue target of ₹1 crore to be achieved by the end of the day. However, by the morning analysis, only 67% of the target had been reached. The challenge was to develop a strategy to close this gap by optimizing pricing, driving more traffic, and improving conversion rates.


### Steps followed 

- Step 1 : Load Data into SQL: The initial step involved importing the CSV dataset into SQL, preparing it for structured analysis and enabling efficient data handling.
- Step 2 : Create Database: A dedicated database was created to store Sales data, ensuring organized storage for streamlined querying and analysis.
- Step 3 : SQL Analysis: Using SQL, specific queries were designed as per the problem statement to extract required KPIs and insights. The output was stored and documented, establishing a clear basis for metrics and trends.
- Step 4 : Connect SQL to Power BI: The final step was to connect Power BI to the SQL database, allowing the visual dashboard to be directly compared with SQL output for accuracy. This integration enabled real-time data visualization, showcasing insights through interactive graphs and charts in Power BI.

### SQL analysis 

KPI's:

    1. Monitor Revenue Performance: Query sales by hour, product category, and region to identify gaps.

    select extract(hour from order_datetime) as hr,
    s.category, c.region, sum(total_revenue) as sales, count(order_id) as total_orders
    from sales s
    inner join customer c on c.customer_id = s.customer_id
    group by extract(hour from order_datetime),s.category, c.region
    order by hr, sales desc;

 ![image](https://github.com/user-attachments/assets/8724a3d0-fa36-4c08-a007-88006ac0a4c4)


    2. Identify Top and Bottom Performers: Find best-selling and slow-moving products.
    a. top 10 best selling products

    select product_id, category, sum(units_sold) as total_unit_sold, sum(total_revenue) as total_sales  
    from sales
    group by product_id, category
    order by total_sales desc
    limit 10;

   
![image](https://github.com/user-attachments/assets/a13da0b2-723d-40b7-9f24-d014ec82d15e)

    b. slow-moving products - slow-moving inventory involves identifying products that are not selling as expected and remain in stock for a longer period. 

    select i.product_id, i.category, i.stock_available, 
    coalesce(sum(s.units_sold),0) as total_unit_sold, i.days_in_inventory 
    from inventory i
    left join sales s on s.product_id = i.product_id
    group by i.product_id, i.category, i.stock_available, i.days_in_inventory 
    having coalesce(sum(s.units_sold),0) <10
    order by i.days_in_inventory desc;

![image](https://github.com/user-attachments/assets/8e5d7906-cd60-4741-a6cd-12a2411a288b)

    3. Analyze Conversion Trends: To analyze conversion trends, the focus is on understanding how effectively visitors are turning into buyers and how this affects sale
       
    select t.hour, t.traffic_source, t.visitors, t.conversion_rate
    ,sum(s.total_revenue) as total_revenue
    ,ROUND((sum(s.total_revenue)/t.visitors),2) as sales_per_visitor
    from traffic t
    left join sales s on extract(hour from s.order_datetime) = t.hour
    group by t.hour, t.traffic_source, t.visitors, t.conversion_rate
    order by t.hour;

![image](https://github.com/user-attachments/assets/0cbe8e0d-b814-4fcd-9493-0af96271a1e6)


    4. Forecast End-of-Day Revenue: Use historical patterns to predict total revenue based on current performance and Predict EOD revenue using current cumulative revenue and remaining hours.

    Step 1: Calculate historical hourly contribution
	Step 2: Calculate cumulative revenue till today
	Step 3: Forecast end-of-day revenue

    with hourly_performance as
        (select extract(hour from order_datetime) as hour
            ,sum(total_revenue) as hourly_revenue
            ,sum(sum(total_revenue)) over() as Total_Day_Revenue
            ,Round((sum(total_revenue)/sum(sum(total_revenue)) over()) *100,2) as hourly_percentage
            from sales
            where order_datetime < '2024-11-21'
            group by hour
            order by hour),
    today_revenue as
        (select sum(total_revenue) as current_revenue
            ,extract(hour from max(order_datetime)) as final_hour
            from sales
            where date(order_datetime) = '2024-11-21')
    select t.current_revenue
    , round((t.current_revenue/hp.hourly_percentage) *100,2) as predicted_eod_revenue
    from today_revenue t
    join hourly_performance hp on hp.hour = t.final_hour;

![image](https://github.com/user-attachments/assets/97c9f6b2-7bc4-43d0-88ef-fae563dbb0ad)

### Insights from the Dashboard:

1. Revenue Performance:

▪️ Current sales stand at ₹6.7M, reaching 67% of the ₹10M target.

▪️ Peak revenue times are observed around 9 AM and 8 PM, highlighting optimal traffic and conversion periods.



2. Category Performance:

▪️ Home Decor leads with the highest revenue of ₹18.23M and 670 units sold.

▪️ Clothing has substantial inventory (8245 units) but shows moderate revenue performance.

▪️ Electronics and Accessories require increased promotional efforts due to their lower revenue per unit sold.



3. Traffic Sources:

▪️ Direct and Social traffic drive the highest number of visits, contributing 2243.6 and 2187.9 units respectively.

▪️ Paid traffic is underperforming and needs optimization to improve conversion rates.



4. Product Performance:

▪️ Top Slow-Moving Products: Clothing items dominate the slow-moving category, with stock levels exceeding demand.

▪️ Days in Inventory: Products like PROD045 have been in stock for over 190 days, signaling a need for discounts or promotional campaigns.



5. Customer Loyalty:

▪️ The Bronze and Platinum segments are the dominant customer groups. Tailored engagement strategies for these segments could boost conversion rates significantly.



Actionable Recommendations:

▪️ Promotions: Focus on Clothing and slow-moving inventory to reduce excess stock and increase sales.

▪️ Paid Traffic: Invest in optimizing Paid Traffic channels to improve conversion rates.

▪️ Peak Hours: Align marketing campaigns with peak revenue hours to maximize conversions.

▪️ Loyalty Campaigns: Launch targeted loyalty campaigns for Bronze and Platinum customers to improve retention and increase sales.





