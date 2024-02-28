---
id: yba-tablet-splitting
summary: このハンズオンでは、簡単なTableの作成を通じて、Tabletの管理について確認します。
status: [[draft]]
authors: Tomohiro Ichimura
categories: workshop,japanese
tags: tablet,yba
feedback_link: https://yugabytedb-japan.github.io/
source: yba-tablet-splitting.md
duration: 0

---

# Tableの作成とTabletの管理

[Codelab Feedback](https://yugabytedb-japan.github.io/)


## 事前確認



* CPU Coreの数
* ノード数
* RF(Replication Factor)


## クラスタへの接続



* YBAインスタンスへのログイン

```
kubectl config set-context <CONTEXT_NAME> --kubeconfig <UNIVERSE_CONFIG_FILE>
kubectl exec -it <POD_NAME> -n <NAMESPACE> -c yugaware -- bash
```

* Nodeへのログイン(ssh)

* `Actions` &gt; `Connect`から得た情報からsshで接続
* YSQLSHの起動

* `ysqlsh`を起動

```
cd /home/yugabyte/tserver/bin
./ysqlsh -h <node_ip_address>
```


## 1. Auto Splitが有効の場合(デフォルトの挙動)



### パラメータ(gflag)の確認

YBAにてUniverseを指定 &gt; `Action` &gt; `Edit GFlag`

* `enable_automatic_tablet_splitting`

* `true`であればauto splitが有効
* `--ysql_num_shards_per_tserver`が`-1`の場合

* auto splitが有効であれば、デフォルトは1で、その後low, high, final phaseに基づいたtabletを自動的に作成
* 4CPU以下のサーバで構成されたクラスタでは、クラスタのノード数に関係せずに、2core以下であれば1 tablet, 4core以下であれば2tabletが作成される
* `--ysql_num_shards_per_tserver`が`-1`ではない場合

* auto splitが有効であれば、`--ysql_num_shards_per_tserver`で指定した数を割り当てた後、low, high, final phaseに基づいてtabletが自動的に(追加)作成される

### テーブルの作成

* レンジシャーディングとハッシュシャーディングでの違いを見てみましょう

#### レンジシャーディング

```
create table t1 ( id int, name text, primary key(id));
```

#### ハッシュシャーディング

```
create table t2 ( id int, name text, primary key(id ASC));
```

### データの投入

```
insert into t1 (id, name) select i, left(md5(random()::text),4) from generate_series(1,100000) i;
```

```
insert into t2 (id, name) select i, left(md5(random()::text),4) from generate_series(1,100000) i;
```

### データの確認

* Tableの確認(Table IDの確認)
`http://&lt;tablet_server_ip&gt;:7000/tables`
* Tableの詳細
`http://&lt;tablet_server_ip&gt;:7000/table?id=&lt;TABLEID&gt;`
* Tablet数の確認 (Table IDで検索)
`http://&lt;tablet_server_ip&gt;:9000/tablets`

### さらに追加

```
insert into t1 (id, name) select i, left(md5(random()::text),4) from generate_series(100001,200000) i;
```

```
insert into t2 (id, name) select i, left(md5(random()::text),4) from generate_series(100001,200000) i;
```


## 2. Auto Splitが無効の場合



### パラメータ(gflag)の確認

* `--ysql_num_shards_per_tserver`が`-1`の場合
* auto splitが無効のため、CPU数に応じた割り当てを実施、その後は変更されない
* 2 core 以下であれば 2 tablet, 4 core以下であれば 4 tablet, 4coreより多い場合は8 tabletとする
* `--ysql_num_shards_per_tserver`が`-1`ではない場合
* 指定した数を割り当てて固定される


## 3. SPLIT INTOによる指定 (hash sharding)



* table作成時にあらかじめ指定をする(その後はauto splitが有効であればphaseに応じて数が増える)
* `--ysql_num_shards_per_tserver`の値は上書きされる  [link](https://docs.yugabyte.com/preview/reference/configuration/yb-tserver/#ysql-num-shards-per-tserver)

```
CREATE TABLE hashdemo (
    id int ,
    name text
    PRIMARY KEY (id HASH)
) SPLIT INTO 16 TABLETS;
```

* なお、YCQLの場合は、`SPLIT WITH tablet`を使う

```
ycqlsh:example> CREATE TABLE tracking (id int PRIMARY KEY) WITH tablets = 10;
```


## 4. SPLIT AT VALUESによる指定 (range sharding)



* range shardingの場合はあらかじめ範囲を指定する必要がある

```
CREATE TABLE rangedemo(
  a int,
  b int,
  primary key(a asc, b desc)
) SPLIT AT VALUES((100), (200), (200, 5));
```


