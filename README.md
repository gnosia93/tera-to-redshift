# tera-to-redshift

This document describe DW migration from teradata to amazon redshift. Altough this article is written based on hypothetical scenario but expect verty helpful for real world migration. 
Tasks are consist of folllowing two categories.

- _Develope Redshift Migration Strategy_
- _Modernize ETL with AWS Glue_ 


### Customer Context ###

- Built EDW on top of Teradata appliance at onpremise.
- Data is ingested using ETL from department stores and online mall.
- Analyze their customer and marketing data and wanted to reduce TCO.
- Considering Predictive analytics in addtion to Descriptive analytics(BI).

Descriptive analytics: A traditional business intelligence method that uses visualizations like pie charts, bar charts, tables, or line graphs. It's the foundation for other types of analytics. For example, you can use descriptive analytics to determine how many sales occurred in a quarter and if they increased or decreased. 
Predictive analytics: Uses historical data to predict likely outcomes and make educated forecasts. It's a more complex type of analytics that uses probabilities instead of simply interpreting existing facts. 

### Proposal ###
proposed Redshift based DW with following advantage   

- Amazon Redshift is a fully managed, petabyte-scale data warehouse.
- Has similiar achitecture with teradta (MPP, postgres based), but much more cheaper.
- Columnar based storage, which is more appropriate for DW.
- Integration with S3 using redshift spectrum, which offload table data into S3.    
- Seamlessly integrated with AWS AI/ML and analytics services.


### POC ###

Check the feasiblity of redshift migration and provide optimal migration strategy.

- Item #1 - Data Type and SQL Conversion _(with minial manual efforts)._
- Item #2 - Find optimal data migration architecture _(to minimize DW downtime)._
- Item #3 - Provide Monitoring and Performance Tuning Practice for Post Migration.
- Item #4 - Provide Cloud ETL with Serverless Architecture _(for SCD)_.   

_In data warehousing, "SCD" stands for "Slowly Changing Dimension," which refers to a dimension table that stores data which changes gradually over time, allowing you to track historical changes in attributes like customer addresses or product details while maintaining accurate reporting across different time periods_
  

#### Source Volumes & Chracteristics ####

- 2TB storage volumns with about 70 tables.
- The numbers of AMPs is 4.
- dimension table has insert and update.
- fact table only has insert.
- a bunch of summary table exists (only insert) : daily, monthly.

@@@ sales_fact @@@  
To develope migration strategy, picked sales fact table which has a large amount of data volume.   
_Below mutli dimensional model is not real diagram and just shown in order to improve your understandings._

![](https://github.com/gnosia93/tera-to-redshift/blob/main/fact-design.png)

- Tuple size : 624 bytes
- Avg Daily Insert : 82 만건 (daily 375 MB)
  - 10.99 gb / month, 131.96 gb / year ---> 659.82 gb / 5 years 
- Estimated migration time for daily data. 
  - export time : 20 sec
  - s3 upload time : 10 sec
  - redshift copy time : 14 sec
  - total elapsed time : 44 sec
- estimated loading time for 5 years : 30.41 Hours


@@@ product_dim @@@  
_https://www.tutorials24x7.com/mysql/guide-to-design-database-for-shopping-cart-in-mysql_     
Among Dimensions, product table is the largest one. we will use this table to calulate dimension table's migration time. 
In case of dimension, Both Insert and Update is happened, so we will migrate it at once with parallel processing

![](https://github.com/gnosia93/tera-to-redshift/blob/main/images/product-dim.png)

- Tuple size : 1800 bytes. 
- Total record count : 40M records 
- AMP number : 4
- Number of CSV file : 4
- Gizip Compressed : No
- Estimated migration time per file (10M records, 18.87 GB)
  - export time : 714 sec (around 12 minutes)
  - s3 upload time :  301 sec (acround 5 minutes)
  - redshift copy time : 
  - total elapsed time : 



```
A typical "sales fact" record in a data warehouse, depending on the level of detail captured,
usually averages around 2 kilobytes (KB) in size, with some variations depending
on the specific data fields included and the system used.
however, this can fluctuate significantly based on the complexity of the sales data and
the number of attributes stored within each record. 
```


### Migration Architecture & ETL ###
![](https://github.com/gnosia93/tera-to-emr/blob/main/images/teradata-mig.png)

- snowball is not adequate, it also takes time upload data into snowball, delivery, s3 upload time. 
- for fact table 
  - daily based incremental migration with regdate column to reduce risk and due to low bandwidth of network.
- for diemsion table / summary
  - after stopping etl, perform migration
- use SCT for schema convertion
- 처음에는 sct 에이전트를 사용하였으나, 테스트시 hang 이슈가 발생하여 --> tpt 로 교체함.

 
### Risk & Challenge ###
- data integrity
    - table count
    - measure sum, group by sum     
- migration time. -- reduce...해야함..
- sct data agent problems.
  - at first, I used sct ata agent, later on replace sct agent with tpt. 
- SQL compatiblity
  - [rollup / cube](https://www.cloudthat.com/resources/blog/aws-reinvent-2022-new-sql-functionalities-in-amazon-redshift)
  - redshift doesn't support rollup / cube --> implement with cte.




## POC Detail ##


#### Task #1 - [Data Type and SQL Conversion](https://docs.informatica.com/integration-cloud/data-ingestion-and-replication/current-version/database-ingestion-and-replication/database-ingestion-and-replication/default-data-type-mappings/teradata-source-and-amazon-redshift-target.html) ####

- No conversion issue such as bigint, byteint, integer, number(p,s), char, varchar(n), date, time, timestamp.
- For INTERVAL data type, SCT supports conversion.
- There are no datatype such as peroid, XML, LOBS etc.
- There are no procedures & macros.
- Redshift didn't support ROLLUP and CUBE --> implemented with CTE.
- 대소문자 이슈

#### Task #2 - Find optimal data migration architecture  ####
- Considering network bandwith (10 Gbps Internet Backbone) 
- Minimize ETL downtime. 
- provide data migration integrity.


#### Task #3 - Provide Monitoring and Performance Tuning Practice for Post Migration ####

##### 3-1. Performnace Monitoring #####
 
  The performance data that you can use in the Amazon Redshift console falls into two categories:
  
  Amazon CloudWatch metrics – Amazon CloudWatch metrics help you monitor physical aspects of your cluster, such as CPU utilization, latency, and throughput. Metric data is displayed directly in the Amazon Redshift console. You can also view it in the CloudWatch console. Alternatively, you can consume it in any other way you work with metrics, such as with the AWS CLI or one of the AWS SDKs.
  
  Query/Load performance data – Performance data helps you monitor database activity and performance. This data is aggregated in the Amazon Redshift console to help you easily correlate what you see in CloudWatch metrics with specific database query and load events. You can also create your own custom performance queries and run them directly on the database. Query and load performance data is displayed only in the Amazon Redshift console. It is not published as CloudWatch metrics.

##### 3-2. Performance Tuning #####
* https://aws.amazon.com/blogs/big-data/top-10-performance-tuning-techniques-for-amazon-redshift/



#### Item #4 - Provide Cloud ETL with Serverless Architecture ####
- Provided DMS Configuration (parquet parrtitoning)
- Glue ETL Sample Code
- Job Sechduling With Glue
- Operation demonstration  

## Conclusion ##

* POC was conducted for 2 months colloborting with customer.
* Optimal migration strategy was developed with architecture, tools and expected DW downtime.
* Due to steep learning curve and lack of usability of AWS Glue, the customer maintained informatica ETL.


## 레퍼런스 ##

* [Coupang Daily Sales](https://kr.investing.com/pro/NYSE:CPNG/explorer/volume_avg_3m)

## Revision History ##
_2024.11.04 draft released_


