# tera-to-redshift


- Task 1 - Redshift Migration
- Task 2 - ETL with AWS Glue 

![](https://github.com/gnosia93/tera-to-emr/blob/main/images/teradata-mig.png)


### Customer Context ###

- Built EDW on top of Teradata appliance at onpremise.
- Data is ingested using ETL from department store(9) and online mall.
- Analyze their customer and marketing data and wanted to reduce TCO.
- Considering Predictive analytics in addtion to Descriptive analytics(BI).

한국 테라데이타 (www.teradata.kr 대표 콘바스)는 애경그룹이 자사의 DW 플랫폼을 ‘그룹 통합 DW 솔루션’으로 채택했다고 13일 라스베가스에서 개막된 '테라데이타 파트너스 사용자 컨퍼런스'에서 발표했다.
애경그룹(www.aekyung.co.kr, 회장 장영신)은 테라데이타 플랫폼을 전사적 DW 시스템으로 구축해 마케팅 및 고객에 대한 정보를 날짜별로 저장하고 통합함으로써 그룹 전체에 대한 비즈니스 통찰력을 얻고 이를 통해 마케팅 전개 및 대고객 서비스 강화를 실현할 수 있을 것으로 기대하고 있다. 애경그룹의 프로젝트는 지난 8월 시작해 2009년 4월까지 진행 될 예정이며 애경 백화점, 삼성플라자, 애경 면세점, 온라인 삼성몰을 시작으로 향후 제주항공까지 구축할 계획이다. 애경그룹은 통합된 고객 정보를 통해 마케팅 생산성을 극대화하고 향후 신규 시장 개척에도 활용한다는 방침이다. 한국테라데이타의 콘바스 사장은 “지난 9월에 출시한 테라데이타 플랫폼의 첫 번째 레퍼런스를 확보하게 됐다"며 "이를 계기로 비즈니스 SI사와 기술 중심 SI사와 협력하여 국내 DW 시장을 적극 공략해 나갈 방침”이라고 말했다.

Descriptive analytics: A traditional business intelligence method that uses visualizations like pie charts, bar charts, tables, or line graphs. It's the foundation for other types of analytics. For example, you can use descriptive analytics to determine how many sales occurred in a quarter and if they increased or decreased. 
Predictive analytics: Uses historical data to predict likely outcomes and make educated forecasts. It's a more complex type of analytics that uses probabilities instead of simply interpreting existing facts. 

- https://m.blog.naver.com/iskrahee/70130255093 


### Proposal ###
proposed Redshift based DW with following advantage   

- Amazon Redshift is a fully managed, petabyte-scale data warehouse that can handle petabytes of data.
- Has similiar achitecture with teradta (MPP, postgres based), but much more cheaper
- Columnar based storage, which is more appropriate for DW.
- Integration with S3 using redshift spectrum, which offload table data into S3.    
- Seamlessly integrated with AWS AI/ML and analytics services.


### POC ###

Check the feasiblity of redshift migration and provide optimal migration strategy.

- Task #1 - Data Type and SQL Conversion _(with minial manual efforts)_
- Task #2 - Find & Provide optimal data migration methodology.
- Task #3 - Providing Monitoring and Performance Tuning Method.

* https://github.com/al-ghaly/Data-Warehouse-Case-Study?tab=readme-ov-file 

  

#### Migration Volumes ####

- 2TB Stoage Volumn
- About 70 tables.
- dimension (insert / update)
- fact (only insert) : event(biz tx) based
- a bunch of summary table (only insert) : daily, monthly.

@@@ sales_fact @@@
![](https://github.com/gnosia93/tera-to-redshift/blob/main/fact-design.png)

- columns : 30 개
- datatype : small int / int / big int / char / varchar, numeric(decimal) 
- record size : 480 bytes (16 byte * 30개 = 480 bytes) 
- avg insert : 82 만건 (daily 375 MB)
  - 10.99 gb / month, 131.96 gb / year ---> 659.82 gb / 5 years 
- estimated migration time for daily data 
  - export time : xx sec
  - s3 upload time : xx sec
  - redshift import time : xx sec

```
A typical "sales fact" record in a data warehouse, depending on the level of detail captured, usually averages around 2 kilobytes (KB) in size, with some variations depending on the specific data fields included and the system used; however, this can fluctuate significantly based on the complexity of the sales data and the number of attributes stored within each record. 
```


@@@ Sales TX Sizeing @@@
- 1일 판매건수 : 273,972 개 (분당점, 수원점)
- 1일 판매건수 (9개점 + 온라인) : 821,917 개 



### Migration Method ###

- VPN network
- snowball is not adequate, it also takes time upload data into snowball, delivery, s3 upload time. 
- for fact table 
  - daily based incremental migration with regdate column to reduce risk and due to low bandwidth of network.
- for diemsion table / summary
  - after stopping etl, perform migration
    
- use SCT for schema convertion
    - data type conversion
    - length issue.
    - not converted SQL issue..

- review DMS / glue for next gen architecture for ETL, drop. due to costs. 
    - used informatica as it is.. 그대로 사용함.
      
- realtime streaming analysis.
    - ?

- 처음에는 sct 에이전트를 사용하였으나, 테스트시 hang 이슈가 발생하여 --> tpt 로 교체함.

 
### RISK / Challenge ###
- data integrity
    - table count
    - measure sum, group by sum     
- migration time. -- reduce...해야함..
- sct data agent problems.
  - at first, I used sct ata agent, later on replace sct agent with tpt. 
- SQL compatiblity
  - [rollup / cube](https://www.cloudthat.com/resources/blog/aws-reinvent-2022-new-sql-functionalities-in-amazon-redshift)
  - redshift doesn't support rollup / cube --> implement with cte.




### post migration ###

- auto distkey / sortkey 사용 ? 
- materialized view 사용 (느린 조인 쿼리에 대해서)  : 테라데이터는 인덱스가 있고, 대신 row format vs 레드쉬프트는 인덱스 없음 columnar.



#### Task #1 - [Data Type and SQL Conversion](https://docs.informatica.com/integration-cloud/data-ingestion-and-replication/current-version/database-ingestion-and-replication/database-ingestion-and-replication/default-data-type-mappings/teradata-source-and-amazon-redshift-target.html) ####

- No conversion problems such as bigint, byteint, integer, number(p,s), char, varchar(n), date, time, timestamp.
- For INTERVAL data type, SCT supports conversion.
- There are no procedures & macros.
- Redshift didn't support ROLLUP and CUBE --> implemented with CTE.
- 대소문자 이슈

#### Task #2 - Find & Provide optimal data migration methodology ####
- Considering network bandwith.
- Minimize ETL downtime. 
- provide data migration integrity.


#### Task #3 - Providing Monitoring and Performance Tuning Method. ####
- Matetiralized View
- DistKey / SortKey -> Auto
- 분산키 설계(DistKey Design)




## 참고자료 ##

* [Datatype Conversion - Accelerate your data warehouse migration to Amazon Redshift – Part 2](https://noise.getoto.net/2021/07/22/accelerate-your-data-warehouse-migration-to-amazon-redshift-part-2/)
* [DMS with VPN](https://dev.to/haintkit/case-study-how-to-replicate-database-from-aws-to-outside-3obc)

* TPT
```
TPT will do it easily.
Just use the Export operator and the DataConnector operator (file writer).
If you specify an instance count for the DC operator, the instance count will be the number of files generated with the output from the Export operator.
Additionally, if you use the -C command line argument (highly recommended for file writing) then the data blocks will be written out in a round-robin fashion, meaning the files will be pretty close in size to each other.
Furthermore, you can also use multiple Export operators (and tie them together with the UNION ALL syntax) and each Export operator could export a subset of the data blocks by using a WHERE clause in the SELECT statement.
```

* Interval
```
```
