### 1. create table ###

```
DROP TABLE sales_fact;
CREATE TABLE sales_fact (
  sales_key bigint NOT NULL AUTO_INCREMENT,
  id1 bigint NOT NULL,
  id2 bigint NOT NULL,
  id3 bigint NOT NULL,
  id4 bigint NOT NULL,
  id5 bigint NOT NULL,
  id6 bigint NOT NULL,
  id7 bigint NOT NULL,
  id8 bigint NOT NULL,
  id9 bigint NOT NULL,
  id10 bigint NOT NULL,
  id11 bigint NOT NULL,
  id12 bigint NOT NULL,
  id13 bigint NOT NULL,
  id14 bigint NOT NULL,
  id15 bigint NOT NULL,
  id16 bigint NOT NULL,
  id17 bigint NOT NULL,
  id18 bigint NOT NULL,
  id19 bigint NOT NULL,
  id20 bigint NOT NULL,
  id21 bigint NOT NULL,
  id22 bigint NOT NULL,
  id23 bigint NOT NULL,
  id24 bigint NOT NULL,
  id25 bigint NOT NULL,
  id26 bigint NOT NULL,
  id27 bigint NOT NULL,
  dummy1 varchar(200) NOT NULL,
  dummy2 varchar(200) NOT NULL,
  PRIMARY KEY (sales_key)
) ENGINE=InnoDB;
```


### 2. Generate Dummpy Data ###
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

db_bigint_max_value = pow(2, 63) / 1000 - 1      # 9,223,372,036,854,776


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

def db_import(conn_pool, tuple_cnt, commit_interval):
    conn = conn_pool.get_connection()
    cursor = conn.cursor(pymysql.cursors.DictCursor)
  #  commit_interval = tuple_cnt / 10
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
        
        cursor.execute(sql, 
            (
                0, 
                db_bigint_max_value + 1,
                db_bigint_max_value + 2,
                db_bigint_max_value + 3,
                db_bigint_max_value + 4,
                db_bigint_max_value + 5,
                db_bigint_max_value + 6,
                db_bigint_max_value + 7,
                db_bigint_max_value + 8,
                db_bigint_max_value + 9,
                db_bigint_max_value + 10,
                db_bigint_max_value + 11,
                db_bigint_max_value + 12,
                db_bigint_max_value + 13,
                db_bigint_max_value + 14,
                db_bigint_max_value + 15,
                db_bigint_max_value + 16,
                db_bigint_max_value + 17,
                db_bigint_max_value + 18,
                db_bigint_max_value + 19,
                db_bigint_max_value + 20,
                db_bigint_max_value + 21,
                db_bigint_max_value + 22,
                db_bigint_max_value + 23,
                db_bigint_max_value + 24,
                db_bigint_max_value + 25,
                db_bigint_max_value + 26,
                db_bigint_max_value + 27,
                'dummy1_' + str(i), 
                'dummy2_' + str(i)
            )
        )
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

thread_cnt = 100
thread_tuple_cnt = 10000
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


"""
https://stackoverflow.com/questions/71564954/pymysql-error-packet-sequence-number-wrong-got-1-expected-0    
"""
```

### 3. check table size and row length ###
```
mysql> SELECT table_name,
    ->     ROUND((data_length+index_length)/1024/1024, 1) AS 'Size(MB)'
    ->     FROM information_schema.tables
    ->     where table_name = 'sales_fact';
+------------+----------+
| TABLE_NAME | Size(MB) |
+------------+----------+
| sales_fact |    802.0 |
+------------+----------+
1 row in set (0.01 sec)

mysql> select data_length / 1000000
    ->     from information_schema.tables
    ->     where table_schema = 'dw'
    ->     and table_name = 'sales_fact';
+-----------------------+
| data_length / 1000000 |
+-----------------------+
|              840.9580 |
+-----------------------+
1 row in set (0.00 sec)
```

### 4. export to csv file ###
```
start_time=$(date +%s);\
mysql -u test -h ec2-43-200-2-190.ap-northeast-2.compute.amazonaws.com -p \
-e "SELECT * FROM dw.sales_fact" | sed 's/\t/","/g;s/^/"/;s/$/"/;' > sales-fact.csv; \
end_time=$(date +%s); \
elapsed=$(( end_time - start_time )); \
echo $elapsed
```
##### 34 sec / 927,889,106 bytes (885 MB) #####

```
head -n 2 sales-fact.cs
```
"sales_key","id1","id2","id3","id4","id5","id6","id7","id8","id9","id10","id11","id12","id13","id14","id15","id16","id17","id18","id19","id20","id21","id22","id23","id24","id25","id26","id27","dummy1","dummy2"
"1","9223372036854776","9223372036854778","9223372036854780","9223372036854780","9223372036854780","9223372036854782","9223372036854784","9223372036854784","9223372036854784","9223372036854786","9223372036854788","9223372036854788","9223372036854788","9223372036854790","9223372036854792","9223372036854792","9223372036854792","9223372036854794","9223372036854796","9223372036854796","9223372036854796","9223372036854798","9223372036854800","9223372036854800","9223372036854800","9223372036854802","9223372036854804","dummy1_0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000","dummy2_0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000"


### 5. Upload to S3 (multipart upload) ###
aws s3 cp to automatically perform a multipart upload when the object is large

```
start_time=$(date +%s);\
aws s3 cp sales-fact.csv s3://gnosia93-s3-tera-to-redshift; \
end_time=$(date +%s); \
elapsed=$(( end_time - start_time )); \
echo $elapsed
```
Completed 445.0 MiB/884.9 MiB (59.7 MiB/s) with 1 file(s) remainind
15 sec elapsed. 

### 6. Redshift Copy ###

```
DROP TABLE sales_fact;
CREATE TABLE sales_fact (
  sales_key bigint NOT NULL,
  id1 bigint NOT NULL,
  id2 bigint NOT NULL,
  id3 bigint NOT NULL,
  id4 bigint NOT NULL,
  id5 bigint NOT NULL,
  id6 bigint NOT NULL,
  id7 bigint NOT NULL,
  id8 bigint NOT NULL,
  id9 bigint NOT NULL,
  id10 bigint NOT NULL,
  id11 bigint NOT NULL,
  id12 bigint NOT NULL,
  id13 bigint NOT NULL,
  id14 bigint NOT NULL,
  id15 bigint NOT NULL,
  id16 bigint NOT NULL,
  id17 bigint NOT NULL,
  id18 bigint NOT NULL,
  id19 bigint NOT NULL,
  id20 bigint NOT NULL,
  id21 bigint NOT NULL,
  id22 bigint NOT NULL,
  id23 bigint NOT NULL,
  id24 bigint NOT NULL,
  id25 bigint NOT NULL,
  id26 bigint NOT NULL,
  id27 bigint NOT NULL,
  dummy1 varchar(200) NOT NULL,
  dummy2 varchar(200) NOT NULL,
  PRIMARY KEY(sales_key)
) 
DISTKEY(sales_key) 
SORTKEY(sales_key);
```
