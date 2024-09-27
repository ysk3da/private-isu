# 2024-08-23

補助チケ 10分 × 2
うたにさん、さのげさん、富所さんを召喚できる

Google Form で特典を入れる

## 初期起動

~~~sh
./bin/benchmarker -t "http://localhost" -u ./userdata
{"pass":true,"score":0,"success":273,"fail":44,"messages":["リクエストがタイムアウトしました (GET /)","リクエストがタイムアウトしました (GET /@vicky)","リクエストがタイムアウトしました (GET /image/1350.jpg)","リクエストがタイムアウトしました (GET /image/3300.jpg)","リクエストがタイムアウトしました (GET /image/919.jpg)","リクエストがタイムアウトしました (GET /image/9989.jpg)","リクエストがタイムアウトしました (GET /image/9995.jpg)","リクエストがタイムアウトしました (POST /login)","リクエストがタイムアウトしました (POST /register)"]}
~~~


## mysqlのindexをはるを確認した

下記コマンドでmysqlコンテナに入ってついでにmysqlを起動できる

~~~sh
docker compose exec -it mysql mysql -uroot -proot isuconp
~~~


- [MySQLのインデックスについて整理する](https://qiita.com/h_tyokinuhata/items/10c4bd346f40637de3ba)

インデックス入っている？

~~~sh
mysql> show tables;
# 以下 mysql内
+-------------------+
| Tables_in_isuconp |
+-------------------+
| comments          |
| posts             |
| users             |
+-------------------+
3 rows in set (0.00 sec)

mysql> show index from comments;
+----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| Table    | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
+----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| comments |          0 | PRIMARY  |            1 | id          | A         |       99185 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
+----------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
1 row in set (0.01 sec)

mysql> show index from posts;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| posts |          0 | PRIMARY  |            1 | id          | A         |        8330 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
1 row in set (0.00 sec)

mysql> show index from users;
+-------+------------+--------------+--------------+--------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| Table | Non_unique | Key_name     | Seq_in_index | Column_name  | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
+-------+------------+--------------+--------------+--------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| users |          0 | PRIMARY      |            1 | id           | A         |         999 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
| users |          0 | account_name |            1 | account_name | A         |         999 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
+-------+------------+--------------+--------------+--------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
2 rows in set (0.01 sec)

~~~

はってあるっぽいね

本のやつやってみる

~~~sh
ALTER TABLE comments ADD INDEX post_id_idx(post_id, created_at DESC);
~~~

### copilotによる解説

このSQL文は、`comments`というテーブルに対してインデックスを追加するためのものです。
具体的には、`post_id`と`created_at`の2つのカラムに基づいてインデックスを作成します。

このインデックスの名前は`post_id_idx`です。

インデックスを追加することで、データベースの検索性能を向上させることができます。
特に、`post_id`と`created_at`の組み合わせで検索を行うクエリの実行速度が速くなります。
`created_at DESC`と指定することで、`created_at`カラムの値が降順でソートされるようになります。
これにより、例えば最新のコメントを素早く取得するようなクエリが効率的に実行されます。

このインデックスは、**データベースが`post_id`と`created_at`の組み合わせを効率的に検索できるようにするためのものです。**
インデックスを適切に設計することで、データベースのパフォーマンスを大幅に向上させることができます。

#### ベンチマーク

~~~sh
./bin/benchmarker -t "http://localhost" -u ./userdata
#
{"pass":true,"score":8877,"success":7915,"fail":0,"messages":[]}
~~~

単純に向上した.


### わかったこと

最初の
> "messages":["リクエストがタイムアウトしました (GET /)","リクエストがタイムアウトしました (GET /@vicky)","リクエストがタイムアウトしました (GET /image/1350.jpg)","リクエストがタイムアウトしました (GET /image/3300.jpg)","リクエストがタイムアウトしました (GET /image/919.jpg)","リクエストがタイムアウトしました (GET /image/9989.jpg)","リクエストがタイムアウトしました (GET /image/9995.jpg)","リクエストがタイムアウトしました (POST /login)","リクエストがタイムアウトしました (POST /register)"]}

これがヒントで

postのコメントのインデックスを複合インデックスにしなさいよ、ということらしい

確認すると

~~~sh
mysql> show index from comments;
+----------+------------+-------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| Table    | Non_unique | Key_name    | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
+----------+------------+-------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
| comments |          0 | PRIMARY     |            1 | id          | A         |       99185 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
| comments |          1 | post_id_idx |            1 | post_id     | A         |       10060 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
| comments |          1 | post_id_idx |            2 | created_at  | D         |       99274 |     NULL |   NULL |      | BTREE      |         |               | YES     | NULL       |
+----------+------------+-------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+---------+------------+
3 rows in set (0.03 sec)
~~~


## alp ってやつを入れてみる

~~~sh
asdf plugin add alp

asdf install alp latest

asdf global alp latest
~~~

んでnginxの設定を変える


~~~sh
git diff
# 以下出力
diff --git a/webapp/etc/nginx/conf.d/php.conf b/webapp/etc/nginx/conf.d/php.conf
index 6cf6834..93d1923 100644
--- a/webapp/etc/nginx/conf.d/php.conf
+++ b/webapp/etc/nginx/conf.d/php.conf
@@ -1,3 +1,15 @@
+log_format json escape=json '{"time":"$time_iso8601",'
+    '"host":"$remote_addr",'
+    '"port":$remote_port,'
+    '"method":"$request_method",'
+    '"uri":"$request_uri",'
+    '"status":"$status",'
+    '"body_bytes":$body_bytes_sent,'
+    '"referer":"$http_referer",'
+    '"ua":"$http_user_agent",'
+    '"request_time":"$request_time",'
+    '"response_time":"$upstream_response_time"}';
+
 server {
     listen 80;
 
@@ -17,4 +29,5 @@ server {
         fastcgi_index index.php;
         fastcgi_pass app:9000;
     }
+    access_log /var/log/nginx/access.log json;
 }
(END)
~~~

nginxを再起動

~~~sh
docker compose exec -it nginx bash
root@2ef20beb65e4:/# cat /var/log/nginx/access.log | alp json
# ですよね^^;
bash: alp: command not found
~~~

そうじゃなくて
こうらしい

~~~sh
cd webapp
cat logs/nginx/access.log|alp json
~~~


設定が不足していた

こうだね

~~~sh
git diff
diff --git a/webapp/docker-compose.yml b/webapp/docker-compose.yml
index a510d56..a7eaa3e 100644
--- a/webapp/docker-compose.yml
+++ b/webapp/docker-compose.yml
@@ -5,6 +5,7 @@ services:
     volumes:
       - ./etc/nginx/conf.d:/etc/nginx/conf.d
       - ./public:/public
+      - ./logs/nginx:/var/log/nginx
     ports:
       - "80:80"
     networks:
@@ -45,6 +46,7 @@ services:
     volumes:
       - mysql:/var/lib/mysql
       - ./sql:/docker-entrypoint-initdb.d
+      - ./logs/mysql:/var/log/mysql
     ports:
       - "3306:3306"
     networks:
diff --git a/webapp/etc/nginx/conf.d/php.conf b/webapp/etc/nginx/conf.d/php.conf
index 6cf6834..93d1923 100644
--- a/webapp/etc/nginx/conf.d/php.conf
+++ b/webapp/etc/nginx/conf.d/php.conf
@@ -1,3 +1,15 @@
+log_format json escape=json '{"time":"$time_iso8601",'
+    '"host":"$remote_addr",'
+    '"port":$remote_port,'
+    '"method":"$request_method",'
+    '"uri":"$request_uri",'
+    '"status":"$status",'
+    '"body_bytes":$body_bytes_sent,'
+    '"referer":"$http_referer",'
+    '"ua":"$http_user_agent",'
+    '"request_time":"$request_time",'
+    '"response_time":"$upstream_response_time"}';
+
 server {
     listen 80;
 
@@ -17,4 +29,5 @@ server {
         fastcgi_index index.php;
         fastcgi_pass app:9000;
     }
+    access_log /var/log/nginx/access.log json;
 }
(END)
~~~

なんもでない

あ、ベンチマークしてから見ればいいのか

でた！！

~~~sh
cat logs/nginx/access.log|alp json -m '/api/estate/[0-9]+,/api/chair/[0-9]+,/api/recommended_estate/[0-9]+' --sort avg -r --show-footers -o count,1xx,2xx,3xx,4xx,5xx,min,max,avg,sum,p99
~~~

なんか 498 回とか呼ばれているやつがいるな


## perconaの方入れてみる

~~~sh
brew install percona-toolkit
~~~


んでこう

むむ
なぜか my.cnfが通らない

できた

~~~sh
git diff
diff --git a/webapp/docker-compose.yml b/webapp/docker-compose.yml
index a510d56..89eb35b 100644
--- a/webapp/docker-compose.yml
+++ b/webapp/docker-compose.yml
@@ -5,6 +5,7 @@ services:
     volumes:
       - ./etc/nginx/conf.d:/etc/nginx/conf.d
       - ./public:/public
+      - ./logs/nginx:/var/log/nginx
     ports:
       - "80:80"
     networks:
@@ -44,7 +45,9 @@ services:
       - "MYSQL_ROOT_PASSWORD=root"
     volumes:
       - mysql:/var/lib/mysql
      - ./etc/mysql/my.cnf:/etc/mysql/my.cnf
       - ./sql:/docker-entrypoint-initdb.d
+      - ./logs/mysql:/var/log/mysql
     ports:
       - "3306:3306"
     networks:
~~~


`webapp/etc/mysql/my.cnf`

~~~sh
[mysqld]

slow_query_log=1
long_query_time=0

[mysqld_safe]
slow_query_log_file=/var/log/mysql/mysqlslow.log
~~~

ベンチマークを起動してから

下記を見てみよう

~~~sh
pt-query-digest --type slowlog ./logs/mysql/mysqlslow.log

# No events processed.
~~~

?? 一旦おいておく


mysql側でslowクエリログをonにした
https://qiita.com/kondo0602/items/b4c3c84579c8c2c1b3be

パーミッションの関係でlogファイルがアクセスできていない

~~~sh
mysql-1  | 2024-08-23 07:56:44+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.4.2-1.el9 started.
mysql-1  | chown: changing ownership of '/var/log/mysql': Permission denied
mysql-1  | chown: changing ownership of '/var/log/mysql/slow': Permission denied
mysql-1  | chown: changing ownership of '/var/log/mysql/mysqlslow.log': Permission denied
~~~


mysqlの構築もDockerfileからに変更した

見れるようになった！

~~~sh
pt-query-digest --type slowlog ./logs/mysql/mysqlslow.log

# 190ms user time, 30ms system time, 34.02M rss, 391.57G vsz
# Current date: Fri Aug 23 17:31:14 2024
# Hostname: kaopc-0905
# Files: ./logs/mysql/mysqlslow.log
# Overall: 960 total, 7 unique, 14.33 QPS, 0.17x concurrency _____________
# Time range: 2024-08-23T08:29:52 to 2024-08-23T08:30:59
# Attribute          total     min     max     avg     95%  stddev  median
# ============     ======= ======= ======= ======= ======= ======= =======
# Exec time            12s     1ms    77ms    12ms    34ms    11ms    10ms
# Lock time          878us       0   160us       0     1us     5us     1us
# Rows sent          5.60M       0   9.83k   5.98k   9.80k   4.65k   9.33k
# Rows examine      25.41M    1000  97.75k  27.11k  97.04k  27.36k  19.40k
# Query size        83.23k      28     142   88.78  136.99   23.04   88.31

# Profile
# Rank Query ID                            Response time Calls R/Call V/M 
# ==== =================================== ============= ===== ====== ====
#    1 0x4858CF4D8CAA743E839C127C71B69E75   8.4222 72.0%   526 0.0160  0.01 SELECT posts
#    2 0xCDEB1AFF2AE2BE51B2ED5CF03D4E749F   1.5291 13.1%   124 0.0123  0.00 SELECT comments
#    3 0x7A12D0C8F433684C3027353C36CAB572   1.0588  9.0%    60 0.0176  0.01 SELECT posts
#    4 0xAA65B65D6FEC3514934B143907BBE8A0   0.4663  4.0%   124 0.0038  0.00 SELECT posts
# MISC 0xMISC                               0.2260  1.9%   126 0.0018   0.0 <3 ITEMS>

# Query 1: 8.09 QPS, 0.13x concurrency, ID 0x4858CF4D8CAA743E839C127C71B69E75 at byte 40700
# This item is included in the report because it matches --limit.
# Scores: V/M = 0.01
# Time range: 2024-08-23T08:29:52 to 2024-08-23T08:30:57
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count         54     526
# Exec time     71      8s     7ms    77ms    16ms    53ms    13ms    11ms
# Lock time     61   542us       0   160us     1us     1us     7us     1us
# Rows sent     89   5.03M   9.77k   9.83k   9.80k   9.80k      31   9.33k
# Rows examine  39  10.06M  19.53k  19.65k  19.59k  19.40k       0  19.40k
# Query size    56  47.26k      92      92      92      92       0      92
# String:
# Databases    isuconp
# Hosts        172.30.0.5
# Users        root
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms  ###############
#  10ms  ################################################################
# 100ms
#    1s
#  10s+
# Tables
#    SHOW TABLE STATUS FROM `isuconp` LIKE 'posts'\G
#    SHOW CREATE TABLE `isuconp`.`posts`\G
# EXPLAIN /*!50100 PARTITIONS*/
SELECT `id`, `user_id`, `body`, `mime`, `created_at` FROM `posts` ORDER BY `created_at` DESC\G

# Query 2: 2.07 QPS, 0.03x concurrency, ID 0xCDEB1AFF2AE2BE51B2ED5CF03D4E749F at byte 1434
# This item is included in the report because it matches --limit.
# Scores: V/M = 0.00
# Time range: 2024-08-23T08:29:53 to 2024-08-23T08:30:53
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count         12     124
# Exec time     13      2s     8ms    36ms    12ms    16ms     4ms    11ms
# Lock time      5    47us       0     5us       0     1us       0       0
# Rows sent      0     124       1       1       1       1       0       1
# Rows examine  46  11.83M  97.66k  97.75k  97.70k  97.04k       0  97.04k
# Query size     9   7.74k      63      64   63.93   62.76       0   62.76
# String:
# Databases    isuconp
# Hosts        172.30.0.5
# Users        root
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms  #####
#  10ms  ################################################################
# 100ms
#    1s
#  10s+
# Tables
#    SHOW TABLE STATUS FROM `isuconp` LIKE 'comments'\G
#    SHOW CREATE TABLE `isuconp`.`comments`\G
# EXPLAIN /*!50100 PARTITIONS*/
SELECT COUNT(*) AS count FROM `comments` WHERE `user_id` = '599'\G

# Query 3: 0.94 QPS, 0.02x concurrency, ID 0x7A12D0C8F433684C3027353C36CAB572 at byte 91238
# This item is included in the report because it matches --limit.
# Scores: V/M = 0.01
# Time range: 2024-08-23T08:29:55 to 2024-08-23T08:30:59
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count          6      60
# Exec time      9      1s     9ms    67ms    18ms    53ms    13ms    13ms
# Lock time      4    40us       0     2us       0     1us       0     1us
# Rows sent     10 582.22k   9.58k   9.76k   9.70k   9.33k       0   9.33k
# Rows examine   4   1.14M  19.40k  19.58k  19.51k  19.40k       0  19.40k
# Query size     9   8.32k     142     142     142     142       0     142
# String:
# Databases    isuconp
# Hosts        172.30.0.5
# Users        root
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms  #####
#  10ms  ################################################################
# 100ms
#    1s
#  10s+
# Tables
#    SHOW TABLE STATUS FROM `isuconp` LIKE 'posts'\G
#    SHOW CREATE TABLE `isuconp`.`posts`\G
# EXPLAIN /*!50100 PARTITIONS*/
SELECT `id`, `user_id`, `body`, `mime`, `created_at` FROM `posts` WHERE `created_at` <= '2016-01-02T11:45:26+09:00' ORDER BY `created_at` DESC\G

# Query 4: 2.03 QPS, 0.01x concurrency, ID 0xAA65B65D6FEC3514934B143907BBE8A0 at byte 107460
# This item is included in the report because it matches --limit.
# Scores: V/M = 0.00
# Time range: 2024-08-23T08:29:52 to 2024-08-23T08:30:53
# Attribute    pct   total     min     max     avg     95%  stddev  median
# ============ === ======= ======= ======= ======= ======= ======= =======
# Count         12     124
# Exec time      3   466ms     2ms    21ms     4ms     6ms     2ms     3ms
# Lock time     11    97us       0    12us       0     1us     1us     1us
# Rows sent      0   1.25k       2      20   10.31   14.52    2.91   10.84
# Rows examine   4   1.19M   9.77k   9.84k   9.81k   9.80k      38   9.80k
# Query size    16  14.04k     115     116  115.93  112.70       0  112.70
# String:
# Databases    isuconp
# Hosts        172.30.0.5
# Users        root
# Query_time distribution
#   1us
#  10us
# 100us
#   1ms  ################################################################
#  10ms  #
# 100ms
#    1s
#  10s+
# Tables
#    SHOW TABLE STATUS FROM `isuconp` LIKE 'posts'\G
#    SHOW CREATE TABLE `isuconp`.`posts`\G
# EXPLAIN /*!50100 PARTITIONS*/
SELECT `id`, `user_id`, `body`, `created_at`, `mime` FROM `posts` WHERE `user_id` = '855' ORDER BY `created_at` DESC\G
~~~

## 静的ファイルをnginxで配信する?

~~~sh
git diff
diff --git a/webapp/etc/nginx/conf.d/php.conf b/webapp/etc/nginx/conf.d/php.conf
index 93d1923..c9cb92a 100644
--- a/webapp/etc/nginx/conf.d/php.conf
+++ b/webapp/etc/nginx/conf.d/php.conf
@@ -19,7 +19,10 @@ server {
     location / {
         try_files $uri /index.php$is_args$args;
     }
-
+    location ~ ^/(favicon\.ico|css/|js/|img/) {
+        root /public;
+        expires 1d;
+    }
     # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
     location ~ \.php {
         #fastcgi_split_path_info ^(.+\.php)(/.+)$;
~~~

nginxを再起動

~~~sh
docker compose down nginx && docker compose up nginx -d
~~~

alpで確認する

~~~sh
cat logs/nginx/access.log|alp json -m "/image/.+,/posts/[0-9]+,/@.+"

+-------+-----+-------+------+-----+-----+--------+--------------------+-------+-------+----------+-------+-------+-------+-------+--------+-----------+-------------+----------------+------------+
| COUNT | 1XX |  2XX  | 3XX  | 4XX | 5XX | METHOD |        URI         |  MIN  |  MAX  |   SUM    |  AVG  |  P90  |  P95  |  P99  | STDDEV | MIN(BODY) |  MAX(BODY)  |   SUM(BODY)    | AVG(BODY)  |
+-------+-----+-------+------+-----+-----+--------+--------------------+-------+-------+----------+-------+-------+-------+-------+--------+-----------+-------------+----------------+------------+
| 5     | 0   | 5     | 0    | 0   | 0   | GET    | /initialize        | 0.082 | 1.028 | 1.992    | 0.398 | 1.028 | 1.028 | 1.028 | 0.326  | 5.000     | 7008.000    | 7028.000       | 1405.600   |
| 93    | 0   | 0     | 93   | 0   | 0   | GET    | /logout            | 0.005 | 0.199 | 7.475    | 0.080 | 0.114 | 0.158 | 0.199 | 0.037  | 5.000     | 5.000       | 465.000        | 5.000      |
| 186   | 0   | 186   | 0    | 0   | 0   | GET    | /login             | 0.006 | 0.194 | 15.570   | 0.084 | 0.107 | 0.159 | 0.186 | 0.036  | 1240.000  | 1240.000    | 230640.000     | 1240.000   |
| 206   | 0   | 206   | 0    | 0   | 0   | GET    | /posts             | 0.033 | 0.431 | 36.622   | 0.178 | 0.283 | 0.301 | 0.354 | 0.077  | 34470.000 | 35660.000   | 7196564.000    | 34934.777  |
| 326   | 0   | 0     | 326  | 0   | 0   | POST   | /comment           | 0.008 | 0.297 | 33.182   | 0.102 | 0.157 | 0.169 | 0.182 | 0.035  | 5.000     | 5.000       | 1630.000       | 5.000      |
| 330   | 0   | 0     | 0    | 330 | 0   | GET    | /admin/banned      | 0.006 | 0.261 | 29.418   | 0.089 | 0.124 | 0.160 | 0.185 | 0.032  | 13.000    | 13.000      | 4290.000       | 13.000     |
| 331   | 0   | 1     | 330  | 0   | 0   | POST   | /register          | 0.013 | 1.023 | 44.574   | 0.135 | 0.183 | 0.195 | 0.262 | 0.064  | 5.000     | 7008.000    | 8658.000       | 26.157     |
| 446   | 0   | 0     | 223  | 223 | 0   | POST   | /                  | 0.006 | 0.340 | 50.158   | 0.112 | 0.176 | 0.202 | 0.257 | 0.050  | 5.000     | 13.000      | 4014.000       | 9.000      |
| 449   | 0   | 449   | 0    | 0   | 0   | GET    | /@.+               | 0.030 | 0.358 | 77.825   | 0.173 | 0.241 | 0.267 | 0.305 | 0.051  | 4509.000  | 33644.000   | 8123346.000    | 18092.085  |
| 1391  | 0   | 7     | 1384 | 0   | 0   | POST   | /login             | 0.007 | 1.032 | 164.055  | 0.118 | 0.170 | 0.178 | 0.249 | 0.076  | 5.000     | 7014.000    | 55982.000      | 40.246     |
| 1751  | 0   | 1751  | 0    | 0   | 0   | GET    | /posts/[0-9]+      | 0.007 | 0.298 | 184.531  | 0.105 | 0.163 | 0.173 | 0.196 | 0.036  | 1563.000  | 6522.000    | 6293523.000    | 3594.245   |
| 1777  | 0   | 1407  | 370  | 0   | 0   | GET    | /js/main.js        | 0.000 | 0.000 | 0.000    | 0.000 | 0.000 | 0.000 | 0.000 | 0.000  | 0.000     | 1824.000    | 2566368.000    | 1444.214   |
| 1777  | 0   | 1407  | 370  | 0   | 0   | GET    | /css/style.css     | 0.000 | 0.000 | 0.000    | 0.000 | 0.000 | 0.000 | 0.000 | 0.000  | 0.000     | 1549.000    | 2179443.000    | 1226.473   |
| 1778  | 0   | 1407  | 371  | 0   | 0   | GET    | /favicon.ico       | 0.000 | 0.000 | 0.000    | 0.000 | 0.000 | 0.000 | 0.000 | 0.000  | 0.000     | 43.000      | 60501.000      | 34.028     |
| 1778  | 0   | 1407  | 371  | 0   | 0   | GET    | /js/timeago.min.js | 0.000 | 0.000 | 0.000    | 0.000 | 0.000 | 0.000 | 0.000 | 0.000  | 0.000     | 1915.000    | 2694405.000    | 1515.413   |
| 1887  | 0   | 1887  | 0    | 0   | 0   | GET    | /                  | 0.030 | 1.031 | 308.923  | 0.164 | 0.215 | 0.234 | 0.293 | 0.052  | 7008.000  | 35772.000   | 46315914.000   | 24544.734  |
| 15783 | 0   | 15783 | 0    | 0   | 0   | GET    | /image/.+          | 0.007 | 0.600 | 1430.715 | 0.091 | 0.147 | 0.160 | 0.182 | 0.038  | 36570.000 | 1253004.000 | 4585399437.000 | 290527.747 |
+-------+-----+-------+------+-----+-----+--------+--------------------+-------+-------+----------+-------+-------+-------+-------+--------+-----------+-------------+----------------+------------+
~~~

あんまカイゼンしてない？

~~~sh
 ./bin/benchmarker -t "http://localhost" -u ./userdata
{"pass":true,"score":9329,"success":8294,"fail":0,"messages":[]}
~~~

## ファイルがあれば返す、なければphpに流す設定をする

~~~sh
diff --git a/webapp/etc/nginx/conf.d/php.conf b/webapp/etc/nginx/conf.d/php.conf
index c9cb92a..2ec0971 100644
--- a/webapp/etc/nginx/conf.d/php.conf
+++ b/webapp/etc/nginx/conf.d/php.conf
@@ -23,6 +23,11 @@ server {
         root /public;
         expires 1d;
     }
+    location /image/ {
+        root /public;
+        expires 1d;
+        try_files $uri $uri/ /index.php?$query_string;
+    }
     # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
     location ~ \.php {
         #fastcgi_split_path_info ^(.+\.php)(/.+)$;
~~~

nginxを再起動

~~~sh
docker compose down nginx && docker compose up nginx -d
docker compose down mysql && docker compose up mysql -d
~~~

## あとちょい

~~~sh
cat logs/nginx/access.log|alp json -m "/image/.+,/posts/[0-9]+,/@.+"
+-------+-----+-------+------+-----+-----+--------+----------------------+-------+-------+----------+-------+-------+-------+-------+--------+-----------+-------------+-----------------+------------+
| COUNT | 1XX |  2XX  | 3XX  | 4XX | 5XX | METHOD |         URI          |  MIN  |  MAX  |   SUM    |  AVG  |  P90  |  P95  |  P99  | STDDEV | MIN(BODY) |  MAX(BODY)  |    SUM(BODY)    | AVG(BODY)  |
+-------+-----+-------+------+-----+-----+--------+----------------------+-------+-------+----------+-------+-------+-------+-------+--------+-----------+-------------+-----------------+------------+
| 1     | 0   | 0     | 1    | 0   | 0   | GET    | /img/ajax-loader.gif | 0.000 | 0.000 | 0.000    | 0.000 | 0.000 | 0.000 | 0.000 | 0.000  | 0.000     | 0.000       | 0.000           | 0.000      |
| 15    | 0   | 15    | 0    | 0   | 0   | GET    | /initialize          | 0.040 | 1.028 | 3.747    | 0.250 | 0.420 | 1.028 | 1.028 | 0.241  | 5.000     | 7008.000    | 7078.000        | 471.867    |
| 242   | 0   | 0     | 242  | 0   | 0   | GET    | /logout              | 0.005 | 0.199 | 19.577   | 0.081 | 0.107 | 0.138 | 0.190 | 0.034  | 5.000     | 5.000       | 1210.000        | 5.000      |
| 484   | 0   | 484   | 0    | 0   | 0   | GET    | /login               | 0.006 | 0.194 | 39.834   | 0.082 | 0.109 | 0.146 | 0.177 | 0.036  | 1240.000  | 1240.000    | 600160.000      | 1240.000   |
| 549   | 0   | 549   | 0    | 0   | 0   | GET    | /posts               | 0.029 | 0.546 | 96.186   | 0.175 | 0.282 | 0.306 | 0.374 | 0.083  | 34470.000 | 35660.000   | 19191823.000    | 34957.783  |
| 844   | 0   | 0     | 844  | 0   | 0   | POST   | /comment             | 0.008 | 0.297 | 87.997   | 0.104 | 0.156 | 0.167 | 0.189 | 0.035  | 5.000     | 5.000       | 4220.000        | 5.000      |
| 854   | 0   | 0     | 0    | 854 | 0   | GET    | /admin/banned        | 0.006 | 0.261 | 75.590   | 0.089 | 0.132 | 0.158 | 0.182 | 0.032  | 13.000    | 13.000      | 11102.000       | 13.000     |
| 855   | 0   | 1     | 854  | 0   | 0   | POST   | /register            | 0.013 | 1.023 | 117.032  | 0.137 | 0.190 | 0.197 | 0.260 | 0.053  | 5.000     | 7008.000    | 11278.000       | 13.191     |
| 1150  | 0   | 4     | 571  | 575 | 0   | POST   | /                    | 0.006 | 0.340 | 129.371  | 0.112 | 0.179 | 0.205 | 0.256 | 0.050  | 5.000     | 297.000     | 11198.000       | 9.737      |
| 1153  | 0   | 1153  | 0    | 0   | 0   | GET    | /@.+                 | 0.024 | 0.359 | 202.945  | 0.176 | 0.250 | 0.272 | 0.310 | 0.052  | 4509.000  | 34715.000   | 21088503.000    | 18290.115  |
| 3581  | 0   | 7     | 3574 | 0   | 0   | POST   | /login               | 0.007 | 1.032 | 418.550  | 0.117 | 0.171 | 0.180 | 0.204 | 0.057  | 5.000     | 7014.000    | 66932.000       | 18.691     |
| 4516  | 0   | 4516  | 0    | 0   | 0   | GET    | /posts/[0-9]+        | 0.007 | 0.322 | 480.325  | 0.106 | 0.161 | 0.173 | 0.199 | 0.035  | 1563.000  | 6734.000    | 16306399.000    | 3610.806   |
| 4568  | 0   | 1417  | 3151 | 0   | 0   | GET    | /favicon.ico         | 0.000 | 0.000 | 0.000    | 0.000 | 0.000 | 0.000 | 0.000 | 0.000  | 0.000     | 43.000      | 60931.000       | 13.339     |
| 4569  | 0   | 1417  | 3152 | 0   | 0   | GET    | /js/timeago.min.js   | 0.000 | 0.000 | 0.000    | 0.000 | 0.000 | 0.000 | 0.000 | 0.000  | 0.000     | 1915.000    | 2713555.000     | 593.906    |
| 4569  | 0   | 1417  | 3152 | 0   | 0   | GET    | /js/main.js          | 0.000 | 0.000 | 0.000    | 0.000 | 0.000 | 0.000 | 0.000 | 0.000  | 0.000     | 1824.000    | 2584608.000     | 565.684    |
| 4569  | 0   | 1417  | 3152 | 0   | 0   | GET    | /css/style.css       | 0.000 | 0.000 | 0.000    | 0.000 | 0.000 | 0.000 | 0.000 | 0.000  | 0.000     | 1549.000    | 2194933.000     | 480.397    |
| 4864  | 0   | 4864  | 0    | 0   | 0   | GET    | /                    | 0.029 | 1.031 | 799.409  | 0.164 | 0.218 | 0.263 | 0.297 | 0.053  | 7008.000  | 35772.000   | 119105726.000   | 24487.197  |
| 40976 | 0   | 40976 | 0    | 0   | 0   | GET    | /image/.+            | 0.006 | 0.897 | 3792.685 | 0.093 | 0.152 | 0.163 | 0.185 | 0.042  | 192.000   | 1253004.000 | 12110607141.000 | 295553.669 |
+-------+-----+-------+------+-----+-----+--------+----------------------+-------+-------+----------+-------+-------+-------+-------+--------+-----------+-------------+-----------------+------------+
~~~

## maxコレクションをしぼってみる

`webapp/etc/mysql/my.cnf`

~~~sh
max_connections=10000  # <- connection の limit を更新
~~~