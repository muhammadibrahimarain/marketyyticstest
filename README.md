# marketyyticstest
sql codes


https://console.cloud.google.com/bigquery?sq=1089546178497:c00d3780af38474689ba5514c68afb1c

-------------------------
Setting Up GCP and BigQuery
Sign Up for GCP:

Go to the Google Cloud Platform website and sign up for an account. If you already have a Google account, you can use it to sign in.
Activate Your Free Trial:

Google offers a free trial with $300 credit to explore and use GCP services. This trial period is an excellent opportunity to try out BigQuery and Google Data Studio without incurring costs. Follow the prompts to activate your free trial.
Create a Project:

Once you're in the GCP console, create a new project. This project will be the container for all the resources and services you'll use, including BigQuery.
Navigate to BigQuery:

In the GCP console, go to the "BigQuery" service. You might need to enable the BigQuery API for your project if it's not already enabled.
Importing Your Data into BigQuery
Prepare Your Dataset:

Make sure your dataset from Google Sheets is clean and ready for analysis. Then, export it to a CSV file.
Create a Dataset in BigQuery:

In BigQuery, a "dataset" is a container for your tables. Create a new dataset within your project for your analysis.
Upload Your Data:

Within the dataset, create a new table and upload the CSV file you exported from Google Sheets. BigQuery will prompt you to configure the schema (column names and data types), which you can usually allow BigQuery to detect automatically based on the CSV.
Performing Descriptive Analysis Using SQL
Write and Run SQL Queries:

Use the BigQuery interface to write and execute SQL queries on your data. Start with basic queries to understand your data (e.g., SELECT statements to view the data, COUNT() to understand the size, AVG(), MIN(), MAX() for basic statistics).
Data Cleaning and Preparation:

Based on your initial exploration, you might need to clean your data (e.g., filtering out invalid entries, dealing with missing values). Write and execute SQL queries in BigQuery to clean and prepare your dataset.
Visualizing Data with Google Data Studio
Access Google Data Studio:

Go to Google Data Studio and sign in with your Google account.
Connect to BigQuery:

In Data Studio, create a new report and connect to BigQuery as your data source. Select the project, dataset, and table you want to visualize.
Create Visualizations:

Use Data Studio's interface to drag and drop different chart types onto your report and configure them to display your data. You can create various visualizations such as bar charts, line graphs, pie charts, and more to represent your analysis findings.
What Happens When the Free Trial Ends
After the free trial ends, if you've used up your $300 credit or the trial period is over (whichever comes first), you won't be automatically charged. Instead, you need to upgrade to a paid account to continue using GCP services.
Data and Resources: Unless you upgrade to a paid account, you may lose access to the resources and data you've created during the trial. Make sure to export or save your work if you decide not to continue with a paid account.
Always Free Tier: GCP offers an Always Free tier for some of their services, including BigQuery. For BigQuery, you get a certain amount of free usage each month, which might be sufficient for small projects. Check the current terms as they can change over time.
By following these steps, you should be able to leverage GCP for your data analysis task effectively. If you encounter specific questions or need further assistance with any of these steps, feel free to ask!
;



-------- Data cleaning, since its invoices, its not possible to create an invoice for a sku that is not tagged to a customer id or for its price to be 0, unit price will always be there even if after discount it becomes 0, similarly in the system its not possible to tag a product with quantity less then 1, hence removed all those invoices and the rest would be genuine invoices---

 with retail as
 ( 
  select distinct *,substr(cast(normal_date_time as string),1,7) as ym,substr(cast(normal_date_time as string),1,10) as ymd
  ,EXTRACT(DAY FROM normal_date_time) daily,EXTRACT(month FROM normal_date_time) monthly,EXTRACT(Year FROM normal_date_time) yearly
  from
  (
  select InvoiceNo,	StockCode,	Description,	Quantity,	InvoiceDate,	UnitPrice,	CustomerID,	Country
  ,TIMESTAMP_ADD(TIMESTAMP_ADD(TIMESTAMP '1899-12-30', INTERVAL CAST(InvoiceDate AS INT64) DAY), INTERVAL CAST((InvoiceDate - CAST(InvoiceDate AS INT64)) * 24 * 60 * 60 AS INT64) SECOND) AS normal_date_time

 from `my-project-for-test-415314.marketlytics_test_data.retail_data`
 where CAST(UnitPrice as float64) > CAST('0' as float64)
 and CAST(Quantity as float64) > CAST('0' as float64) 
 and CustomerID is not null
 )
 )

---- Country Stats
Select country, min(Quantity) min_quantity,max(Quantity) max_quantity, min(UnitPrice) min_price, max(unitprice) maxprice,STDDEV(Quantity) stdev_quantity,AVG(quantity) avg_quantity, STDDEV(UnitPrice) stdev_UnitPrice, avg(unitprice) avg_unitprice,sum(quantity) total_quantity, 
from retail
group by country
;

-- custperfor

select *
from (
select *
from (
select CustomerID,count(distinct invoiceno) orders, sum(sales) sales,min(items_sold) min_sold, max(items_sold) max_sold,min(sales) min_sales, max(sales) max_sales,max(repur_flag) repur_flag
from (
  select *, case when lead(CustomerID) OVER (PARTITION BY CustomerID ORDER BY ymd) is not null then 1 else 0 end as repur_flag
from (
Select country, InvoiceNo,	StockCode,CustomerID,sum(UnitPrice*quantity) as sales, sum(quantity) as items_sold,yearly,monthly,ymd
from retail
group by country, InvoiceNo,	StockCode,CustomerID,yearly,monthly,ymd
)
)
group by CustomerID
)
)
order by orders desc
;


-- ------- TOP 10 SKU --

select *
from (
select *, RANK() OVER (ORDER BY orders desc) AS rn
from (
select StockCode,description,count(distinct invoiceno) orders, sum(sales) sales
from (
Select country,description, InvoiceNo,	StockCode,CustomerID,sum(UnitPrice*quantity) as sales, sum(quantity) as items_sold,yearly,monthly,ymd
from retail
group by country,description, InvoiceNo,	StockCode,CustomerID,yearly,monthly,ymd
)
group by StockCode,description
)
)
where rn <= 10

;

-- Overall 
select *
from (
select country,yearly,monthly,count(distinct InvoiceNo) orders, sum(sku) as skus, count(distinct CustomerID) as customers, sum(sales) sales, sum(items_sold) items_sold,count(distinct next_purchase) as repurchase_customers
from
(
select country, InvoiceNo, count(distinct StockCode) as sku, CustomerID, sum(sales) as sales, sum(items_sold) as items_sold,yearly,monthly,next_purchase
from(
  select *, lead(CustomerID) OVER (PARTITION BY CustomerID ORDER BY ymd) next_purchase
from 
(
 select country, InvoiceNo,	StockCode,CustomerID,sum(UnitPrice*quantity) as sales, sum(quantity) as items_sold,yearly,monthly,ymd
 from retail
 group by country, InvoiceNo,	StockCode,CustomerID,yearly,monthly,ymd
)
)
group by country, InvoiceNo,CustomerID,yearly,monthly,next_purchase
)
group by country,yearly,monthly
)
where yearly is not null
and monthly is not null
;



-- ;
-- select *
-- from (
--  select substr(Description,1,3) as des_test, count(distinct StockCode) stocks
--  from `my-project-for-test-415314.marketlytics_test_data.retail_data`
--  where CAST(UnitPrice as float64) > CAST('0' as float64)
--  and CAST(Quantity as float64) > CAST('0' as float64) 
--  and CustomerID is not null
--  group by substr(Description,1,3),Quantity
--  ) where des_test is null;

--  select * 
--  from `my-project-for-test-415314.marketlytics_test_data.retail_data`
--  where Description is not null
--  and substr(Description,1,5) = 'AMAZO'
--  CAST(UnitPrice as float64) > CAST('0' as float64) 
--  group by substr(Description,1,5);
