### 1. create table ###
```
CREATE TABLE product_dim (
  product_key bigint NOT NULL AUTO_INCREMENT,
  id1 bigint NOT NULL,
  id2 bigint NOT NULL,
  id3 bigint NOT NULL,
  id4 bigint NOT NULL,
  id5 bigint NOT NULL,
  id6 bigint NOT NULL,
  id7 bigint NOT NULL,
  id8 bigint NOT NULL,
  id9 bigint NOT NULL,
  dummy1 varchar(200) NOT NULL,
  dummy2 varchar(200) NOT NULL,
  dummy3 varchar(200) NOT NULL,
  dummy4 varchar(200) NOT NULL,
  dummy5 varchar(200) NOT NULL,
  dummy6 varchar(200) NOT NULL,
  dummy7 varchar(200) NOT NULL,
  dummy8 varchar(200) NOT NULL,
  dummy9 varchar(200) NOT NULL,
  sell_yn char(1) NOT NULL,
  regdate datetime DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY(product_key)
) ENGINE=InnoDB;
```

### 2. generate dummy data ###

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
    sql = 'truncate dw.product_dim'
    cursor = conn.cursor()
    cursor.execute(sql)


def db_export(conn):
    sql = 'select count(1) from dw.product_dim'
    cursor = conn.cursor()
    cursor.execute(sql)

    rs = cursor.fetchall()
    for tuple in rs:
        print(tuple)


def db_import(conn_pool, tuple_cnt, commit_interval):
    conn = conn_pool.get_connection()
    cursor = conn.cursor(pymysql.cursors.DictCursor)
  #  commit_interval = tuple_cnt / 10
  #  print("commit_interval : ", commit_interval)
    
    for i in range(tuple_cnt):
        sql = '''insert into dw.product_dim values (
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
             rpad(%s, 200, '1'),
             rpad(%s, 200, '2'),
             rpad(%s, 200, '3'),
             rpad(%s, 200, '4'),
             rpad(%s, 200, '5'),
             rpad(%s, 200, '6'),
             rpad(%s, 200, '7'),
             rpad(%s, 200, '8'),
             rpad(%s, 200, '9'),
             'Y',
             CURRENT_TIMESTAMP
        )'''
                
        cursor.execute(sql, 
            (0, i, i, i, i, i, i, i, i, i, 
                'dummy1_' + str(i), 
                'dummy2_' + str(i),
                'dummy3_' + str(i),
                'dummy4_' + str(i),
                'dummy5_' + str(i),
                'dummy6_' + str(i),
                'dummy7_' + str(i),
                'dummy8_' + str(i),
                'dummy9_' + str(i))
        )
        if i % commit_interval == 0 or i == tuple_cnt - 1:
            print(threading.get_native_id(), ' commit ', i)
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
thread_cnt = 100
thread_tuple_cnt = 100000
thread_commit_interval = 1000

# generate 1M tuples
for i in range(thread_cnt):
    t = threading.Thread(target=db_import, args=(conn_pool, thread_tuple_cnt, thread_commit_interval))
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
```

confirm generated product_dim table data 
```
mysql> select count(1) from dw.product_dim;
+----------+
| count(1) |
+----------+
|  9962015 |
+----------+
1 row in set (4 min 1.81 sec)

mysql> select * from dw.product_dim limit 1 \G
*************************** 1. row ***************************
product_key: 1
        id1: 9223372036854776
        id2: 9223372036854778
        id3: 9223372036854780
        id4: 9223372036854780
        id5: 9223372036854780
        id6: 9223372036854782
        id7: 9223372036854784
        id8: 9223372036854784
        id9: 9223372036854784
     dummy1: dummy1_0111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111
     dummy2: dummy2_0222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222
     dummy3: dummy3_0333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333
     dummy4: dummy4_0444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444
     dummy5: dummy5_0555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555
     dummy6: dummy6_0666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666
     dummy7: dummy7_0777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777
     dummy8: dummy8_0888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888
     dummy9: dummy9_0999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999
    sell_yn: Y
    regdate: 2024-11-27 23:58:38
1 row in set (0.00 sec)
```


### 3. check table size and row length ###
```
mysql> SELECT table_name,
    ->        ROUND((data_length+index_length)/1024/1024, 1) AS 'Size(MB)'
    -> FROM information_schema.tables
    -> where table_name = 'product_dim';
+-------------+----------+
| TABLE_NAME  | Size(MB) |
+-------------+----------+
| product_dim |  26451.0 |
+-------------+----------+
1 row in set (0.00 sec)

mysql> select data_length / 10000000
    -> from information_schema.tables
    -> where table_schema = 'dw'
    -> and table_name = 'product_dim';
+------------------------+
| data_length / 10000000 |
+------------------------+
|              2773.5884 |
+------------------------+
1 row in set (0.00 sec)
```

### 4. export to csv file ###
```
start_time=$(date +%s);\
mysql -u test -h ec2-43-200-2-190.ap-northeast-2.compute.amazonaws.com -p \
-e "SELECT * FROM dw.product_dim" | sed 's/\t/","/g;s/^/"/;s/$/"/;' > product-dim.csv; \
end_time=$(date +%s); \
elapsed=$(( end_time - start_time )); \
echo $elapsed
```
##### 714 sec (around 12 minutes) / 20,261,627,575 bytes (18.87 GB) #####

```
 % head -n 2 product-dim.csv
```

"product_key","id1","id2","id3","id4","id5","id6","id7","id8","id9","dummy1","dummy2","dummy3","dummy4","dummy5","dummy6","dummy7","dummy8","dummy9","sell_yn","regdate"
"1","9223372036854776","9223372036854778","9223372036854780","9223372036854780","9223372036854780","9223372036854782","9223372036854784","9223372036854784","9223372036854784","dummy1_0111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111","dummy2_0222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222","dummy3_0333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333333","dummy4_0444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444444","dummy5_0555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555555","dummy6_0666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666666","dummy7_0777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777777","dummy8_0888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888888","dummy9_0999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999","Y","2024-11-27 23:58:38"


### 5. Upload to S3 (multipart upload) ###
aws s3 cp to automatically perform a multipart upload when the object is large
```
start_time=$(date +%s);\
aws s3 cp product-dim.csv s3://gnosia93-s3-tera-to-redshift; \
end_time=$(date +%s); \
elapsed=$(( end_time - start_time )); \
echo $elapsed
```
##### Completed 3.6 GiB/18.9 GiB (63.7 MiB/s) with 1 file(s) / elapsed 301 sec #####


### 6. Redshift Copy ###
```
DROP TABLE product_dim;

CREATE TABLE product_dim (
  product_key bigint NOT NULL,
  id1 bigint NOT NULL,
  id2 bigint NOT NULL,
  id3 bigint NOT NULL,
  id4 bigint NOT NULL,
  id5 bigint NOT NULL,
  id6 bigint NOT NULL,
  id7 bigint NOT NULL,
  id8 bigint NOT NULL,
  id9 bigint NOT NULL,
  dummy1 varchar(200) NOT NULL,
  dummy2 varchar(200) NOT NULL,
  dummy3 varchar(200) NOT NULL,
  dummy4 varchar(200) NOT NULL,
  dummy5 varchar(200) NOT NULL,
  dummy6 varchar(200) NOT NULL,
  dummy7 varchar(200) NOT NULL,
  dummy8 varchar(200) NOT NULL,
  dummy9 varchar(200) NOT NULL,
  sell_yn char(1) NOT NULL,
  regdate datetime NOT NULL,
  PRIMARY KEY(product_key)
)
DISTKEY(product_key) 
SORTKEY(product_key);
```

```
COPY sales_fact 
FROM 's3://gnosia93-s3-tera-to-redshift/product-dim.csv' 
IAM_ROLE 'arn:aws:iam::499514681453:role/service-role/AmazonRedshift-CommandsAccessRole-20241127T183927'
CSV IGNOREHEADER 1
```



