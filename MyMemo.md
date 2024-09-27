# 2024/09/27 社内 ISUCON 2.5

## 事前準備編

だいぶ忘れていた

php に変更するところからスタート

ベンチマーカーを回したら、初期状態じゃなかった

```json
{ "pass": true, "score": 9216, "success": 8219, "fail": 0, "messages": [] }
```

sql に index はったままだったので DL しなおした

なぜか解凍コマンドが書いてないのでしらべた

```sh
bunzip2 dump.sql.bz2
```

だめでした。orz

DB を差し替えることではリセットされないので削除するコマンドを探す

https://qiita.com/pugiemonn/items/8a6b322654aa65e2966b

```sh
ALTER TABLE テーブル名 DROP INDEX インデックス名;
```

てことで

```sh
ALTER TABLE comments DROP INDEX post_id_idx;
```

改めてベンチマーカーを回す

```json
{
  "pass": true,
  "score": 0,
  "success": 204,
  "fail": 43,
  "messages": [
    "リクエストがタイムアウトしました (GET /)",
    "リクエストがタイムアウトしました (GET /image/1137.jpg)",
    "リクエストがタイムアウトしました (GET /image/1349.jpg)",
    "リクエストがタイムアウトしました (GET /image/5719.jpg)",
    "リクエストがタイムアウトしました (GET /image/9504.jpg)",
    "リクエストがタイムアウトしました (GET /image/9989.jpg)",
    "リクエストがタイムアウトしました (GET /image/9995.jpg)",
    "リクエストがタイムアウトしました (POST /login)",
    "リクエストがタイムアウトしました (POST /register)"
  ]
}
```

はいOKです。