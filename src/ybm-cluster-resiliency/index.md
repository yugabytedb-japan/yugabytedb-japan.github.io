---
id: ybm-cluster-resiliency
summary: このハンズオンでは、ワークロード・シミュレーション・ツールを使用してYugabyteDB Managedのクラスターでノード停止をシミュレートし、クラスターが処理を継続できることを確認します。
status: [draft]
authors: Arisa Izuno
categories: ybm
tags: ybm
feedback_link: https://yugabytedb-japan.github.io/
source: 10PpyYJ3OUVuqqPBFpUOBWvrfn_GD1ueQLNJvTbYK8c4
duration: 70

---

# YugabyteDB Managed の耐障害性と拡張性

[Codelab Feedback](https://yugabytedb-japan.github.io/)


## はじめに
Duration: 01:00


**Last Updated:** 2022-01-20

### **分散SQLデータベースの高可用性と耐障害性**

分散SQLデータベースであるYugabyteDBは、拡張性（水平スケール）や高可用性を前提とした設計となっており、デフォルトで高い耐障害性を備えています。スケール時にもサービス停止は発生せず、ローリング・アップグレードといったゼロダウンタイムでのオペレーションが可能です。

このハンズオンでは、YugabyteDB Managedを使用して、ノード障害やクラスタのスケール時にもダウンタイムが発生せず、クラスタが処理を継続できることを確認します。

### **ハンズオンで実施すること**

このハンズオンでは、YugabyteDB Managedでクラスタを構成し、障害発生や構成変更の際にもサービス停止が発生しないことを確認します。以下の内容を実施します:

* YugabyteDB Managedへのサインアップ
* 3ノード・クラスタの構成
* ワークロード・シミュレータを使用した、テーブルの作成、データの挿入
* ワークロード・シミュレータを使用した、ノード停止やクラスタのスケール

<img src="img/6d4c704f9de3e8fe.png" alt="6d4c704f9de3e8fe.png"  width="624.00" />

### **ハンズオンで学習すること**

* YugabyteDB Managedのコンソール操作
* YugabyteDB Managedでのクラスタ作成
* YugabyteDB ManagedのREST API
* YugabyteDB Managedクラスタの耐障害性とスケーラビリティ

### **ハンズオン実施に必要なもの**

* YugabyteDB Managedアカウント
* Java 19
ワークロードシミュレータを実行するために必要です。Homebrewを使用している場合は、ターミナルから `brew install openjdk` を入力して最新のJava環境をインストールしてください。


## YugabyteDB Managedへのサインアップ
Duration: 03:00


YugabyteDB ManagedはフルマネージドのDBaaS (データベース・アズ・サービス）です。サインアップしてアカウントを作成することで、すぐにデータベースを使い始めることができます。初めてYugabyteDB Managedを使用する方は、以下の手順に従ってアカウントを作成してください。

1. ブラウザで [こちら](https://cloud.yugabyte.com/signup)にアクセスし、アカウントを作成してください。

<img src="img/5768cf0371033212.png" alt="5768cf0371033212.png"  width="624.00" />

2. 入力したEメールアドレス宛に、確認のメールが届きます。**[Verify Email]** ボタンをクリックしてください。

<img src="img/99ef95b51b691889.png" alt="99ef95b51b691889.png"  width="389.50" />

3. ログイン画面にリダイレクトされます。設定したユーザーIDとパスワードでログインしてください。ログインが成功したら以下のような画面が表示されます。

<img src="img/89ae48e00d977444.png" alt="89ae48e00d977444.png"  width="624.00" />

以上で、YugabyteDB Managedへのサインアップは完了です。


## トポロジーの選択
Duration: 01:00


YugabyteDB Managedでは、UIの項目選択によってクラスタ構成やノード仕様を設定することができます。

以下のような項目を設定します：

* クラウド・ベンダー（AWS, GCP）
* データベースのバージョン（安定版、プレビュー版）
* シングル・リージョンかマルチリージョンか
* 耐障害性のレベル
* なし
* ノードレベル
* AZレベル
* データの分散方法（マルチリージョン・クラスタのみ）
* マルチリージョン・クラスタ
* ジオ・パーティショニング

<img src="img/528e21d80ef87331.png" alt="528e21d80ef87331.png"  width="624.00" />

このハンズオンでは、シングル・リージョンにAZレベルの耐障害性をもつクラスタ（上の図の真ん中）を作成します。


## 3ノードクラスタの作成
Duration: 15:00


ここではGCP東京リージョンの各アベイラビリティ・ゾーンにノードを配置して、ゾーンレベルの耐障害性をもつ3ノードクラスタを構成します。

1. YugabyteDB Managedのアカウントにログインします。
2. 左側のメニューから**[Clusters]**を選択し、**[Add Cluster]**ボタンをクリックしてください。
3. クラスタ作成のウィザードが開始します。右側のDedicatedにある **[Request a Free Trial]**ボタンをクリックしてください。

<img src="img/d0b7b52b487b3608.png" alt="d0b7b52b487b3608.png"  width="624.00" />

4. トライアル申し込みのウィンドウが表示されます。Eメールアドレス、電話番号、希望する連絡先を選択して、**[Send Request]** ボタンをクリックしてください。

<img src="img/51e3c9eb2941f5d8.png" alt="51e3c9eb2941f5d8.png"  width="449.50" />

5. General settingsページが表示されます。クラスタの名前には適当な名前が自動生成されます。[GCP]を選択し、安定版 [Stable Release] が選択されていることを確認して [Next] をクリックしてください。

<img src="img/5c332ecd267633c7.png" alt="5c332ecd267633c7.png"  width="624.00" />

6. Cluster setupページが表示されます。ページ上部にある [Single-Region Deployment] を選択します。1の耐障害性レベルには [Availability Zone Level]、2のリージョンには [Tokyo] を選択してください。 クラスタの仕様は、vCPUを最小の [2] に設定します。vCPUのサイズを変更すると、メモリのとディスクのサイズは自動的に変更されます。

<img src="img/abd04254394a2518.png" alt="abd04254394a2518.png"  width="624.00" />

7. Cluster setupページの下部には、VPCの設定を行う箇所があります。このハンズオンでは使用しませんので、[Use VPC Peering] をオフにしたまま、[Next]をクリックしてください。

<img src="img/33c14465437cc765.png" alt="33c14465437cc765.png"  width="624.00" />

8. Network Accessの設定ページが表示されます。[Add Current IP Address]をクリックして、自分の端末のIPアドレスをアクセス許可リストに追加してください。

<img src="img/736eef68579e8061.png" alt="736eef68579e8061.png"  width="624.00" />

9. [Next]をクリックします。
10. DB Credentialsページが表示されます。ユーザー名とパスワードは自動設定されます。設定をカスタマイズしたい場合は、[Add your own credentials]をクリックしてユーザー名をパスワードを自分で設定します。このままで問題なければ [Download credentials]ボタンをクリックして、アクセス情報のファイルをローカルに保存してください。

<img src="img/e4e6075adfd218ec.png" alt="e4e6075adfd218ec.png"  width="624.00" />

> aside negative
> 
> **Note:** このアクセス情報は、ハンズオンの後のステップで使用します。ファイルを保存した場所を忘れないようにしてください。

11. [Create Cluster]ボタンをクリックします。プロビジョニングが開始され、DBクラスタが開始するまでに数分かかります。

<img src="img/2b9869e47b2b81ff.png" alt="2b9869e47b2b81ff.png"  width="624.00" />

12. 起動が完了するとクラスターのダッシュボードが表示されます。

<img src="img/19b1e4407f407ae2.png" alt="19b1e4407f407ae2.png"  width="624.00" />

以上で、クラスタの作成は完了です。


## クラスタのアクセス情報の確認
Duration: 05:00


クラスタにアクセスするには、いくつかの情報やファイルが必要です。このセクションでは、以下の情報を確認します。

* データベースのユーザー名とパスワード（前のステップで作成したもの）
* 作成したクラスタのID
* YugabyteDB ManagedのアカウントID、プロジェクトID
* YugabyteDB Managed REST APIを使用するためのAPI Key

1. 前のステップで作成したクラスタのダッシュボードを開きます。
2. クラスタのダッシュボードの右上にある、[Coneect]ボタンをクリックしてください。
3. クラスタに接続するための方法が複数表示されます。一番下にある[Connect to your Application]の右向き矢印をクリックします。

<img src="img/33908e1aa021ef64.png" alt="33908e1aa021ef64.png"  width="573.80" />

4. SSL通信のための証明書をダウンロードします。[Download CA  Cert]のリンクをクリックして、証明書ファイル (root.crt) をダウンロードしてください。

<img src="img/28a00b2ab527e454.png" alt="28a00b2ab527e454.png"  width="577.04" />

> aside negative
> 
> **Note:** この証明書は、ハンズオンの後のステップで使用します。ファイルを保存した場所を忘れないようにしてください。

5. [2)Use the following parameters to connect to your cluster]セクションの下で、YSQLの[Parameters]タブを選択します。[Host] の横にあるコピーボタンをクリックしてメモしてください。ホスト名は、&lt;Region&gt;.&lt;ClusterID&gt;.&lt;Cloud&gt;.ybdb.io で設定されています。最初の.(ドット) の後の文字列(上の例では、4e095914-fb49-4b71-8e3a-3dee1073f221)が、クラスターIDです。

<img src="img/217baee793e08093.png" alt="217baee793e08093.png"  width="624.00" />

6. 右上にあるXをクリックして、[Connect to Cluster]のウィンドウを閉じます。
7. 左側のメニューから、[Admin]を選択します。[API Keys]タブを選択して、[Create API Key] ボタンをクリックしてください。

<img src="img/1b0472942539617d.png" alt="1b0472942539617d.png"  width="624.00" />

8. API Keyの名前に任意の名前を入力します。Roleは[Admin]を選択し、[Generate Key]ボタンをクリックしてください。

<img src="img/69ac166c8d48dfb2.png" alt="69ac166c8d48dfb2.png"  width="624.00" />

9. APIキーが表示されます。キーはこの画面でしか表示されないので、必ずキーをコピーしてメモしてください。
10. [View Details] セクションを開くと、アカウントIDとプロジェクトIDが表示されます。この2つもコピーして、メモしておいてください。

<img src="img/e896dc9083e67328.png" alt="e896dc9083e67328.png"  width="624.00" />

これで、クラスタへのアクセスに必要な情報が確認できました。


## シミュレータのセットアップ
Duration: 05:00


YB Workload Simulatorは、アプリケーションから見たレイテンシやスループットをシミュレートするJavaアプリケーションで、YugabyteDBのコードサンプルとして [GitHub](https://github.com/YugabyteDB-Samples/yb-workload-simulator)で公開されています。

シミュレータを使用することで、以下のことを実施できます。

* ワークロード実行のためのテーブルを作成/ドロップ
* ワークロード実行のためのデータをロード
* TPSやJavaのスレッド数を調整してワークロードを実行
* クラスタのトポロジーを可視化
* YugabyteDB Managedのオペレーション（ノード停止/再開、クラスタのスケール）

> aside negative
> 
> **Note:** シミュレータの実行にはJava 19が必要です。コマンドツール(Terminal等) でjava -versionと入力し、Javaのインストール・バージョンを確認してください。

1.  [リリース](https://github.com/YugabyteDB-Samples/yb-workload-simulator/releases)ページにアクセスして、最新のシミュレータのjarファイルをダウンロードしてください。
2. 前のステップでダウンロードしたクラスタのアクセス情報、証明書、アカウントIDやプロジェクトID、API Keyを確認して、以下のコマンドを入力します。

```
java -Dnode=<host name> \
     -Ddbname=<dbname> \
     -Ddbuser=<dbuser> \
     -Ddbpassword=<dbpassword> \
     -Dssl=true \
     -Dsslmode=verify-full \
     -Dsslrootcert=<path-to-cluster-certificate> \
     -Dybm-api-key=<ybm-api-key>
     -Dybm-account-id=<ybm-account-id> \
     -Dybm-project-id=<ybm-project-id> \
     -Dybm-cluster-id=<ybm-cluster-id> \
     -jar ./yb-workload-sim-0.0.2.jar
```

| &lt;host name&gt; | クラスタの接続情報で確認したホストのアドレス(例)asia-northeast1.4e095914-fb49-4b71-8e3a-3dee1073f221.gcp.ybdb.io |
| --- | --- |
| &lt;dbname&gt; | yugabyte |
| &lt;dbuser&gt; | クラスタのアクセス情報(&lt;cluster_name&gt;.txt)ファイルに記載(例) admin |
| &lt;dbpassword&gt; | クラスタのアクセス情報(&lt;cluster_name&gt;.txt)ファイルに記載 |
| &lt;path-to-cluster-certificate&gt; | クラスタの接続情報でダウンロードしたroot.crtファイルへのパス |
| &lt;ybm-api-key&gt; | Adminメニューで生成したAPI Key |
| &lt;ybm-account-id&gt; | API Key生成時に確認したアカウントID。管理コンソール右上のユーザーアイコンからも確認可能 |
| &lt;ybm_project-id&gt; | API Key生成時に確認したプロジェクトID。管理コンソール右上のユーザーアイコンからも確認可能 |
| &lt;ybm-cluster-id&gt; | ホスト名のリージョンの後にある、クラスタのUUID(例) 4e095914-fb49-4b71-8e3a-3dee1073f221 |

3. シミュレータが実行されると、以下のようなアスキーアートが表示されます。

<img src="img/3e7520515eb84f2a.png" alt="3e7520515eb84f2a.png"  width="624.00" />

4. ブラウザを開いて、 [http://localhost:8080/](http://localhost:8080/) にアクセスしてください。シミュレータが表示されます。

<img src="img/ccda210ca3e2b94d.png" alt="ccda210ca3e2b94d.png"  width="624.00" />

5. [Network Diagram and Aggregate Statistics]ウィンドウの右上にある、歯車アイコンをクリックし、[Yugabyte Options] を選択してください。

<img src="img/4ba23d70bf9cf771.png" alt="4ba23d70bf9cf771.png"  width="260.00" />

6. パスワードの設定を要求する画面が表示されます。任意のパスワードを設定し、メモしておいてください。[OK]ボタンをクリックして、一度ウィンドウを閉じます。
7. 再度、歯車アイコンをクリックして[Yugabyte Options]を選択し、[Database in Use]に[Yugabyte Managed]を選択します。パスワードの入力を求められたら、先ほどのパスワードを入力し[Validate Password]をクリックしてください。

<img src="img/4d724c5bc4b15374.png" alt="4d724c5bc4b15374.png"  width="624.00" />

8. API Access KeyやアカウントIDを入力する画面が表示されます。メモしておいた内容を入力し、[OK] ボタンをクリックしてください。

<img src="img/1c17dde957fc6101.png" alt="1c17dde957fc6101.png"  width="624.00" />

9. 設定が保存されると、ネットワーク図の下にクラスタを操作するボタンが表示されます。

<img src="img/8ccd49293c102497.png" alt="8ccd49293c102497.png"  width="624.00" />


## テーブル作成とデータのロード
Duration: 05:00


ワークロード・シミュレータでは、シミュレーションを実行するためのテーブルやデータを生成する機能を提供しています。

1. シミュレータの左上にある、ハンバーガーアイコンをクリックしてください。

<img src="img/81db91963cb9115c.png" alt="81db91963cb9115c.png"  width="274.00" />

2. [Workload Management for Generic]ウィンドウが開きます。[Usable Operations]タブを表示してください。

<img src="img/f019badb3a989f1a.png" alt="f019badb3a989f1a.png"  width="624.00" />

3. [Create Tables]セクションを開き、 [Run Create Tables Workload]ボタンをクリックしてください。"Workload CREATE_TABLES successfully submitted"のメッセージがウィンドウ下部に表示されます。
4. YugabyteDB Managedのコンソールで、クラスタのダッシュボードの[Tables]タブを表示すると、3つのテーブルが作成されていることがわかります。

<img src="img/70ab7f49edf26f53.png" alt="70ab7f49edf26f53.png"  width="624.00" />

5. シミュレータ画面に戻り、[Workload Management for Generic]ウィンドウを開きます。
6. [Seed Data]セクションを開き、[Run Seed Data Workload]ボタンをクリックしてください。"Workload SEED_DATA successfully submitted."のメッセージがウィンドウ下部に表示されます。
7. YugabyteDB Managedのコンソールで、実際にデータがロードされたことを確認してみましょう。クラスタのダッシュボード右上にある[Connect]ボタンをクリックし、表示されるウィンドウで[Launch Cloud Shell]をクリックします。

<img src="img/731b0fefd345f245.png" alt="731b0fefd345f245.png"  width="624.00" />

8. データベース名やユーザー名を確認し、[Confirm]ボタンをクリックしてください。

<img src="img/c51e511aac00fb57.png" alt="c51e511aac00fb57.png"  width="600.00" />

9. 新しいブラウザ・タブが開き、クラスタにアクセスするためのシェルが表示されます。クラスタアクセス情報のファイルに記載された、パスワードを入力してください。

<img src="img/cff23d35348c2544.png" alt="cff23d35348c2544.png"  width="624.00" />

10. パスワードが認証されると、YSQLの入力モードになります。`SELECT count(*) from generic1;`と入力し、エンターキーを押してください。1000行のデータがロードされていることがわかります。

<img src="img/44c1c6b61012da6a.png" alt="44c1c6b61012da6a.png"  width="624.00" />

11. 続いて、データの一部を参照するため、`SELECT * from generic1 limit 10;`と入力してください。ロードされたデータの内容を確認したら、`exit;`と入力してシェルを終了し、タブを閉じます。 <img src="img/cff453010c653faf.png" alt="cff453010c653faf.png"  width="624.00" />


## ワークロードの実行
Duration: 05:00


シミュレータで作成したテーブルとデータを使用して、YugabyteDB Managedのクラスタに読み取り/書き込みのワークロードを実行します。

1. ワークロード・シミュレータのブラウザ・タブを開き、左上のハンバーガーアイコンから[Workload Management for Generic]ウィンドウを表示してください。
2. [Simulation]セクションで、スループットとスレッドの設定を確認します。読み取りだけでなく、書き込みのワークロードも生成するため、[Include new Inserts]のボタンをオンにして、[Run Simulation Workload]ボタンをクリックしてください。

<img src="img/601fa97c8854aae0.png" alt="601fa97c8854aae0.png"  width="624.00" />

3. "Workload RUN_SIMULATION successfully submitted."のメッセージがウィンドウ下部に表示されることを確認し、[Close]ボタンをクリックしてウィンドウを閉じてください。
4. レイテンシーとスループットのグラフを確認してください。ワークロードが継続的に生成され、データベースがサービスを継続していることがわかります。

<img src="img/45636d4b386c295.png" alt="45636d4b386c295.png"  width="624.00" />


## AZ障害のシミュレート
Duration: 10:00


このハンズオンの前のステップで、複数のアベイラビリティゾーン(AZ)にノードを配置したクラスタを構成しました。そのため、AZレベルの障害であれば、クラスタは影響を受けずにサービスを継続することができる耐障害性を備えています。

このセクションでは、クラスタを構成する一つのノードを停止することでAZレベル障害を想定したシミュレーションを行います。

1. 前のステップで実行したワークロードのシミュレーションが継続していることを確認してください。
2. シミュレータのネットワーク図で、任意のノードを選択します。（選択されたノードがは青色になります。）ネットワーク図の下にある[Stop Selected Nodes]ボタンをクリックしてください。

<img src="img/938a17b26c9c9d4d.png" alt="938a17b26c9c9d4d.png"  width="624.00" />

3. 確認ウィンドウが表示されたら、[Yes]をクリックしてください。

<img src="img/f9f1c3ffa8497671.png" alt="f9f1c3ffa8497671.png"  width="624.00" />

4. パフォーマンスを表示するグラフでは、数秒のレイテンシ増加とスループット停止が発生します。これはRaftコンセンサスを行うタブレット・ピアが新しいリーダーを選択し、処理を再開するまでのタイムラグです。また、ネットワーク図からは停止したノードが消えて2ノード構成になったことがわかります。

<img src="img/1aded9a771e285ee.png" alt="1aded9a771e285ee.png"  width="624.00" />

5. YugabyteDB Managedのコンソールでの表示も確認しましょう。ダッシュボードの[Node] タブを表示すると、1つのノードが停止していることを示す赤い警告アイコンが表示されています。一方で、クラスタ自体はサービスを継続しているため、緑色のハートアイコン(Cluster Healthy and Fully Operational)が表示されています。

<img src="img/b8405efddc715d2f.png" alt="b8405efddc715d2f.png"  width="624.00" />

6. シミュレータに戻り、ネットワーク図の下にある[Restart Missing Nodes]ボタンをクリックしてください。確認ウィンドウが表示されたら、[Yes]をクリックします。

<img src="img/a142e54f988b33f7.png" alt="a142e54f988b33f7.png"  width="624.00" />

7. ノードの再起動とデータの同期には数分かかります。ノードの準備が整うと、タブレットの再配置とリーダー選挙が行われるため数秒のパフォーマンス低下が発生し、再起動したノードがネットワーク図に表示されます。

<img src="img/e69e8c2ecbeaa928.png" alt="e69e8c2ecbeaa928.png"  width="624.00" />

以上で、ノード停止によるAZ障害のシミュレートは完了です。クラスタ構成が変更される時に数秒のパフォーマンス低下が発生しますが、1つのノードが失われてもクラスタとしてはサービスが継続できることが確認できました。


## クラスターのスケール
Duration: 15:00


前のステップでは、ノード停止をシミュレートしました。ここでは、ワークロードが増加した時にクラスタをスケールアウトしたり、スケールインするオペレーションを行います。

1. 前のステップで実行したワークロードのシミュレーションが継続していることを確認してください。
2. クラスタへのリクエストが増加したことをシミュレートするため、追加のワークロードを実行します。左上にあるハンバーガーアイコンから、[Workload Management for Generic]ウィンドウを開き、[Include new Inserts]のオプションをオフにしてから[Run Simulation Workload]ボタンをクリックしてください。

<img src="img/b12ee8a635cee05a.png" alt="b12ee8a635cee05a.png"  width="624.00" />

3. [Active Workloads]タブを表示すると、２種類のワークロードが実行中であることがわかります。[Close]ボタンをクリックしてウィンドウを閉じてください。

<img src="img/5e93db07e10b02e0.png" alt="5e93db07e10b02e0.png"  width="624.00" />

4. シミュレータのネットワーク図の下にある、[Scale Cluster]ボタンをクリックします。
5. クラスタのサイズを6に設定し、[OK]をクリックしてください。

<img src="img/ee398a6fbb2826e7.png" alt="ee398a6fbb2826e7.png"  width="405.00" />

> aside negative
> 
> **Note:** YB Workload Simulatorは、YugabyteDB ManagedのREST APIを使用してクラスタの構成変更を行います。4ノードや5ノードへの拡張は、現時点ではサポートされていません。

6. "Cluster scaling initiated successfully!"のメッセージが表示されたことを確認して、[Close]ボタンでウィンドウを閉じます。
7. ノードの準備が整うと、新しく追加されたノードが、ネットワーク図に表示されます。タブレットが再配置されるタイミングでパフォーマンスの低下が発生しますが、クラスタ全体はサービスを継続していることを確認してください。

<img src="img/2df489ee49bd20fe.png" alt="2df489ee49bd20fe.png"  width="624.00" />

8. クラスタのノード数変更は、YugabyteDB Managedのコンソールからも可能です。クラスターのスケールインをManagedの画面から行ってみましょう。クラスタのダッシュボードを開きます。
9. 右上にある[Actions]ボタンをクリックして、[Edit Infrastructure]を選択します。(または[Settings]タブの[Regions]セクションの右にある、[Edit Infrastructure]からも可能です。)

<img src="img/2866c91f160a6ef1.png" alt="2866c91f160a6ef1.png"  width="624.00" />

10. [Nodes]のボックスに表示される下向き矢印をクリックして、ノードの数を3に変更します。

<img src="img/c9b57f05bdd2292d.png" alt="c9b57f05bdd2292d.png"  width="624.00" />

11. [Confirm and Save Changes]ボタンをクリックしてください。クラスタの構成変更中であるメッセージが画面上部に表示されます。

<img src="img/57c96e2200d934f6.png" alt="57c96e2200d934f6.png"  width="624.00" />

12. [View Details]ボタンをクリックして、タスクの進行状況を確認してください。

<img src="img/694c7c64a2ebb647.png" alt="694c7c64a2ebb647.png"  width="534.33" />

13. ワークロードシミュレータでも、ノード削除時のパフォーマンス低下やネットワーク図の変化が観察できることを確認してください。

<img src="img/8a3934a3f4c6e03c.png" alt="8a3934a3f4c6e03c.png"  width="624.00" />

以上で、ワークロード実行中のスケールアウトおよびスケールインのテストは完了です。構成変更時に数秒のパフォーマンス低下が発生しますが、クラスタ全体としてはサービスが継続できることを確認しました。


## ワークロードの停止と環境のクリーンアップ
Duration: 05:00


ここまで実行したワークロードの停止と、クラスタの停止または削除を行います。

1. ワークロードシミュレータ画面で、左上にあるハンバーガーアイコンをクリックします。
2. [Active Workloads]タブを表示すると、実行中のワークロードのリストが表示されます。ゴミ箱アイコンをクリックして、全てのワークロードを停止してください。

<img src="img/81a34369ffa64a24.png" alt="81a34369ffa64a24.png"  width="624.00" />

3. [Close]ボタンをクリックしてウィンドウを閉じます。
4. ワークロードが停止され、パフォーマンスのグラフが表示されなくなったことを確認してください。シミュレータを実行していたブラウザタブを閉じます。
5. シミュレータを起動したコマンドツール(Terminal等) に戻り、ctl+Cを入力して、アプリケーションの実行を停止してください。
6. YugabyteDB Managedのコンソール画面を開きます。クラスタのダッシュボードで右上にある[Action]ボタンをクリックし、 [Terminate Cluster]を選択してください。

<img src="img/3eb518302c5a07b2.png" alt="3eb518302c5a07b2.png"  width="624.00" />

> aside negative
> 
> **Note:** [Terminate Cluster]はクラスタの全てのノードを停止し、削除します。継続してREST APIのテスト等を実施したい場合は、[Pause Cluster]でクラスタを停止し、クラスタIDやデータを維持しても構いません。

7. クラスタ削除の確認画面が表示されます。クラスタの名前を入力して、[Delete]ボタンをクリックしてください。

<img src="img/a74576e365d3d74.png" alt="a74576e365d3d74.png"  width="600.00" />

以上で、環境のクリーンアップは完了です。


## まとめ



YugabyteDB Managedの耐障害性と拡張性のハンズオンが完了しました！

YugabyteDB Managedでは、ノード障害やAZ障害に耐えるクラスタ構成が簡単にできることが確認できたと思います。このハンズオンではシミュレータを使用しましたが、カスタムのアプリケーションでも同様のスケーラビリティと耐障害性を実現することができます。

### 次におすすめのハンズオン

以下のハンズオンも実施してみてください。

* XX
* YY

### 参考資料

*  [YugabyteDB Managed 公式ドキュメント](https://docs.yugabyte.com/preview/yugabyte-cloud/)
*  [YugabyteDB API Doc](https://api-docs.yugabyte.com/)
*  [[DSS 2022] Enterprise Scale: Uncover how a Distributed Architecture Delivers Scale and Resilience](https://www.youtube.com/watch?v=HCaadv-oKG4)


