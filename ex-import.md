
### export from DB ###

![](https://github.com/gnosia93/tera-to-redshift/blob/main/images/sales_fact.png)

avg_row_length = 480bytes

```
select count(1) from dw.sales_fact;
```
레코드 건수 - 100만건

```
SELECT table_name,
       ROUND((data_length+index_length)/1024/1024, 1) AS 'Size(MB)'
FROM information_schema.tables
where table_name = 'sales_fact';
```
DB 테이블 사이즈 - 622 MB 

```
% start_time=$(date +%s);\
mysql -u test -h ec2-43-200-2-190.ap-northeast-2.compute.amazonaws.com -p \
-e "SELECT * FROM dw.sales_fact" | sed 's/\t/","/g;s/^/"/;s/$/"/;' > sales_fact.csv; \
end_time=$(date +%s); \
elapsed=$(( end_time - start_time )); \
echo $elapsed
```
CSV 파일 생성시간 - 20 초

```
% ls -la
total 950408
drwxr-xr-x    4 soonbeom  staff        128 11 27 16:29 .
drwxr-x---+ 124 soonbeom  staff       3968 11 27 16:29 ..
-rw-r--r--    1 soonbeom  staff       2443 11 27 16:27 import.py
-rw-r--r--    1 soonbeom  staff  476889980 11 27 16:38 sales_fact.csv
```
파일 사이즈 - 454.79 MB

아래는 CSV 파일 내용
![](https://github.com/gnosia93/tera-to-redshift/blob/main/images/sales_fact_samples.png)




### Upload to S3 ###
