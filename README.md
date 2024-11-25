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



### @@@ Data Type Conversion @@@  ###

* [Accelerate your data warehouse migration to Amazon Redshift – Part 2](https://noise.getoto.net/2021/07/22/accelerate-your-data-warehouse-migration-to-amazon-redshift-part-2/)


* Identify 칼럼은 쓰지 않는다.











@@@ SQL Conversion @@@




@@@ Migration Speed @@@




@@@ Migration Integrity @@@




@@@ Table 갯수 / 가장 큰 테이블 사이즈 ? / Migration 시간 ... ??? @@@@



@@@ Performance Tuning @@@

- Matetiralized View
- DistKey -> Sames as Teradata, SortKey -> 날짜칼럼
- 
