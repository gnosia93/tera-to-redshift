# tera-to-redshift


- Task 1 - Redshift Migration
- Task 2 - ETL & Streaming Processing 

![](https://github.com/gnosia93/tera-to-emr/blob/main/images/teradata-mig.png)


```
TPT will do it easily.
Just use the Export operator and the DataConnector operator (file writer).
If you specify an instance count for the DC operator, the instance count will be the number of files generated with the output from the Export operator.
Additionally, if you use the -C command line argument (highly recommended for file writing) then the data blocks will be written out in a round-robin fashion, meaning the files will be pretty close in size to each other.
Furthermore, you can also use multiple Export operators (and tie them together with the UNION ALL syntax) and each Export operator could export a subset of the data blocks by using a WHERE clause in the SELECT statement.
```

### Customer Context ###

- Department store Operating more than 10 branches
- teradata based DW 

### Migration Method ###

- VPN network
- snowball is not adequate, it also takes time upload data into snowball, delivery, s3 upload time. 
- daily based incremental migration with regdate column to reduce risk and due to low bandwidth of network.
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
- [rollup / cube](https://www.cloudthat.com/resources/blog/aws-reinvent-2022-new-sql-functionalities-in-amazon-redshift)

### 테이블 구성 ### 
- dimension (insert / update)
- fact (only insert) : event(biz tx) based
- a bunch of summary table (only insert) : hourly, daily, monthly.



### post migration ###

- auto distkey / sortkey 사용 ? 
- materialized view 사용 (느린 조인 쿼리에 대해서)  : 테라데이터는 인덱스가 있고, 대신 row format vs 레드쉬프트는 인덱스 없음 columnar.


### @@@ Data Type Conversion @@@  ###

* [Accelerate your data warehouse migration to Amazon Redshift – Part 2](https://noise.getoto.net/2021/07/22/accelerate-your-data-warehouse-migration-to-amazon-redshift-part-2/)




@@@ SQL Conversion @@@




@@@ Migration Speed @@@




@@@ Migration Integrity @@@




@@@ Table 갯수 / 가장 큰 테이블 사이즈 ? / Migration 시간 ... ??? @@@@



@@@ Performance Tuning @@@

- Matetiralized View
- DistKey -> Sames as Teradata, SortKey -> 날짜칼럼
- 
