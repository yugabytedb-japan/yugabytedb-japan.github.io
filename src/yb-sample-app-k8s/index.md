---
id: yb-sample-app-k8s
summary: このハンズオンでは、YugabyteDB Smart Driverを使って、ロードバランス機能を確認していきます
status: [[[draft]]]
authors: Tomohiro Ichimura
categories: workshop,japanese
tags: java,k8s,smartdriver,yba
feedback_link: https://yugabytedb-japan.github.io/
source: yb-sample-app-k8s.md
duration: 0

---

# Smart Driver on k8s

[Codelab Feedback](https://yugabytedb-japan.github.io/)


## 事前準備



事前に取得して頂きたい情報は以下となります。

* k8s上で作成したuniverse(cluster)名
* ゾーンの情報
* namespace名


## [Option] Clusterの準備　（ OSSの場合 )



helmでは以下のようにデプロイをします。

```language-shell
% helm install yb-demo yugabytedb/yugabyte \
--version 2.19.3 \
--set resource.master.requests.cpu=0.5,resource.master.requests.memory=0.5Gi,\
resource.tserver.requests.cpu=0.5,resource.tserver.requests.memory=0.5Gi,\
replicas.master=1,replicas.tserver=1 --namespace hol2
NAME: yb-demo
LAST DEPLOYED: Wed Mar  6 04:39:21 2024
NAMESPACE: hol2
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Get YugabyteDB Pods by running this command:
  kubectl --namespace hol2 get pods

2. Get list of YugabyteDB services that are running:
  kubectl --namespace hol2 get services

3. Get information about the load balancer services:
  kubectl get svc --namespace hol2

4. Connect to one of the tablet server:
  kubectl exec --namespace hol2 -it yb-tserver-0 bash

5. Run YSQL shell from inside of a tablet server:
  kubectl exec --namespace hol2 -it yb-tserver-0 -- /home/yugabyte/bin/ysqlsh -h yb-tserver-0.yb-tservers.hol2

6. Cleanup YugabyteDB Pods
  For helm 2:
  helm delete yb-demo --purge
  For helm 3:
  helm delete yb-demo -n hol2
  NOTE: You need to manually delete the persistent volume
  kubectl delete pvc --namespace hol2 -l app=yb-master
  kubectl delete pvc --namespace hol2 -l app=yb-tserver
```


## サンプルアプリを含んだDockerイメージの起動



* namespaceに対して、サンプルイメージを実行します ( namespaceのオプション(`-n`)は環境に合わせて変えてください)

```language-bash
kubectl run yb-sample-apps -it --rm  --image yugabytedb/yb-sample-apps -n yb-demo --command -- sh
```

* 今回については、最新のイメージとして [Yugabyte SE](https://hub.docker.com/r/yogendra/yb-sample-apps/tags)のものを利用します。

```language-bash
kubectl run yb-sample-apps -it --rm  --image yogendra/yb-sample-apps -n hol2yb-demo --command -- sh
```


## サンプルアプリの実行



* 今回利用するアプリのソースは [こちら](https://github.com/yugabyte/yb-sample-apps)
* 実行方法は以下の通り(1分程度に収まるように`num_reads`, `num_writes`を調整してます)

```language-shell
% java -jar yb-sample-apps.jar \
   --workload SqlInserts \
   --nodes <tserver-service address>:5433 \
   --num_threads_write 4 \
   --num_threads_read 6 \
   --num_unique_keys 200000 \
   --num_reads 150000  \
   --num_writes 200000
```

> aside negative
> 
> **Note:** `&lt;tserver-service address&gt;` は `yb-tserver-0.yb-tservers.hol2.svc.cluster.local`のようなFQDN、あるいはIPアドレスを指定します

実行結果例:

```language-shell
# java -jar yb-sample-apps.jar --workload SqlInserts --nodes 10.100.139.243:5433 --num_threads_write 4 --num_threads_read 6 --num_unique_keys 200000 --num_reads 150000  --num_writes 200000
0 [main] INFO com.yugabyte.sample.Main  - Starting sample app...
34 [main] INFO com.yugabyte.sample.common.CmdLineOpts  - Using a randomly generated UUID : 35ca917b-bff2-4a65-b1a6-fd7c1b9e624d
39 [main] INFO com.yugabyte.sample.common.CmdLineOpts  - App: SqlInserts
39 [main] INFO com.yugabyte.sample.common.CmdLineOpts  - Run time (seconds): -1
39 [main] INFO com.yugabyte.sample.common.CmdLineOpts  - Adding node: 10.100.139.243:5433
39 [main] INFO com.yugabyte.sample.common.CmdLineOpts  - Num reader threads: 6, num writer threads: 4
39 [main] INFO com.yugabyte.sample.common.CmdLineOpts  - Num unique keys to insert: 200000
40 [main] INFO com.yugabyte.sample.common.CmdLineOpts  - Num keys to update: 0
40 [main] INFO com.yugabyte.sample.common.CmdLineOpts  - Num keys to read: 150000
40 [main] INFO com.yugabyte.sample.common.CmdLineOpts  - Value size: 0
40 [main] INFO com.yugabyte.sample.common.CmdLineOpts  - Restrict values to ASCII strings: false
40 [main] INFO com.yugabyte.sample.common.CmdLineOpts  - Perform sanity check at end of app run: false
40 [main] INFO com.yugabyte.sample.common.CmdLineOpts  - Table TTL (secs): -1
40 [main] INFO com.yugabyte.sample.common.CmdLineOpts  - Local reads: false
40 [main] INFO com.yugabyte.sample.common.CmdLineOpts  - Read only load: false
42 [main] INFO com.yugabyte.sample.common.CmdLineOpts  - SqlInserts workload: using driver set for load-balance = true
224 [main] INFO com.yugabyte.sample.apps.SqlInserts  - Created table: postgresqlkeyvalue
5237 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 2251.20 ops/sec (2.48 ms/op), 11281 total ops  |  Write: 1768.57 ops/sec (2.22 ms/op), 8845 total ops  |  Uptime: 5013 ms |
10238 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 2676.93 ops/sec (2.24 ms/op), 24669 total ops  |  Write: 1489.01 ops/sec (2.69 ms/op), 16291 total ops  |  Uptime: 10014 ms |
15239 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 2704.13 ops/sec (2.22 ms/op), 38191 total ops  |  Write: 1477.66 ops/sec (2.70 ms/op), 23680 total ops  |  Uptime: 15015 ms |
20239 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 2687.15 ops/sec (2.23 ms/op), 51628 total ops  |  Write: 1479.45 ops/sec (2.70 ms/op), 31078 total ops  |  Uptime: 20015 ms |
25240 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 2661.54 ops/sec (2.25 ms/op), 64937 total ops  |  Write: 1500.26 ops/sec (2.67 ms/op), 38580 total ops  |  Uptime: 25015 ms |
30240 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 2657.95 ops/sec (2.26 ms/op), 78228 total ops  |  Write: 1514.26 ops/sec (2.64 ms/op), 46152 total ops  |  Uptime: 30016 ms |
35240 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 2651.55 ops/sec (2.26 ms/op), 91487 total ops  |  Write: 1514.44 ops/sec (2.64 ms/op), 53725 total ops  |  Uptime: 35016 ms |
40241 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 2702.13 ops/sec (2.22 ms/op), 104999 total ops  |  Write: 1478.26 ops/sec (2.71 ms/op), 61117 total ops  |  Uptime: 40017 ms |
45241 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 2667.93 ops/sec (2.25 ms/op), 118340 total ops  |  Write: 1496.86 ops/sec (2.67 ms/op), 68602 total ops  |  Uptime: 45017 ms |
50242 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 2683.36 ops/sec (2.24 ms/op), 131758 total ops  |  Write: 1453.87 ops/sec (2.75 ms/op), 75872 total ops  |  Uptime: 50018 ms |
55242 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 2705.76 ops/sec (2.22 ms/op), 145288 total ops  |  Write: 1476.87 ops/sec (2.71 ms/op), 83257 total ops  |  Uptime: 55018 ms |
60243 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 943.31 ops/sec (2.28 ms/op), 150005 total ops  |  Write: 3144.89 ops/sec (1.28 ms/op), 98983 total ops  |  Uptime: 60019 ms |
65243 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 0.00 ops/sec (0.00 ms/op), 150005 total ops  |  Write: 4083.43 ops/sec (0.98 ms/op), 119402 total ops  |  Uptime: 65019 ms |
70244 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 0.00 ops/sec (0.00 ms/op), 150005 total ops  |  Write: 4078.59 ops/sec (0.98 ms/op), 139797 total ops  |  Uptime: 70020 ms |
75244 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 0.00 ops/sec (0.00 ms/op), 150005 total ops  |  Write: 4045.22 ops/sec (0.99 ms/op), 160025 total ops  |  Uptime: 75020 ms |
80245 [Thread-0] INFO com.yugabyte.sample.common.metrics.MetricsTracker  - Read: 0.00 ops/sec (0.00 ms/op), 150005 total ops  |  Write: 4054.26 ops/sec (0.99 ms/op), 180298 total ops  |  Uptime: 80021 ms |
85121 [main] INFO com.yugabyte.sample.Main  - The sample app has finished
```

### podの状態の確認

```language-shell
% kubectl --namespace hol2 get pods
NAME                          READY   STATUS    RESTARTS       AGE
nginx-test-6cbcbd4574-w5l8x   1/1     Running   0              6d19h
yb-master-0                   2/2     Running   0              140m
yb-sample-apps                1/1     Running   0              76m
yb-tserver-0                  2/2     Running   1 (114m ago)   140m
```

### Load Balancerをオフにする

```language-shell
java -jar yb-sample-apps.jar --workload SqlInserts --nodes 10.100.139.243:5433 --load_balance false --num_threads_write 4 --num_threads_read 6 --num_reads 150000  --num_writes 200000
```


## Topology Aware



### 対象となるplacementを選択

* `"kubernetes.ap-northeast-1.ap-northeast-1a,kubernetes.ap-northeast-1.ap-northeast-1c,kubernetes.ap-northeast-1.ap-northeast-1d"`

```
java -jar yb-sample-apps.jar --workload SqlInserts --nodes 10.100.139.243:5433 --topology_keys "kubernetes.ap-northeast-1.ap-northeast-1a,kubernetes.ap-northeast-1.ap-northeast-1c,kubernetes.ap-northeast-1.ap-northeast-1d" --num_threads_write 4 --num_threads_read 6 --num_unique_keys 200000 --num_reads 150000  --num_writes 200000
```


## Topology Aware with Fallback



### 対象となるplacementを選択、fallbackの順序も指定

* 1~10のレンジで設定が可能。1が起動時に選択されるplacementで、以降2, 3,...と続く
* 例: `"kubernetes.ap-northeast-1.ap-northeast-1a:1,kubernetes.ap-northeast-1.ap-northeast-1c:2,kubernetes.ap-northeast-1.ap-northeast-1d:3"`

### 1. load_balance と topology_keys にfallbackオプション(`:1`)を指定してワークロードを実行

```
java -jar yb-sample-apps.jar --workload SqlInserts --nodes <YOUR NODE>:5433 --topology_keys "kubernetes.ap-northeast-1.ap-northeast-1a:1,kubernetes.ap-northeast-1.ap-northeast-1c:2,kubernetes.ap-northeast-1.ap-northeast-1d:3" --num_threads_write 4 --num_threads_read 6 --num_unique_keys 200000 --num_reads 150000  --num_writes 200000
```

### 2. 状況の確認

いずれかの方法にて、fallback指定したサーバ(`:1`で指定したサーバ)のみにアクセスが発生していることを確認してください

* YBA dashboard
* Tserver dashboard (master ui:7000)

### 3. node(pod)の停止

```
kubectl delete pod yb-tserver-0 -n YOUR_NAMESPACE --force
```

### 4. 状態の確認

アプリからは`FATAL: Could not reconnect to database`のようなメッセージが表れますが、動作は継続します。 これは、指定したサーバにfallbackされている、あるいはfallbackが指定されてなければ、接続可能なサーバに接続しているためです。 `Step 2.` と同様にダッシュボードで確認をしてみてください。

### 5. 停止

アプリを停止してください

### 6. [Option] fallbackの指定方法

もし`topology_keys`にてfallbackオプションを指定しない場合で、最初のtserverを停止させた場合、残りのtserverにアクセスが発生していることが確認できます。

```
--topology_keys "aws.us-east.us-east-1a"
```

Step 2,3,4と同様に確認してみてください。

> aside negative
> 
> **Note:** 指定したtopologyがない場合は、nodeで指定したサーバにアクセスして`load_balance`のオプションに従って処理を行います

Reference

[YugabyteDB Smart Driver](https://docs.yugabyte.com/preview/drivers-orms/smart-drivers/) [YugabyteDB workload generator](https://github.com/yugabyte/yb-sample-apps)


