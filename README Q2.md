# marketyyticstest
sql codes


https://console.cloud.google.com/bigquery?sq=1089546178497:c00d3780af38474689ba5514c68afb1c

------------------------- BASIC STEPS: Setting up BigQuery ------------------------
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

-------------------------------- Replicating the Queries -----------------------------

The dataset I used has three to four tables and in the code file, all of them are pasted, you can also see them from here

https://console.cloud.google.com/bigquery?sq=1089546178497:c00d3780af38474689ba5514c68afb1c&project=reliable-strata-415314&ws=!1m4!1m3!8m2!1s1089546178497!2sc00d3780af38474689ba5514c68afb1c

1) CTE Retail
The Dataset has 8 columns(InvoiceNo,	StockCode,	Description,	Quantity,	InvoiceDate,	UnitPrice,	CustomerID,	Country). Their details point to the fact that it is about a retail store sales, across different countries to different customers who are buying various products in large stocks. 

Since we are going to access the main table continuesly, a cte is made and if u go to the comment line ,  (CTE made to make fetching data easier), you will be able to see it. Two additional things done in this cte is data cleaning,since its invoices, its not possible to create an invoice for a sku that is not tagged to a customer id or for its price to be 0, unit price will always be there even if after discount it becomes 0, similarly in the system its not possible to tag a product with quantity less then 1, hence removed all those invoices and the rest would be genuine invoices, so we removed rows where quantity, unit price was less then 0 or customer id was null. 
Second thing was extracting day and month to view patterns across day and month.

2) Overall
Next Step comes into calcualting some basic kpi that a retail store would have, for example, total amount of orders placed, total sales, total customers etc. 
The Overall part of query ( at the end) takes out these basic things, we have done, skus, orders, sales, item quantity, customers and repurchase rate ( which came out to be very high), other things that can be taken out after this is orders/sku, or sales/orders etc to find various ratios that would show the performance regardless of sales increasing or decreasing in various months etc.

3) TOP 10 SKU
After this we took out top 10 sku so that retail stores what is the highest in demand and makes sure that is always in stock

4) custperfor
If you scroll down to custperfor you would see the query that shows you which customers are buying the most in descending order.

 5) Country Performance
Last is Country Performance, which in overall as well is visible, but here the thought process is to show which countries have what kind of variations of prices and quantities in related to sku etc so that the team knows how much quantity should they expect to keep in their stocks etc.

Visualisation 
https://lookerstudio.google.com/u/0/reporting/cd2c76ef-fb99-4c79-a3e9-a7bdca2fce8f/page/tdJrD
To make them into visualisation, we made each of the above queries into tables, and converted that into charts and graphs using looker studio. 




