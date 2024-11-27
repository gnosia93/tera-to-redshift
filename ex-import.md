### Generate dummy sales_fact table ###
100 만건을 생성한다.
```
import pymysql
import pymysqlpool
import time
import threading

host = 'ec2-43-200-2-190.ap-northeast-2.compute.amazonaws.com'
user = 'test'
password = 'test'
db = 'dw'
charset = 'utf8'

config = {'host':host, 'user':user, 'password':password, 'database':db, 'charset':charset}
conn_pool = pymysqlpool.ConnectionPool(size = 100, maxsize = 200, name = 'db_pool', **config)

def db_trunc(conn):
    sql = 'truncate dw.sales_fact'
    cursor = conn.cursor()
    cursor.execute(sql)

def db_export(conn):
    sql = 'select count(1) from dw.sales_fact'
    cursor = conn.cursor()
    cursor.execute(sql)

    rs = cursor.fetchall()
    for tuple in rs:
        print(tuple)

def db_import(conn_pool, tuple_cnt):
    conn = conn_pool.get_connection()
    cursor = conn.cursor(pymysql.cursors.DictCursor)
    commit_interval = tuple_cnt / 10
  #  print("commit_interval : ", commit_interval)
    
    for i in range(tuple_cnt):
        sql = '''insert into dw.sales_fact values (
            %s,
             %s,
             %s,
             %s,
             %s,
             %s,
             %s,
             %s,
             %s,
             %s,
             rpad(%s, 200, '0'),
             rpad(%s, 200, '0')
        )'''
        
        cursor.execute(sql, (0, i, i, i, i, i, i, i, i, i, 'dummy1_' + str(i), 'dummy2_' + str(i)))
        if i % commit_interval == 0 or i == tuple_cnt - 1:
            conn.commit()
            
    conn.close()

import_cnt = 1000000

# truncate table
conn = conn_pool.get_connection()
db_trunc(conn)
conn.close()

# import table
thread_pool = []
start_time = time.time()

# generate 1M tuples
for i in range(100):
    t = threading.Thread(target=db_import, args=(conn_pool, 10000))
    t.start()
    thread_pool.append(t)
    print(i, ' thread ', t.name, ' started...' )

# print(thread_pool)
for t in thread_pool:
    print(t.name, ' joined...')
    t.join()

end_time = time.time()
elapsed_time = end_time - start_time

print(f"Elapsed time for import DB: {elapsed_time} seconds")

# export table
conn = conn_pool.get_connection()
# db_export(conn)
conn.close()

"""
https://stackoverflow.com/questions/71564954/pymysql-error-packet-sequence-number-wrong-got-1-expected-0    
"""
```




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

버킷생성
```
aws s3api create-bucket \
    --bucket gnosia93-s3-tera-to-redshift \
    --region ap-northeast-2 \
    --create-bucket-configuration LocationConstraint=ap-northeast-2
```

파일업로드 시간측정 (10초)
```
% start_time=$(date +%s);\
aws s3 cp sales_fact_old.csv s3://gnosia93-s3-tera-to-redshift; \
end_time=$(date +%s); \
elapsed=$(( end_time - start_time )); \
echo $elapsed
```

