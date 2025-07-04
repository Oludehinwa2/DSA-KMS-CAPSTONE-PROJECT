                  # DSA-KMS-CAPSTONE-PROJEC
		  
         # 🏢 KMS Retail Business Analysis – SQL Case Study
	 
## 👩🏽‍💻 Project Title:
KMS Mega Stores Performance & Customer Insights (2009–2012)
Tools Used: SQL Server, Excel (for result presentation & dashboard)

## 📘 Overview
This project analyzes four years of transactional data for KMS Mega Stores to uncover insights into product performance, customer behavior, shipping efficiency, and regional sales patterns. It forms part of a Business Intelligence Capstone using SQL for deep querying and business reasoning.

As a Junior Data Analyst, I conducted extensive SQL-based analysis across two tables:

	•	kms_orders: Primary dataset containing sales, product, region, shipping, and customer details
 
	•	order_status: Secondary table tracking return statuses
 
## 🎯 Objectives
	•	Evaluate sales and profit performance across categories and regions
 
	•	Identify high- and low-performing customers and products
 
	•	Uncover shipping patterns and operational costs
 
	•	Analyze returns and customer segments
 
	•	Recommend strategies for increasing revenue and customer satisfaction
 
📌 Key Business Questions Answered (with SQL)

### Case Scenario I – Business Performance

	1.	Highest-selling product category
 
	2.	Top 3 and bottom 3 regions by sales
 
	3.	Appliance sales in Ontario
 
	4.	Recommendations for increasing revenue from bottom 10 customers
 
	5.	Shipping method with the highest cost
 
### Case Scenario II – Customer Insights

	6.	Top 5 valuable customers and what they typically buy
 
	7.	Best-performing Small Business customer
 
	8.	Most frequent Corporate customer (2009–2012)
 
	9.	Most profitable Consumer customer
 
	10.	Which customers returned products and their segment
 
	11.	Shipping cost estimation & average shipping days
 
	12.	Priority vs Shipping Mode efficiency
 
## 🛠 SQL Techniques Used

	•	GROUP BY, ORDER BY, JOIN, DISTINCT, WHERE, OFFSET-FETCH
 
	•	DATEDIFF() for shipping days
 
	•	SUM(), AVG(), ROUND() for aggregations
 
	•	Subqueries and filtered joins for customer behavior and returns
 
	•	Data cleanup using LTRIM(), RTRIM() for status fields

# Queries

[Uploading   SELECT name FROM sys.databases;
   USE KSM_CASE_STUDY_DB
   GO

   SELECT TABLE_NAME 
   FROM INFORMATION_SCHEMA. TABLES
   WHERE TABLE_TYPE='BASE TABLE';

   EXEC sp_rename '[KSM_CASE_STUDY_DB]', 'kms_orders';


   SELECT * FROM kms_orders


                           Case Scenario I � Business Performance

1. Which product category had the highest sales?

SELECT [Product_Category], SUM(Sales) AS Total_Sales
FROM [kms_orders]
GROUP BY [Product_Category]
ORDER BY Total_Sales DESC;

2. What are the Top 3 and Bottom 3 regions in terms of sales?

-- Top 3 Regions
SELECT Region, SUM(Sales) AS Total_Sales
FROM [kms_orders]
GROUP BY Region
ORDER BY Total_Sales DESC
OFFSET 0 ROWS FETCH NEXT 3 ROWS ONLY;

-- Bottom 3 Regions
SELECT Region, SUM(Sales) AS Total_Sales
FROM [kms_orders]
GROUP BY Region
ORDER BY Total_Sales ASC
OFFSET 0 ROWS FETCH NEXT 3 ROWS ONLY;

3. What were the total sales of Appliances in Ontario?

-- Check distinct Product Categories
SELECT DISTINCT [Product_Category] FROM [kms_orders]

-- Check distinct Province values
SELECT DISTINCT Province FROM [kms_orders]

                       Now
  SELECT Region, SUM(Sales) AS [Total Sales of Appliance]
FROM [kms_orders]
WHERE Province = 'Ontario'
  AND Product_Sub_Category = 'Appliances'
GROUP BY Region;

           

4. What can KMS do to increase revenue from the bottom 10 customers?

SELECT [Customer_Name], SUM(Sales) AS Total_Sales
FROM [kms_orders]
GROUP BY [Customer_Name]
ORDER BY Total_Sales ASC
OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY;

5. KMS incurred the most shipping cost using which shipping method?

SELECT [Ship_Mode], SUM([Shipping_Cost]) AS Total_Shipping_Cost
FROM [kms_orders]
GROUP BY [Ship_Mode]
ORDER BY Total_Shipping_Cost DESC;

                
				             Case Scenario II � Customer Insights

6. Who are the most valuable customers, and what do they buy?

SELECT [Customer_Name], SUM(Sales) AS Total_Spent
FROM [kms_orders]
GROUP BY [Customer_Name]
ORDER BY Total_Spent DESC
OFFSET 0 ROWS FETCH NEXT 5 ROWS ONLY;

          what they typically buy:

SELECT [Customer_Name], [Product_Category], COUNT(*) AS Order_Count
FROM [kms_orders]
WHERE [Customer_Name] IN (
    SELECT TOP 5 [Customer_Name]
    FROM [kms_orders]
    GROUP BY [Customer_Name]
    ORDER BY SUM(Sales) DESC
)
GROUP BY [Customer_Name], [Product_Category]
ORDER BY [Customer_Name];

7. Which Small Business customer had the highest sales?

SELECT [Customer_Name], SUM(Sales) AS Total_Sales
FROM [kms_orders]
WHERE [Customer_Segment] = 'Small Business'
GROUP BY [Customer_Name]
ORDER BY Total_Sales DESC
OFFSET 0 ROWS FETCH NEXT 1 ROWS ONLY;

8. Which Corporate customer placed the most orders (2009�2012)?

SELECT [Customer_Name], COUNT([Order_ID]) AS Order_Count
FROM [kms_orders]
WHERE [Customer_Segment] = 'Corporate'
GROUP BY [Customer_Name]
ORDER BY Order_Count DESC
OFFSET 0 ROWS FETCH NEXT 1 ROWS ONLY;
             OR
  SELECT TOP 1
    Customer_Name,
    COUNT(DISTINCT Order_ID) AS Total_Orders
FROM
    [kms_orders]
WHERE
    Customer_Segment = 'Corporate'
    AND Order_Date BETWEEN '2009-01-01' AND '2012-12-31'
GROUP BY
    Customer_Name
ORDER BY
    Total_Orders DESC;

9. Which Consumer customer was the most profitable?

SELECT [Customer_Name], SUM(Profit) AS Total_Profit
FROM [kms_orders]
WHERE [Customer_Segment] = 'Consumer'
GROUP BY [Customer_Name]
ORDER BY Total_Profit DESC
OFFSET 0 ROWS FETCH NEXT 1 ROWS ONLY;


10.Which customers returned items, and what segment do they belong to?

        SELECT*FROM Order_Status

SELECT DISTINCT k.[Customer_Name], k.[Customer_Segment]
FROM kms_orders k
JOIN order_status o
  ON k.[Order_ID] = o.[Order_ID]
WHERE o.Status = 'Returned';
       OR
SELECT DISTINCT Customer_Name, Customer_Segment
FROM kms_orders k, Order_Status
WHERE Status = 'Returned'
      
            SIR ANOTHER BUEATIFUL WAY(STILL NO.10)
      SELECT DISTINCT 
    k.[Customer_Name], 
    k.[Customer_Segment]
FROM 
    kms_orders k
JOIN 
    order_status o
    ON k.[Order_ID] = o.[Order_ID]
WHERE 
    LTRIM(RTRIM(o.Status)) = 'Returned';
 SELECT COUNT(*) AS Returned_Order_Lines
FROM 
    kms_orders k
JOIN 
    order_status o
    ON k.[Order_ID] = o.[Order_ID]
WHERE 
    LTRIM(RTRIM(o.Status)) = 'Returned';
	SELECT 
    k.[Order_ID],
    k.[Order_Date],
    k.[Ship_Date],
    k.[Customer_Name],
    k.[Customer_Segment],
    k.[Province],
    k.[Region],
    k.[Product_Category],
    k.[Product_Sub_Category],
    k.[Product_Name],
    k.[Sales],
    k.[Profit],
    k.[Shipping_Cost],
    o.Status AS Return_Status
FROM 
    kms_orders k
JOIN 
    order_status o
    ON k.[Order_ID] = o.[Order_ID]
WHERE 
    LTRIM(RTRIM(o.Status)) = 'Returned'
ORDER BY 
    k.[Customer_Name], k.[Order_Date];
     


11a. Did the company ship based on Order Priority efficiently?

SELECT 
    [Order_Priority],
    [Ship_Mode],
    COUNT(*) AS Order_Count,
    SUM([Shipping_Cost]) AS Total_Shipping
FROM [kms_orders]
GROUP BY [Order_Priority], [Ship_Mode]
ORDER BY [Order_Priority], [Ship_Mode];

11b. What is the estimated total shipping cost and average shipping days?

SELECT 
    ROUND(SUM(Sales - Profit), 2) AS Estimated_Shipping_Cost,
    AVG(DATEDIFF(DAY, [Order_Date], [Ship_Date])) AS Avg_Ship_Days
FROM [kms_orders];


OR

select Ship_Mode,
sum (Shipping_Cost) as Total_Shipping from [kms_orders]
group by Ship_Mode
order by Total_Shipping desc

select Order_Priority,
        Ship_Mode,
count (*) as Total_Orders,
avg(Shipping_Cost) as [Average Shippping Cost]
from [kms_orders]
group by Order_Priority,
         Ship_Mode
order by Order_Priority,
         Total_Orders,
          [Average Shippping Cost]desc

select Ship_Mode,
      Order_Priority,
avg(datediff(day,[Order_Date],[Ship_Date])) as Avg_Ship_Days from [kms_orders]
group by Order_Priority,
         Ship_Mode
Order by Avg_Ship_Days desc,
         Ship_Mode,
		 Order_Priority




























































































































































































SQLQuery4.sql…]()

 
 
## 📊 Sample Insights
	•	📈 Technology was the highest revenue-generating category
 
	•	🌍 West and Ontario were the top-performing regions
 
	•	📦 Standard Class incurred the highest shipping costs
 
	•	🤝 The Consumer segment returned the most items
 
	•	⏱️ Average shipping time: 4–5 days
 
	•	🧾 Estimated shipping cost derived from Sales - Profit

## 💡 Business Recommendations

	•	Focus marketing on top regions and product categories
 
	•	Incentivize high-value customers with loyalty programs
 
	•	Optimize shipping for high-cost modes like Standard Class
 
	•	Follow up on returns to reduce churn
 
	•	Use product bundling or targeted promotions in low-performing regions
 
  •	Improve logistics for ‘High Priority’ orders to align shipping mode with urgency
  
  •	Use customer segmentation to tailor upselling strategies for underperforming clients.
  
📁 Folder Structure (Recommended)
📂 KMS-SQL-Case-Study/
├── 📄 README.md
├── 📁 SQL Queries/
│   ├── business_performance_queries.sql
│   ├── customer_insights_queries.sql
├── 📁 Results/
│   ├── Top_Customers_Report.xlsx
│   ├── Region_Sales_Table.xlsx
│   ├── Shipping_Cost_Analysis.xlsx
└── 📁 Dashboard/
    └── KMS_Performance_Summary.xlsx
## 🧾 Conclusion

The KMS SQL case study provided a comprehensive view of the business’s performance between 2009 and 2012. By leveraging structured query techniques, we extracted actionable insights across multiple dimensions — including customer behavior, sales trends, product profitability, and shipping efficiency.

Through this analysis, it became clear that regional focus, customer segmentation, and shipping strategy are critical levers for improving overall business performance. We also identified key customer groups, high-return segments, and opportunities to optimize costs, especially around logistics.

This project not only demonstrates my ability to apply SQL in a real-world business context, but also shows how data can guide strategic decisions for e-commerce or retail platforms. Going forward, these insights could serve as a strong foundation for advanced dashboarding (Power BI, Excel), reporting automation, or further customer lifetime value (CLV) and return rate analysis
