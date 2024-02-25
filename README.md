# marketyyticstest
sql codes


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
