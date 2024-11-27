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




