
## export from DB ##

* sales_fact.csv 생성
```
% start_time=$(date +%s);\
mysql -u test -h ec2-43-200-2-190.ap-northeast-2.compute.amazonaws.com -p \
-e "SELECT * FROM dw.sales_fact" | sed 's/\t/","/g;s/^/"/;s/$/"/;' > sales_fact.csv; \
end_time=$(date +%s); \
elapsed=$(( end_time - start_time )); \
echo $elapsed
```
20 초

```
% ls -la
total 950408
drwxr-xr-x    4 soonbeom  staff        128 11 27 16:29 .
drwxr-x---+ 124 soonbeom  staff       3968 11 27 16:29 ..
-rw-r--r--    1 soonbeom  staff       2443 11 27 16:27 import.py
-rw-r--r--    1 soonbeom  staff  476889980 11 27 16:38 sales_fact.csv
```

