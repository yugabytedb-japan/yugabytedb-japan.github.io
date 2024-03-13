---
id: yb-geo-partition-k8s
summary: このハンズオンでは、YugabyteDB Geo Partitionを使って、データに地域性を持たせた地理分散機能を確認していきます
status: [[[draft]]]
authors: Tomohiro Ichimura
categories: workshop,japanese
tags: geopartition,k8s,yba
feedback_link: https://yugabytedb-japan.github.io/
source: yb-geo-partition-k8s.md
duration: 0

---

# YugabyteDB Geo Partitionを使った地域性を持たせたデータの地理分散

[Codelab Feedback](https://yugabytedb-japan.github.io/)


## 事前準備



事前に取得して頂きたい情報は以下となります。

* k8s上で作成したuniverse(cluster)名
* ゾーンの情報


## tablespaceの作成



* お使いの環境に合わせて, `cloud`, `region`, `zone`の値を変更してください

```
CREATE TABLESPACE zone_a_tablespace WITH (
  replica_placement='{"num_replicas": 1, "placement_blocks":
  [{"cloud":"cloud1","region":"datacenter1","zone":"zone-a","min_num_replicas":1}]}'
);

CREATE TABLESPACE zone_b_tablespace WITH (
  replica_placement='{"num_replicas": 1, "placement_blocks":
  [{"cloud":"cloud1","region":"datacenter1","zone":"zone-b","min_num_replicas":1}]}'
);

CREATE TABLESPACE zone_c_tablespace WITH (
  replica_placement='{"num_replicas": 1, "placement_blocks":
  [{"cloud":"cloud1","region":"datacenter1","zone":"zone-c","min_num_replicas":1}]}'
);

```

### Tableの作成

* 親テーブルの作成

```
CREATE TABLE bank_transactions (
    user_id   INTEGER NOT NULL,
    account_id INTEGER NOT NULL,
    geo_partition VARCHAR,
    account_type VARCHAR NOT NULL,
    amount NUMERIC NOT NULL,
    txn_type VARCHAR NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
) PARTITION BY LIST (geo_partition);
```

* 子テーブルの作成とインデックスの作成(Zone A)

```
CREATE TABLE bank_transactions_a
    PARTITION OF bank_transactions
      (user_id, account_id, geo_partition, account_type,
      amount, txn_type, created_at,
      PRIMARY KEY (user_id HASH, account_id, geo_partition))
    FOR VALUES IN ('A') TABLESPACE zone_a_tablespace;

CREATE INDEX ON bank_transactions_a(account_id) TABLESPACE zone_a_tablespace;
```

* 子テーブルの作成とインデックスの作成(Zone B)

```
CREATE TABLE bank_transactions_b
    PARTITION OF bank_transactions
      (user_id, account_id, geo_partition, account_type,
      amount, txn_type, created_at,
      PRIMARY KEY (user_id HASH, account_id, geo_partition))
    FOR VALUES IN ('B') TABLESPACE zone_b_tablespace;

CREATE INDEX ON bank_transactions_b(account_id) TABLESPACE zone_b_tablespace;

```

* 子テーブルの作成とインデックスの作成(Zone C)

```
CREATE TABLE bank_transactions_c
    PARTITION OF bank_transactions
      (user_id, account_id, geo_partition, account_type,
      amount, txn_type, created_at,
      PRIMARY KEY (user_id HASH, account_id, geo_partition))
    FOR VALUES IN ('C') TABLESPACE zone_c_tablespace;


CREATE INDEX ON bank_transactions_c(account_id) TABLESPACE zone_c_tablespace;

```

* このような形になっていることを確認します

```
yugabyte=# \d
                List of relations
 Schema |        Name         | Type  |  Owner
--------+---------------------+-------+----------
 public | bank_transactions   | table | yugabyte
 public | bank_transactions_a | table | yugabyte
 public | bank_transactions_b | table | yugabyte
 public | bank_transactions_c | table | yugabyte
(4 rows)
```


## データの投入



```
INSERT INTO bank_transactions
    VALUES (100, 10001, 'A', 'checking', 120.50, 'debit');
```

* データの確認

```
yugabyte=# select * from bank_transactions;
 user_id | account_id | geo_partition | account_type | amount | txn_type |        created_at
---------+------------+---------------+--------------+--------+----------+---------------------------
     100 |      10001 | A             | checking     |  120.5 | debit    | 2024-03-13 07:54:21.06421
(1 row)

yugabyte=# select * from bank_transactions_a;
 user_id | account_id | geo_partition | account_type | amount | txn_type |        created_at
---------+------------+---------------+--------------+--------+----------+---------------------------
     100 |      10001 | A             | checking     |  120.5 | debit    | 2024-03-13 07:54:21.06421
(1 row)

yugabyte=# select * from bank_transactions_b;
 user_id | account_id | geo_partition | account_type | amount | txn_type | created_at
---------+------------+---------------+--------------+--------+----------+------------
(0 rows)

yugabyte=# select * from bank_transactions_c;
 user_id | account_id | geo_partition | account_type | amount | txn_type | created_at
---------+------------+---------------+--------------+--------+----------+------------
(0 rows)

```

* 同様に他の地域にもデータを投入してみましょう。

```
INSERT INTO bank_transactions
    VALUES (200, 20001, 'B', 'savings', 1000, 'credit');
INSERT INTO bank_transactions
    VALUES (300, 30001, 'C', 'checking', 105.25, 'debit');
```


## 地域性を考慮したクエリ



```
yugabyte=# select * from bank_transactions where geo_partition='A';
 user_id | account_id | geo_partition | account_type | amount | txn_type |        created_at
---------+------------+---------------+--------------+--------+----------+---------------------------
     100 |      10001 | A             | checking     |  120.5 | debit    | 2024-03-13 07:54:21.06421
(1 row)

```

* (参考) regionが異なる場合は、カラムを指定せずに以下を実施 (今回はzone毎に設定のため、全てのエントリが返ります)

```
yugabyte=# select * from bank_transactions where yb_is_local_table(tableoid);
 user_id | account_id | geo_partition | account_type | amount | txn_type |         created_at
---------+------------+---------------+--------------+--------+----------+----------------------------
     100 |      10001 | A             | checking     |  120.5 | debit    | 2024-03-13 07:54:21.06421
(1 rows)
```


## 地域を移動しながらの操作



* 移動しながら操作している同一のユーザを想定して、異なる地域に対してデータを投入してみましょう。

```
INSERT INTO bank_transactions
    VALUES (100, 10001, 'B', 'savings', 2000, 'credit');
INSERT INTO bank_transactions
    VALUES (100, 10001, 'C', 'checking', 105, 'debit');
```

* 結果

```
yugabyte=# select * from bank_transactions_B where user_id=100;
 user_id | account_id | geo_partition | account_type | amount | txn_type |         created_at
---------+------------+---------------+--------------+--------+----------+----------------------------
     100 |      10001 | B             | savings      |   2000 | credit   | 2024-03-13 08:09:08.775121
(1 row)

yugabyte=# select * from bank_transactions_C where user_id=100;
 user_id | account_id | geo_partition | account_type | amount | txn_type |         created_at
---------+------------+---------------+--------------+--------+----------+----------------------------
     100 |      10001 | C             | checking     |    105 | debit    | 2024-03-13 08:09:08.796655
(1 row)

yugabyte=# select * from bank_transactions where user_id=100 order by created_at desc;
 user_id | account_id | geo_partition | account_type | amount | txn_type |         created_at
---------+------------+---------------+--------------+--------+----------+----------------------------
     100 |      10001 | C             | checking     |    105 | debit    | 2024-03-13 08:09:08.796655
     100 |      10001 | B             | savings      |   2000 | credit   | 2024-03-13 08:09:08.775121
     100 |      10001 | A             | checking     |  120.5 | debit    | 2024-03-13 07:54:21.06421
(3 rows)

```

> aside negative
> 
> **Note:** 指定したtopologyがない場合は、nodeで指定したサーバにアクセスして`load_balance`のオプションに従って処理を行います

Reference

[Row-level geo-partitioning](https://docs.yugabyte.com/stable/explore/multi-region-deployments/row-level-geo-partitioning/)

[Geo Partition Helper Function](https://docs.yugabyte.com/preview/api/ysql/exprs/geo_partitioning_helper_functions/)


