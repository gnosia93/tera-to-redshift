* In the middle of refactroing

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


def sql_executor(conn_pool, tuple_cnt, commit_interval, sql, tuple_callback):
    conn = conn_pool.get_connection()
    cursor = conn.cursor(pymysql.cursors.DictCursor)
  #  commit_interval = tuple_cnt / 10
  #  print("commit_interval : ", commit_interval)
       
    for i in range(tuple_cnt):
        tuple = tuple_callback(i)

        cursor.execute(sql, tuple)
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

thread_cnt = 1
thread_tuple_cnt = 3
thread_commit_interval = 2

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
    

def tuple_callback(idx):
    print(idx, ' callback..')
    return (
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
                'dummy1_' + str(idx), 
                'dummy2_' + str(idx)
            )


# generate 1M tuples
for i in range(thread_cnt):
    t = threading.Thread(target=sql_executor, args=(conn_pool, thread_tuple_cnt, thread_commit_interval, sql, tuple_callback))
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
db_export(conn)
conn.close()

```
