---
id: ybm-handson-trial
summary: はじめてのYugabyteDB Aeon
status: [[draft]]
categories: ybm
feedback_link: https://yugabytedb-japan.github.io/
source: index.md
duration: 0

---

# はじめてのYugabyteDB Aeon

[Codelab Feedback](https://yugabytedb-japan.github.io/)


## はじめに



**Last Updated:** 2025-01-28

### **YugabyteDB Aeonとは？**

YugabyteDBはクラウドネイティブなアプリケーションに最適な、100%オープンソースの分散SQLデータベースです。分散SQLとは、従来のデータベースが単一ディスクへの書き込みを前提としたモノリシックなアーキテクチャだったのに対し、分散アーキテクチャ、つまり複数ノードへの書き込みや読み取りを前提とした様々な仕組みを提供しています。

このハンズオンで使用するYugabyteDB Aeonは、YugabyteDBをコアコンポーネントのデータベースとして使用している、フルマネージドのサービスです。サーバーやクラウドのアカウントを事前に準備しなくても、すぐにYugabyteDBをサービスとして使用することができます。

### **ハンズオンで学習すること**

* YugabyteDB Aeonのコンソール操作
* YugabyteDB Aeonでのクラスタ作成
* YugabyteDB AeonのWebコンソールを使用したデータベースの操作とタブレットの確認
* YugabyteDB Aeonクラスタの耐障害性とスケーラビリティ

### **ハンズオンで実施すること**

このハンズオンでは、YugabyteDB Aeonの環境を使用して以下の内容を実施します。

* YugabyteDB Aeon クラスタの作成 (シングルリージョン + 複数のAZの構成)
* PostgreSQLとの互換性の確認 (例: テーブル作成、データ挿入、クエリ実行)
* タブレットの確認
* ワークロード・シミュレータを使用した、ノード停止やクラスタのスケールアウト

### **ハンズオン実施に必要なもの**

* YugabyteDB Aeonアカウント
* ハンズオン実施端末のグローバルIPアドレス  (クラスタへのホワイトリスト登録のため)
* Java (バージョンがJava 19以上)


## YugabyteDB Aeonへのサインアップとログイン



YugabyteDB AeonはフルマネージドのDBaaS (データベース・アズ・サービス）です。サインアップしてアカウントを作成することで、すぐにデータベースを使い始めることができます。初めてYugabyteDB Aeonを使用する方は、以下の手順に従ってアカウントを作成してください。

* ブラウザで  [こちら](https://cloud.yugabyte.com/signup)にアクセスし、アカウントを作成してください。

> aside negative
> 
> **Note:** 企業のファイアウォール・ルールにより、yugabyte.com や hcaptcha.com がブロックされる場合があります。サインアップページにアクセスできない、[私は人間です] という画像確認のメニューが表示されない場合は、ホワイトリスト登録について貴社セキュリティ担当者にご確認ください。

2. 入力したEメールアドレス宛に、確認のメールが届きます。**[Finish Signup]** ボタンをクリックしてください。

<img src="img/14e49cfca864e065.png" alt="verifyemail.png"  width="624.00" />

3. 必要事項を入力し、アカウントの作成を完了してください。

<img src="img/9d1b9efa3328f933.png" alt="finishsignup.png"  width="624.00" />

4. アカウント作成が完了後、  [ログイン画面](https://cloud.yugabyte.com/login)に遷移します。上記で設定したユーザーIDとパスワードでログインしてください。

<img src="img/aa8afe05a71c7a0e.png" alt="loginscreen.png"  width="624.00" />

5. ログインが成功したら以下のような画面が表示されます。

<img src="img/fc184f3f5ed7a8dd.png" alt="loginscreen2.png"  width="624.00" />

以上で、YugabyteDB Aeonへのログインは完了です。


## クラスタの作成 (トポロジーの選択)



YugabyteDB Aeonでは、UIの項目選択によってクラスタ構成やノード仕様を設定することができます。

以下のような項目を設定します：

* クラウド・プロバイダー（AWS, Azure, GCP）
* データベースのバージョン（Production Track/Innovation Track）
* シングル・リージョンかマルチリージョンか
* 耐障害性のレベル
* なし
* ノードレベル
* AZレベル
* データの分散方法（マルチリージョン・クラスタのみ）
* マルチリージョン・クラスタ
* ジオ・パーティショニング

<img src="img/528e21d80ef87331.png" alt="topology.png"  width="624.00" />

このハンズオンでは、シングル・リージョンにAZレベルの耐障害性をもつクラスタを作成します。


## クラスタの作成 (シングル・リージョン + AZレベル障害耐性)



ここではAWS東京リージョンの各アベイラビリティ・ゾーンにノードを配置して、ゾーンレベルの耐障害性をもつ3ノードクラスタを構成します。

* 左側のメニューから **[Clusters]** を選択し、 **[Create Cluster]** ボタンをクリックしてください。

<img src="img/d487e95339e60ea4.png" alt="createcluster.png"  width="624.00" />

2. クラスタ作成のウィザードが開始します。右側のDedicatedにある **[Choose]** ボタンをクリックしてください。

<img src="img/25dec4d7e8c7f5e5.png" alt="choosecluster.png"  width="624.00" />

3. General settingsページが表示されます。クラスタの名前には適当な名前が自動生成されます。今回は **[test-cluster]** と名前を入力します。また、クラウド・プロバイダーには **[AWS]** 、データベース・バージョンは **[Innovation Track]** を選択して、 **[Next]** をクリックしてください。

<img src="img/ff1dca45c32e0196.png" alt="generalsettigs.png"  width="598.32" />

4. Cluster Setupページが表示されます。ページ上部にある **[Single-Region Deployment]** を選択します。1の耐障害性レベルには **[Availability Zone Level]** 、2のリージョンには **[Tokyo]** を選択してください。 クラスタの仕様は、vCPUを最小の **[2]** に設定します。

<img src="img/50ada1688da1068c.png" alt="clustersetup.png"  width="578.58" />

5. Cluster setupページの下部には、VPCの設定を行う箇所があります。このハンズオンでは使用しませんので、**[Select a VPC]** をオフにしたまま、**[Next]** をクリックしてください。

<img src="img/e5d9d1671b1d1c02.png" alt="vpc.png"  width="624.00" />

6. Network Accessの設定ページが表示されます。**[Add Current IP Address]** をクリックして、自分の端末のIPアドレスをアクセス許可リストに追加してください。

<img src="img/7d6dab0a9fdf308f.png" alt="addip.png"  width="612.27" />

> aside negative
> 
> **Note:** [Could not detect a valid IPv4 address] というエラーメッセージが表示される場合は、コマンドラインや外部サイトを使用して、ハンズオンで使用しているマシンのグローバルIPを確認し、**[Create New IP Allow List]** からIP許可リストに追加してください。

7. **[Next]** をクリックします。保管データの暗号化の設定を行うページが表示されます。今回は使用しないため、そのままで **[Next]** をクリックしてください。

<img src="img/69c6f32f4eda436e.png" alt="encryptcluster.png"  width="624.00" />

8. DB Credentialsページが表示されます。ユーザー名とパスワードは自動設定されます。設定をカスタマイズしたい場合は、 **[Add your own credentials]** をクリックしてユーザー名をパスワードを自分で設定します。このままで問題なければ **[Download credentials]** ボタンをクリックして、アクセス情報のファイルをローカルに保存してください。

<img src="img/45831c74cff50aaf.png" alt="dbcred.png"  width="624.00" />

> aside negative
> 
> **Note:** このアクセス情報は、ハンズオンの後のステップで使用します。ファイルを保存した場所を忘れないようにしてください。

9. **[Create Cluster]** ボタンをクリックします。プロビジョニングが開始され、DBクラスタが開始するまでに数分かかります。

<img src="img/683eb3674905a9b5.png" alt="provision.png"  width="624.00" />

10. 起動が完了するとクラスターのダッシュボードが表示されます。

<img src="img/957da16a54967f69.png" alt="clustersetupdone.png"  width="624.00" />

以上で、クラスタの作成は完了です。


## Cloud Shell を使用したクラスタへのアクセス



YugabyteDB Aeonのクラスタにアクセスするには、いくつかの方法があります。

* Cloud Shell （ブラウザベースのCLI）
* YugabyteDB Client Shell (MacやLinuxのターミナル、またはDockerを使用するCLI）
* その他のデータベース・クライアントアプリケーション (pgAdmin, DBeaverなど）

このハンズオンでは、Cloud Shellを使用したアクセスを行います。

* 前のステップで作成したクラスタのダッシュボードを開きます。
* クラスタのダッシュボードの右上にある、[Coneect]ボタンをクリックしてください。
* クラスタに接続するための方法が複数表示されます。一番上にある[Launch Cloud Shell]ボタンをクリックしてください。

<img src="img/731b0fefd345f245.png" alt="731b0fefd345f245.png"  width="624.00" />

4. データベース名やユーザー名を確認し、[Confirm]ボタンをクリックしてください。

<img src="img/c51e511aac00fb57.png" alt="c51e511aac00fb57.png"  width="600.00" />

5. 新しいブラウザ・タブが開き、クラスタにアクセスするためのシェルが表示されます。クラスタ作成時にダウンロードしたcredentialファイルに記載された、パスワードを入力してください。

<img src="img/cff23d35348c2544.png" alt="cff23d35348c2544.png"  width="624.00" />

> aside negative
> 
> **Note:** Windowsではブラウザの種類によって、ctl+vによるCloud Shellへの貼り付けができない場合があります。右クリックをしてメニューから[貼り付け]を選択してください。

6. パスワードが認証されると、あらかじめ作成されたyugabyteデータベースに接続し、YSQLの入力モード (yugabyte=&gt;) になります。

これで Cloud Shell からクラスタにアクセスすることができました。


## PostgreSQLとの互換性の確認



YugabyteDB Aeonでは、初めて使用する開発者向けのチュートリアルが提供されています。ここでは、チュートリアルのガイドにそって、前のステップで接続したyugabyteデータベースにテーブルを作成します。

* 前のステップでyugabyteデータベースに接続した、Cloud Shellのブラウザ・タブが開いていることを確認してください。
* 左側に表示された[YugabyteDB Quick Start Guide]から、[Step 1. Create a Table]をクリックしてください。チュートリアルの手順がCloud Shellの下側に表示されます。

<img src="img/3c3ce4326fa5e8be.png" alt="3c3ce4326fa5e8be.png"  width="624.00" />

3. Step1では、部門(dept)テーブルを作成します。テーブルを作成するSQLをCloud Shellに直接入力するか、SQLのボックス右上にある[Copy]ボタンをクリックして貼り付ける、もしくは、[Run]ボタンをクリックして、SQLを実行してください。[CREATE TABLE]のメッセージが表示されたことを確認します。

<img src="img/3a6d432f2731fb40.png" alt="3a6d432f2731fb40.png"  width="624.00" />

4. 同様に、Step2では従業員(emp)テーブルを作成します。[CREATE TABLE]のメッセージを確認してください。

<img src="img/dea12a6bbb2d6723.png" alt="dea12a6bbb2d6723.png"  width="624.00" />

5. チュートリアル手順の上部にある[Next Step &gt;]ボタンをクリックするか、左側のガイドから[Step 2. Insert Data]をクリックして、データを挿入する手順を表示します。

<img src="img/3d7721a5aa45a0d5.png" alt="3d7721a5aa45a0d5.png"  width="624.00" />

6. 部門(dept)テーブルにデータを挿入します。[Step1]に表示されたSQLを実行してください。4行のデータが挿入され、[INSERT 0 4]のメッセージが表示されます。

<img src="img/57c6ae71f99214e8.png" alt="57c6ae71f99214e8.png"  width="624.00" />

7. 同様に、従業員(emp)テーブルにデータを挿入します。[Step2]に表示されたSQLを実行してください。14行のデータが挿入され、[INSERT 0 14]のメッセージが表示されるはずです。

<img src="img/17f0e2c2c966e6db.png" alt="17f0e2c2c966e6db.png"  width="624.00" />

8. 改めて、部門(dept)テーブルと従業員(emp)テーブルにデータが挿入されたことを確認します。これでテーブル作成とデータ挿入のステップは完了です。

<img src="img/f58fe0996a6d579.png" alt="57c6ae71f99214e8.png"  width="624.00" />

9. データの挿入が完了すると、右側のウィンドウにチュートリアルへのリンクが表示されます。[Let's get started with the tutorials -&gt;]をクリックしてください。

<img src="img/d9dd157ff1c40707.png" alt="d9dd157ff1c40707.png"  width="243.50" />

10. 左側ウィンドウにYSQLのチュートリアルが表示されます。先ほど作成したテーブルを使用して、Join(テーブル結合)やプリペアドステートメントなどの機能を試してみましょう。
11. [Scenario 1. SQL Updates]をクリックします。こちらでは、JobがManagerではない従業員に対して現在の給与(sal)に100を加算し、給与額の値を更新します。

<img src="img/456678368d077050.png" alt="update_salary.png"  width="624.00" />

12. [Scenario 2. Join]をクリックします。こちらでは、empテーブルを自己結合し、Managerより給与が高い社員を抽出するクエリを実行します。

<img src="img/4aad49f8adbb0110.png" alt="selfjoin.png"  width="624.00" />

13. [Scenario 3. Prepared Statements]をクリックします。こちらでは、社員番号をパラメータとして受け取り、従業員(emp)テーブルから名前(ename)と給与(sal)を返すプリペアドステートメントを作成します。

<img src="img/e792ad7a6bee3a89.png" alt="ps1.png"  width="624.00" />

14. PREPAREという応答が返ったことが確認できたら、`execute emp_territories(7900);` のようにパラメータに従業員番号(例: 7900)を入れて実行してみてください。下記の様に、従業員番号7900の社員の名前と給与が返ってきたことが確認できます。

<img src="img/348fa78b4b4cfdcc.png" alt="ps2.png"  width="624.00" />

15. プリペアドステートメントは、セッション内に限り有効で、他のユーザーと共有することはできません。使用しなくなったプリペアドステートメントは無効化することでメモリを解放することができます。`deallocate emp_territories;` と入力して作成したステートメントを削除してください。

<img src="img/850a169fa8c643c9.png" alt="ps3.png"  width="624.00" />

16. 他にも再帰クエリなど、先ほど作成したテーブルを使用して、様々なSQL機能を試すことができます。このハンズオンでは実施しませんが、お時間のある時に是非試してみてください。

<img src="img/dfa7f4575bcb445e.png" alt="tutorial1.png"  width="190.71" />  <img src="img/81f59d0356e1fd7e.png" alt="tutorial2.png"  width="176.50" />  <img src="img/e293d9f1fe50576d.png" alt="tutorial3.png"  width="181.42" />


## タブレットの確認



YugabyteDBは、テーブル内のデータを「タブレット（tablet）」または「シャード（shard）」と呼ばれる小さな単位に分割します。また、シャーディング（Sharding）とは、大規模なテーブルを「シャード」と呼ばれる小さな断片に分割し、それらを複数のサーバーに分散させるプロセスを指します。主にレンジシャーディング（range sharding）とハッシュシャーディング（hash sharding）をサポートしています。

<img src="img/3d37a669e761a25a.png" alt="sharding.png"  width="624.00" />

まず、下記のSQLを実行し、empテーブルの情報を確認します。

```
yugabyte=> \d+ emp;   
```

出力結果を確認すると、下記のとおりempnoがプライマリキーとして設定されています。また、empno列の値に基づいてハッシュ関数が計算され、その結果によってデータが異なるシャードに分散されるハッシュシャーディングが使用されています。

<img src="img/498a2fa3ac237b20.png" alt="pksharding.png"  width="624.00" />

ここで、下記のSQLを実行します。num_hash_key_columnsは上記のとおり、ハッシュキーとして設定されているプライマリキーの数を示しており、ハッシュシャーディングによって、empテーブルのデータが3つのタブレット(num_tablets)に分散されていることが確認できます。ハッシュシャーディングを使用することで、データが均等に分散され、特定のタブレットにデータが集中することを防ぎ、パフォーマンスの向上とスケーラビリティの確保が図られます。

```
yugabyte=> SELECT * FROM yb_table_properties('emp'::regclass);   
```

<img src="img/62c691e21a928b11.png" alt="tabletnumber.png"  width="624.00" />

また、YugabyteDB AeonではReplication Factor（レプリケーションファクター）が3に設定されています。今回3つのノードを作成したため、各タブレットは3つのノードにコピーされます。1つがリーダー（Leader）で、残りの2つがフォロワー（Follower）です。

<img src="img/d113ea3875d0b9a3.png" alt="tabletimage.png"  width="624.00" />

これで、テーブルのタブレットに関する状況を確認できました。


## YugabyteDB Aeonの耐障害性と拡張性



### **分散SQLデータベースの高可用性と耐障害性**

分散SQLデータベースであるYugabyteDBは、拡張性（水平スケール）や高可用性を前提とした設計となっており、デフォルトで高い耐障害性を備えています。スケール時にもサービス停止は発生せず、ローリング・アップグレードといったゼロダウンタイムでのオペレーションが可能です。

このハンズオンでは、YugabyteDB Aeonを使用して、ノード障害やクラスタのスケール時にもダウンタイムが発生せず、クラスタが処理を継続できることを確認します。

### **クラスタのアクセス情報の確認**

このハンズオンでは、ワークロード・シミュレータを使用したノード停止やクラスタのスケールアウトを実施します。こちらの実施にあたり、いくつかの情報やファイルが必要となります。このセクションでは、以下の情報を確認します。

* データベースのユーザー名とパスワード
* 作成したクラスタのID
* YugabyteDB AeonのアカウントID、プロジェクトID
* YugabyteDB Aeon REST APIを使用するためのAPI Key
* クラスタのダッシュボードを開きます。
* クラスタのダッシュボードの右上にある、[Coneect]ボタンをクリックしてください。
* クラスタに接続するための方法が複数表示されます。一番下にある[Connect to your Application]の右向き矢印をクリックします。

<img src="img/33908e1aa021ef64.png" alt="33908e1aa021ef64.png"  width="573.80" />

4. SSL通信のための証明書をダウンロードします。[Download CA  Cert]のリンクをクリックして、証明書ファイル (root.crt) をダウンロードしてください。

<img src="img/28a00b2ab527e454.png" alt="28a00b2ab527e454.png"  width="577.04" />

> aside negative
> 
> **Note:** この証明書は、ハンズオンの後のステップで使用します。ファイルを保存した場所を忘れないようにしてください。

5. [2)Use the following parameters to connect to your cluster]セクションの下で、YSQLの[Parameters]タブを選択します。[Host] の横にあるコピーボタンをクリックしてメモしてください。ホスト名は、&lt;Region&gt;.&lt;ClusterID&gt;.&lt;Cloud&gt;.ybdb.io で設定されています。最初の.(ドット) の後の文字列(上の例では、4e095914-fb49-4b71-8e3a-3dee1073f221)が、クラスターIDです。

<img src="img/217baee793e08093.png" alt="217baee793e08093.png"  width="624.00" />

6. 右上にあるXをクリックして、[Connect to Cluster]のウィンドウを閉じます。
7. 左側のメニューから、[Security]を選択します。[API Keys]タブを選択して、[Create API Key] ボタンをクリックしてください。

<img src="img/5b53a266991a0bd7.png" alt="apikey.png"  width="624.00" />

8. API Keyの名前に任意の名前を入力します。Roleは[Admin]を選択し、[Generate Key]ボタンをクリックしてください。

<img src="img/69ac166c8d48dfb2.png" alt="69ac166c8d48dfb2.png"  width="624.00" />

9. APIキーが表示されます。キーはこの画面でしか表示されないので、必ずキーをコピーしてメモしてください。
10. [View Details] セクションを開くと、アカウントIDとプロジェクトIDが表示されます。この2つもコピーして、メモしておいてください。

<img src="img/e896dc9083e67328.png" alt="e896dc9083e67328.png"  width="624.00" />

これで、クラスタへのアクセスに必要な情報が確認できました。


## シミュレータのセットアップ



YB Workload Simulatorは、アプリケーションから見たレイテンシやスループットをシミュレートするJavaアプリケーションで、YugabyteDBのコードサンプルとして   [GitHub](https://github.com/YugabyteDB-Samples/yb-workload-simulator)で公開されています。

シミュレータを使用することで、以下のことを実施できます。

* ワークロード実行のためのテーブルを作成/ドロップ
* ワークロード実行のためのデータをロード
* TPSやJavaのスレッド数を調整してワークロードを実行
* クラスタのトポロジーを可視化
* YugabyteDB Aeonのオペレーション（ノード停止/再開、クラスタのスケール）

> aside negative
> 
> **Note:** シミュレータの実行にはJava 19以上のバージョンが必要です。コマンドツール(Terminal等) でjava -versionと入力し、Javaのインストール・バージョンを確認してください。

*  [リリース](https://github.com/YugabyteDB-Samples/yb-workload-simulator/releases)ページにアクセスして、シミュレータのjarファイルをダウンロードしてください。今回は、  [yb-workload-sim-0.0.7.jar](https://github.com/YugabyteDB-Samples/yb-workload-simulator/releases/download/v0.0.7/yb-workload-sim-0.0.7.jar)を使用します。
* 前のステップでダウンロードしたクラスタのアクセス情報、証明書、アカウントIDやプロジェクトID、API Keyを確認して、以下のコマンドを入力します。

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
     -jar ./yb-workload-sim-0.0.7.jar
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

<img src="img/45fef054723a15fe.png" alt="simulator1.png"  width="624.00" />

4. ブラウザを開いて、   [http://localhost:8080/](http://localhost:8080/) にアクセスしてください。シミュレータが表示されます。

<img src="img/c55a27e870b9d6ef.png" alt="simulator2.png"  width="624.00" />

5. [Network Diagram and Aggregate Statistics]ウィンドウの右上にある、歯車アイコンをクリックし、[Yugabyte Options] を選択してください。

<img src="img/4ba23d70bf9cf771.png" alt="4ba23d70bf9cf771.png"  width="260.00" />

6. パスワードの設定を要求する画面が表示されます。任意のパスワードを設定し、メモしておいてください。[OK]ボタンをクリックして、一度ウィンドウを閉じます。
7. 再度、歯車アイコンをクリックして[Yugabyte Options]を選択し、[Database in Use]に[Yugabyte Managed]を選択します。パスワードの入力を求められたら、先ほどのパスワードを入力し[Validate Password]をクリックしてください。

<img src="img/4d724c5bc4b15374.png" alt="4d724c5bc4b15374.png"  width="624.00" />

8. API Access KeyやアカウントIDを入力する画面が表示されます。メモしておいた内容を入力し、[OK] ボタンをクリックしてください。

<img src="img/fe90354f3621bf3c.png" alt="configscreen.png"  width="624.00" />

9. 設定が保存されると、ネットワーク図の下にクラスタを操作するボタンが表示されます。

<img src="img/4020bd74e6efefd5.png" alt="simulatornetwork.png"  width="624.00" />


## テーブル作成とデータのロード



ワークロード・シミュレータでは、シミュレーションを実行するためのテーブルやデータを生成する機能を提供しています。

* シミュレータの左上にある、ハンバーガーアイコンをクリックしてください。

<img src="img/9834cfa79f3c27b9.png" alt="menu.png"  width="274.00" />

2. [Workload Management for Quickship]ウィンドウが開きます。[Usable Operations]タブを表示してください。

<img src="img/f15333eb1dd3fc06.png" alt="createtable.png"  width="624.00" />

3. [Create Tables]セクションを開き、 [Run Create Tables Workload]ボタンをクリックしてください。"Workload CREATE_TABLES successfully submitted"のメッセージがウィンドウ下部に表示されます。
4. YugabyteDB Aeonのコンソールで、クラスタのダッシュボードの[Tables]タブを表示し、ProductsとOrdersの2つのテーブルが作成されていることを確認します。

<img src="img/48202c778f26ee91.png" alt="tableinfo.png"  width="624.00" />

5. シミュレータ画面に戻り、[Workload Management for Quickship]ウィンドウを開きます。
6. [Seed Data]セクションを開き、[Run Seed Data Workload]ボタンをクリックしてください。"Workload SEED_DATA successfully submitted."のメッセージがウィンドウ下部に表示されます。
7. YugabyteDB Aeonのコンソールで、実際にデータがロードされたことを確認してみましょう。クラスタのダッシュボード右上にある[Connect]ボタンをクリックし、表示されるウィンドウで[Launch Cloud Shell]をクリックします。

<img src="img/0.png" alt="731b0fefd345f245.png"  width="624.00" />

8. データベース名やユーザー名を確認し、[Confirm]ボタンをクリックしてください。

<img src="img/c51e511aac00fb57.png" alt="c51e511aac00fb57.png"  width="600.00" />

9. 新しいブラウザ・タブが開き、クラスタにアクセスするためのシェルが表示されます。クラスタアクセス情報のファイルに記載された、パスワードを入力してください。

<img src="img/cff23d35348c2544.png" alt="cff23d35348c2544.png"  width="624.00" />

10. パスワードが認証されると、YSQLの入力モードになります。`SELECT count(*) from orders;`と入力し、エンターキーを押してください。10000行のデータがロードされていることがわかります。

<img src="img/b71066a1a95b4dff.png" alt="selectorder.png"  width="624.00" />

11. 続いて、データの一部を参照するため、`SELECT * from orders limit 10;`と入力してください。ロードされたデータの内容を確認したら、`exit;`と入力してシェルを終了し、タブを閉じます。   <img src="img/1d4c537342f5330e.png" alt="selectorder2.png"  width="624.00" />


## ワークロードの実行



シミュレータで作成したテーブルとデータを使用して、YugabyteDB Aeonのクラスタに読み取り/書き込みのワークロードを実行します。

* ワークロード・シミュレータのブラウザ・タブを開き、左上のハンバーガーアイコンから[Workload Management for Quickship]ウィンドウを表示してください。
* [Simulation]セクションで、スループットとスレッドの設定を確認します。読み取りだけでなく、書き込みのワークロードも生成するため、[Include placing of new orders]のボタンをオンにして、[Run Simulation Workload]ボタンをクリックしてください。

<img src="img/9dd4ac959d814263.png" alt="simulationworkload.png"  width="624.00" />

3. "Workload RUN_SIMULATION successfully submitted."のメッセージがウィンドウ下部に表示されることを確認し、[Close]ボタンをクリックしてウィンドウを閉じてください。
4. レイテンシーとスループットのグラフを確認してください。ワークロードが継続的に生成され、データベースがサービスを継続していることがわかります。

<img src="img/fc1d1d310fb3f51c.png" alt="workload1.png"  width="624.00" />


## AZ障害のシミュレート



このハンズオンの前のステップで、複数のアベイラビリティゾーン(AZ)にノードを配置したクラスタを構成しました。そのため、AZレベルの障害であれば、クラスタは影響を受けずにサービスを継続することができる耐障害性を備えています。

このセクションでは、クラスタを構成する一つのノードを停止することでAZレベル障害を想定したシミュレーションを行います。

* 前のステップで実行したワークロードのシミュレーションが継続していることを確認してください。
* シミュレータのネットワーク図で、任意のノードを選択します。（選択されたノードがは赤色になります。）ネットワーク図の下にある[Stop Selected Nodes]ボタンをクリックしてください。

<img src="img/d0e90af3a7828b52.png" alt="nodeselect.png"  width="624.00" />

3. 確認ウィンドウが表示されたら、[Yes]をクリックしてください。

<img src="img/692cb888ed70f437.png" alt="stopnode.png"  width="624.00" />

4. パフォーマンスを表示するグラフでは、数秒のレイテンシ増加とスループット停止が発生します。これはRaftコンセンサスを行うタブレット・ピアが新しいリーダーを選択し、処理を再開するまでのタイムラグです。また、ネットワーク図からは停止したノードが消えて2ノード構成になったことがわかります。

<img src="img/6e740f51f7b2d047.png" alt="diagram.png"  width="624.00" />

5. YugabyteDB Aeonのコンソールでの表示も確認しましょう。ダッシュボードの[Node] タブを表示すると、1つのノードが停止していることを示す赤い警告アイコンが表示されています。一方で、クラスタ自体はサービスを継続しているため、緑色のハートアイコン(Cluster Healthy and Fully Operational)が表示されています。

<img src="img/73410b8f09358cc.png" alt="clusterui.png"  width="624.00" />

6. シミュレータに戻り、ネットワーク図の下にある[Restart Missing Nodes]ボタンをクリックしてください。確認ウィンドウが表示されたら、[Yes]をクリックします。

<img src="img/1fae0a39897ae1b0.png" alt="restartconfirmation.png"  width="624.00" />

7. ノードの再起動とデータの同期には数分かかります。ノードの準備が整うと、タブレットの再配置とリーダー選挙が行われ、再起動したノードがネットワーク図に表示されます。

<img src="img/75369a79660276e9.png" alt="restart2.png"  width="624.00" />

以上で、ノード停止によるAZ障害のシミュレートは完了です。クラスタ構成が変更される時に数秒のパフォーマンス低下が発生しますが、1つのノードが失われてもクラスタとしてはサービスが継続できることが確認できました。


## クラスターのスケールアウト



前のステップでは、ノード停止をシミュレートしました。ここでは、ワークロードが増加した時にクラスタをスケールアウトするオペレーションを行います。

* 前のステップで実行したワークロードのシミュレーションが継続していることを確認してください。
* クラスタへのリクエストが増加したことをシミュレートするため、追加のワークロードを実行します。左上にあるハンバーガーアイコンから、[Workload Management for Quickship]ウィンドウを開き、[Include placing of new orders]のオプションをオフにしてから[Run Simulation Workload]ボタンをクリックしてください。

<img src="img/df388481d207e97d.png" alt="workloadagain.png"  width="624.00" />

3. [Active Workloads]タブを表示すると、２種類のワークロードが実行中であることがわかります。[Close]ボタンをクリックしてウィンドウを閉じてください。

<img src="img/d4f208e923d90ef5.png" alt="twoworkloads.png"  width="624.00" />

4. クラスタのノード数変更を、YugabyteDB Aeonのコンソールから実施します。クラスターのスケールアウトをYugabyteDB Aeonの画面から行ってみましょう。クラスタのダッシュボードを開きます。
5. 右上にある[Actions]ボタンをクリックして、[Edit Infrastructure]を選択します。(または[Settings]タブの[Regions]セクションの右にある、[Edit Infrastructure]からも可能です。)

<img src="img/94ac1df70ab4d9ff.png" alt="editinfra.png"  width="624.00" />

6. [Nodes]のボックスに表示される上向き矢印をクリックして、ノードの数を6に変更します。

<img src="img/a385495a1a40737a.png" alt="numbernodes.png"  width="624.00" />

7. [Confirm and Save Changes]ボタンをクリックしてください。クラスタの構成変更中であるメッセージが画面上部に表示されます。

<img src="img/7d60c7c7c89599e8.png" alt="clusterediting.png"  width="624.00" />

8. [View Details]ボタンをクリックして、タスクの進行状況を確認してください。

<img src="img/9ec91447be2ea8b2.png" alt="clusteractivityinfo.png"  width="534.33" />

9. ワークロードシミュレータでも、ノード削除時のパフォーマンス低下やネットワーク図の変化が観察できることを確認してください。

<img src="img/fd53eeac213a8ede.png" alt="nodeaction1.png"  width="624.00" />

10. ノード数の変更が完了後、YugabyteDB Aeonのコンソールで、クラスタのダッシュボードの[Nodes]タブを表示し、ノードの数が6になっていることを確認してください。

<img src="img/7a25f30848f90068.png" alt="nodeaction3.png"  width="624.00" />

以上で、ワークロード実行中のスケールアウトおよびスケールインのテストは完了です。構成変更時に数秒のパフォーマンス低下が発生しますが、クラスタ全体としてはサービスが継続できることを確認しました。


## ワークロードの停止と環境のクリーンアップ



ここまで実行したワークロードの停止と、クラスタの停止または削除を行います。

* ワークロードシミュレータ画面で、左上にあるハンバーガーアイコンをクリックします。
* [Active Workloads]タブを表示すると、実行中のワークロードのリストが表示されます。ゴミ箱アイコンをクリックして、全てのワークロードを停止してください。

<img src="img/1dac45cfca4769f5.png" alt="stopworkload.png"  width="624.00" />

3. [Close]ボタンをクリックしてウィンドウを閉じます。
4. ワークロードが停止され、パフォーマンスのグラフが表示されなくなったことを確認してください。シミュレータを実行していたブラウザタブを閉じます。
5. シミュレータを起動したコマンドツール(Terminal等) に戻り、ctl+Cを入力して、アプリケーションの実行を停止してください。
6. YugabyteDB Aeonのコンソール画面を開きます。クラスタのダッシュボードで右上にある[Action]ボタンをクリックし、 [Terminate Cluster]を選択してください。

<img src="img/d1949647388e1066.png" alt="terminatecluster1.png"  width="624.00" />

> aside negative
> 
> **Note:** [Terminate Cluster]はクラスタの全てのノードを停止し、削除します。継続してREST APIのテスト等を実施したい場合は、[Pause Cluster]でクラスタを停止し、クラスタIDやデータを維持しても構いません。

7. クラスタ削除の確認画面が表示されます。クラスタの名前を入力して、[Delete]ボタンをクリックしてください。

<img src="img/5049de1a325dce4.png" alt="terminatecluster2.png"  width="600.00" />

以上で、環境のクリーンアップは完了です。


## まとめ



このハンズオンでは、YugabyteDB Aeonを使用して、下記の内容を確認しました。

* YugabyteDB Aeonのコンソール操作
* YugabyteDB Aeonでのクラスタ作成
* YugabyteDB AeonのWebコンソールを使用したデータベースの操作とタブレットの確認
* YugabyteDB Aeonクラスタの耐障害性とスケーラビリティ

さらにYugabyte Aeonについて学びたい場合は、下記のリンクをご参照ください。

*  [YugabyteDB Aeon公式ドキュメント](https://docs.yugabyte.com/preview/yugabyte-cloud/)
*  [YugabyteDB Managed Basics](https://university.yugabyte.com/courses/yugabytedb-managed-basics)


