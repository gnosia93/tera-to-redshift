### 1. create table ###

```
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
    t = threading.Thread(target=db_import, args=(conn_pool, 100000))
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


"""
export sales_fact table to csv
% mysql -u test -h ec2-43-200-2-190.ap-northeast-2.compute.amazonaws.com -p \
-e "SELECT * FROM dw.sales_fact" | sed 's/\t/","/g;s/^/"/;s/$/"/;' > sales_fact.csv
Enter password:
"""
```
