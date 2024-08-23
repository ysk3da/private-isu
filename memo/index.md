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
pt-query-digest logs/mysql/mysqlslow.log

# No events processed.
~~~

?? 一旦おいておく

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



