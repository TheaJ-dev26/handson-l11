# Hands On L11

## Approach
- Create S3 bucket, and create 2 folders in it: `raw` and `processed`. Download the Kaggle dataset and upload the `Amazon Sale Report.csv` into the `raw` folder.
- Create an IAM role, set the service to Glue, and give the following permissions: AmazonS3FullAccess, AWSGlueConsoleFullAccess, AWSGlueServiceRole.
- Create Crawler in AWS Glue. Add data source, browsing S3 for the `raw` folder, create a Glue database, and choose it as the target database. Then run the Crawler
- Open Athena console, browse S3 and select `processed` folder as the output location.

<b>S3 Buckets</b>
<img src="https://github.com/TheaJ-dev26/handson-l11/blob/main/images/S3_Buckets.png">

<b>IAM Roles</b>
<img src="https://github.com/TheaJ-dev26/handson-l11/blob/main/images/IAM_Role.png">

<b>Crawler Cloudwatch</b>
<img src="https://github.com/TheaJ-dev26/handson-l11/blob/main/images/Crawler_Cloudwatch.png">

<b>Query 1</b>'

```
SELECT * FROM handson-l11"."raw" limit 10;
```
<img src="https://github.com/TheaJ-dev26/handson-l11/blob/main/SQL%20Queries%20and%20Results/Query%201%20Results.png">


<b>Query 2 </b>

```
SELECT 
    "Category",
    COUNT(*) AS total_orders
FROM "handson-l11"."raw"
GROUP BY "Category"
LIMIT 10;
```

<img src="https://github.com/TheaJ-dev26/handson-l11/blob/main/SQL%20Queries%20and%20Results/Query%202%20Results.png">

<b>Query 3</b>

```
SELECT 
    "Fulfilment",
    COUNT(*) AS total_orders,
    SUM("Qty") AS total_units_sold,
    SUM("Amount") AS total_revenue
FROM "handson-l11"."raw"
WHERE "Status" NOT IN ('Cancelled', 'Pending')
GROUP BY "Fulfilment"
ORDER BY total_revenue DESC
LIMIT 10;
```

<img src="https://github.com/TheaJ-dev26/handson-l11/blob/main/SQL%20Queries%20and%20Results/Query%203%20Results.png">

<b>Query 4</b>

```
SELECT 
    date_trunc(
        'month',
        date_parse("Date", '%m-%d-%y')
    ) AS month,
    COUNT(*) AS total_orders,
    SUM(CAST("Amount" AS DOUBLE)) AS total_revenue
FROM "handson-l11"."raw"
WHERE "Status" NOT LIKE 'Cancelled%'
  AND "Status" NOT LIKE 'Pending%'
GROUP BY 1
ORDER BY 1 ASC
LIMIT 10;
```

<img src="https://github.com/TheaJ-dev26/handson-l11/blob/main/SQL%20Queries%20and%20Results/Query%204%20Results.png">

<b>Query 5</b>

```
WITH sku_sales AS (
    SELECT 
        "Category",
        "SKU",
        SUM("Amount") AS total_revenue,
        SUM("Qty") AS total_units_sold
    FROM "handson-l11"."raw"
    WHERE "Status" NOT IN ('Cancelled', 'Pending')
      AND "Qty" > 0
    GROUP BY "Category", "SKU"
),
ranked AS (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY "Category"
               ORDER BY total_revenue DESC
           ) AS rank
    FROM sku_sales
)
SELECT 
    "Category",
    "SKU",
    total_revenue,
    total_units_sold,
    rank
FROM ranked
WHERE rank <= 5
LIMIT 10;
```

<img src="https://github.com/TheaJ-dev26/handson-l11/blob/main/SQL%20Queries%20and%20Results/Query%205%20Results.png">