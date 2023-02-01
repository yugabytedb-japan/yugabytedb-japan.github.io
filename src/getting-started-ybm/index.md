---
id: getting-started-ybm
summary: このハンズオンでは、YugabyteDB Managed でデータベースを作成し、テーブルの作成やデータの挿入・読取・更新などの簡単な操作を行います。
status: [draft]
authors: Arisa Izuno
categories: workshop,japanese,beginner
tags: ybm
feedback_link: https://yugabytedb-japan.github.io/
source: 14ZQPs77BXVbGtRZ3AzbroHwOvVrAKyu8B3mZCmmuPns
duration: 35

---

# YugabyteDB Managed 入門

[Codelab Feedback](https://yugabytedb-japan.github.io/)


## はじめに
Duration: 01:00


**Last Updated:** 2022-02-03

### **分散SQLデータベースとは？**

YugabyteDBはクラウドネイティブなアプリケーションに最適な、100%オープンソースの分散SQLデータベースです。分散SQLとは、従来のデータベースが単一ディスクへの書き込みを前提としたモノリシックなアーキテクチャだったのに対し、分散アーキテクチャ、つまり複数ノードへの書き込みや読み取りを前提とした様々な仕組みを提供しています。

このハンズオンでは、YugabyteDB Managedを使用して、使い慣れたSQLを使って簡単にデータベースやテーブルを作成したり、データの書き込みや読み取りができることを確認します。

### **ハンズオンで実施すること**

このハンズオンでは、YugabyteDB Managedの無料枠を使用して単一ノードクラスタを作成します。以下の内容を実施します:

* YugabyteDB Managedへのサインアップ
* 単一ノード・クラスタの作成
* テーブルの作成
* データの挿入、読取、更新

### **ハンズオンで学習すること**

* YugabyteDB Managedのコンソール操作
* YugabyteDB Managedでのクラスタ作成
* YugabyteDB ManagedのWebコンソールを使用したデータベースの操作

### **ハンズオン実施に必要なもの**

* インターネット接続可能な端末
* ブラウザ (Chrome, Safari, Microsoft Edge, Firefoxなど)
* YugabyteDB Managedアカウント (ハンズオン内で登録します)
* pgAdmin (Step7実施の場合)


## YugabyteDB Managedへのサインアップ
Duration: 03:00


YugabyteDB ManagedはフルマネージドのDBaaS (データベース・アズ・サービス）です。サインアップしてアカウントを作成することで、すぐにデータベースを使い始めることができます。初めてYugabyteDB Managedを使用する方は、以下の手順に従ってアカウントを作成してください。

1. ブラウザで [こちら](https://cloud.yugabyte.com/signup)にアクセスし、アカウントを作成してください。

<img src="img/5768cf0371033212.png" alt="5768cf0371033212.png"  width="624.00" />

2. 入力したEメールアドレス宛に、確認のメールが届きます。[Verify Email] ボタンをクリックしてください。

<img src="img/99ef95b51b691889.png" alt="99ef95b51b691889.png"  width="389.50" />

3. ログイン画面にリダイレクトされます。設定したユーザーIDとパスワードでログインしてください。ログインが成功したら以下のような画面が表示されます。

<img src="img/310bfd3620274448.png" alt="310bfd3620274448.png"  width="624.00" />

以上で、YugabyteDB Managedへのサインアップは完了です。


## 無料枠のクラスター作成
Duration: 01:00


YugabyteDB Managedでは、アカウントにつき1つ、期間の限りなく使用可能な1ノードのクラスタを作成可能です。高可用性やスケールアウトといった本番運用に必要な機能は試すことができませんが、Postgres互換のYSQLやCassanrda互換のYCQLを試すなど、チュートリアルに最適なSandbox環境として使用することができます。

以下は、SandboxとDedicated (有償環境) の機能比較です。

このハンズオンでは、この無料枠で1ノードのクラスタを作成します。


## シングルノードクラスタの作成
Duration: 10:00


ここではYugabyteDB Managedの無料枠を使用して、1ノードのクラスタを作成します。

1. YugabyteDB Managedのアカウントにログインします。
2. 左側のメニューから[Clusters]を選択し、[Add Cluster]ボタンをクリックしてください。
3. クラスタ作成のウィザードが開始します。左側のSandboxの下にある [Choose]ボタンをクリックしてください。

<img src="img/944159f66e92fd45.png" alt="944159f66e92fd45.png"  width="624.00" />

4. Cluster settingsページが表示されます。クラスタの名前には適当な名前が自動生成されます。お好きなクラウド・プロバイダとリージョンを選択して、[Next] をクリックしてください。

<img src="img/a8a8216a18668b95.png" alt="a8a8216a18668b95.png"  width="624.00" />

5. Network Accessページが表示されます。[Add Current IP Address]をクリックして、自分の端末のIPアドレスをアクセス許可リストに追加してください。

<img src="img/956d2dd7623dca44.png" alt="956d2dd7623dca44.png"  width="624.00" />

6. [Next]をクリックします。
7. DB Credentialsページが表示されます。ユーザー名とパスワードは自動設定されます。設定をカスタマイズしたい場合は、[Add your own credentials]をクリックしてユーザー名やパスワードを自分で設定します。 [Download credentials]ボタンをクリックして、アクセス情報のファイルをローカルに保存してください。

<img src="img/e4e6075adfd218ec.png" alt="e4e6075adfd218ec.png"  width="624.00" />

> aside negative
> 
> **Note:** このアクセス情報は、ハンズオンの後のステップで使用します。ファイルを保存した場所を忘れないようにしてください。

8. [Create Cluster]ボタンをクリックします。プロビジョニングが開始され、DBクラスタが開始するまでに数分かかります。

<img src="img/b767e2af29c4f507.png" alt="b767e2af29c4f507.png"  width="624.00" />

9. 起動が完了するとクラスターのダッシュボードが表示されます。

<img src="img/48aa512f673d3a4.png" alt="48aa512f673d3a4.png"  width="624.00" />

以上で、クラスタの作成は完了です。


## Cloud Shell を使用したクラスタへのアクセス
Duration: 03:00


YugabyteDB Managedのクラスタにアクセスするには、いくつかの方法があります。

* Cloud Shell （ブラウザベースのCLI）
* YugabyteDB Client Shell (MacやLinuxのターミナル、またはDockerを使用するCLI）
* その他のデータベース・クライアントアプリケーション (pgAdmin, DBeaverなど）

このセクションでは、Cloud Shellを使用したアクセスを行います。

1. 前のステップで作成したクラスタのダッシュボードを開きます。
2. クラスタのダッシュボードの右上にある、[Coneect]ボタンをクリックしてください。
3. クラスタに接続するための方法が複数表示されます。一番上にある[Launch Cloud Shell]ボタンをクリックしてください。

<img src="img/731b0fefd345f245.png" alt="731b0fefd345f245.png"  width="624.00" />

4. データベース名やユーザー名を確認し、[Confirm]ボタンをクリックしてください。

<img src="img/c51e511aac00fb57.png" alt="c51e511aac00fb57.png"  width="600.00" />

5. 新しいブラウザ・タブが開き、クラスタにアクセスするためのシェルが表示されます。クラスタ作成時にダウンロードしたcredentialファイルに記載された、パスワードを入力してください。

<img src="img/cff23d35348c2544.png" alt="cff23d35348c2544.png"  width="624.00" />

6. パスワードが認証されると、あらかじめ作成されたyugabyteデータベースに接続し、YSQLの入力モード (yugabyte=&gt;) になります。

        これでCloud Shellからクラスタにアクセスすることができました。


## テーブル作成とデータ挿入
Duration: 05:00


YugabyteDB Managedでは、初めて使用する開発者向けのチュートリアルが提供されています。ここでは、チュートリアルのガイドにそって、前のステップで接続したyugabyteデータベースにテーブルを作成します。

1. 前のステップでyugabyteデータベースに接続した、Cloud Shellのブラウザ・タブが開いていることを確認してください。
2. 左側に表示された[YugabyteDB Quick Start Guide]から、[Step 1. Create a Table]をクリックしてください。チュートリアルの手順がCloud Shellの下側に表示されます。

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

8. データの挿入が完了すると、右側のウィンドウにチュートリアルへのリンクが表示されます。[Let's get started with the tutorials -&gt;]をクリックしてください。

<img src="img/d9dd157ff1c40707.png" alt="d9dd157ff1c40707.png"  width="243.50" />

9. 左側ウィンドウにYSQLのチュートリアルが表示されます。先ほど作成したテーブルを使用して、様々なSQL機能を試すことができます。このハンズオンでは実施しませんが、お時間のある時に是非やってみてください。

<img src="img/996e2b93101ac291.png" alt="996e2b93101ac291.png"  width="190.71" /> <img src="img/b4a7c9b265d07362.png" alt="b4a7c9b265d07362.png"  width="176.50" /> <img src="img/5dc20d13d1f4d979.png" alt="5dc20d13d1f4d979.png"  width="181.42" />

これでテーブル作成とデータ挿入のステップは完了です。


## pgAdminを使用したクラスタへのアクセス
Duration: 06:00


pgAdminはPostgreSQLを管理するGUIのツールです。PostgreSQL互換のYugabyteDBでも使用することができます。

このセクションでは、pgAdminを使用してYugabyteDB Managedのクラスタにアクセスします。

pgAdminは [こちら](https://www.pgadmin.org/download/)からダウンロードできます。

インストールの詳細な手順については、 [SRA OSS社のブログ](https://www.sraoss.co.jp/tech-blog/pgsql/pgadmin4/)で紹介されていますので、参考にしてみてください。

1. アプリケーションからYugabyteDB Managedのクラスタにアクセスするには、SSL通信のための証明書が必要です。クラスターのダッシュボードを開いて、右上にある[Connect]ボタンをクリックしてください。

<img src="img/85c3a3a7747c77e7.png" alt="85c3a3a7747c77e7.png"  width="624.00" />

2. 表示されたウィンドウで、[Connect to your Application]の右にある矢印ボタンを押します。

<img src="img/fb9e5b3cbf91f1dc.png" alt="fb9e5b3cbf91f1dc.png"  width="624.00" />

3. [Download CA Cert]のリンクをクリックして、クラスタに接続するためのCA証明書をダウンロードしてください。

<img src="img/d8fd687cd3cd1d45.png" alt="d8fd687cd3cd1d45.png"  width="624.00" />

> aside negative
> 
> **Note:** この証明書はClient ShellやpgAdminのようなリモートのクライアントからクラスタにアクセスするために必要です。クラスターのダッシュボードから何度でもダウンロードは可能ですが、ファイルを保存した場所を忘れないようにしてください。

4. クラスタの接続パラメータをコピーし、ホスト名部分 (2行目の＠マーク以下、ybdb.ioまで）メモしておいてください。
5. pgAdminを開きます。Master passwordの入力を求められるので、初めて開く場合はパスワードを設定してください。

<img src="img/8346b078b9c549c3.png" alt="8346b078b9c549c3.png"  width="520.50" />

6. (オプション) pgAdminの画面を日本語表示にする場合は、メニューの[Preference(設定)]から設定画面を開き、[User language(ユーザ言語)]をJapaneseに設定します。

<img src="img/5681ced5812b5db1.png" alt="5681ced5812b5db1.png"  width="624.00" />

7. pgAdminの左上で[Servers]を右クリックし、[登録] &gt; [サーバ...]を選択してください。

<img src="img/89b4ee40521457f5.png" alt="89b4ee40521457f5.png"  width="396.45" />

8. [General]タブで、任意の名前を入力します。(接続にはホスト名を使用するので、実際のクラスター名を入力する必要はありません。）

<img src="img/a407a41d2eb8eec4.png" alt="a407a41d2eb8eec4.png"  width="395.56" />

9. [接続]タブで、[ホスト名/アドレス]に先ほどコピーしたホスト名を入力します。[ポート番号]は5433、[管理用データベース]はyugabyte、[ユーザ名]と[パスワード]には、クラスターのcredentialファイルに設定した情報を入力します。また、[パスワードを保存]をオンにしてください。

<img src="img/6ec610d7b8e91ee.png" alt="6ec610d7b8e91ee.png"  width="393.72" />

10. [SSL]タブで、[SSLモード]をVerify-Fullに設定します。また、先ほどダッシュボードからダウンロードしたCA証明書ファイルを[ロール認証]に設定してください。

<img src="img/d5a28f1a327c0ba2.png" alt="d5a28f1a327c0ba2.png"  width="396.50" />

11. [保存]ボタンをクリックして、ウィンドウを閉じます。
12. pgAdminの左側のリストに、登録したサーバが表示されます。データベースを選択し、[PSQLツール]をクリックしてください。

<img src="img/a693962ee840fbbb.png" alt="a693962ee840fbbb.png"  width="258.57" />

> aside negative
> 
> **Note:** pgAdminのバージョンによって、接続直後にyugabyteデータベースが表示される場合と、表示されない (postgres等、別のデータベースのみ表示される) 場合があります。

13. Yugabyteデータベースに接続できていない場合は、`\c yugabyte`と入力して、接続先を切り替えてください。

<img src="img/474f85a07b0d1d14.png" alt="474f85a07b0d1d14.png"  width="624.00" />

14. `\d` と入力すると、作成済みのテーブルが表示されます。

<img src="img/d64003b2d60da118.png" alt="d64003b2d60da118.png"  width="338.50" />

15. 前のステップで挿入したデータを確認してみましょう。select * from emp where deptno='20'; と入力してください。部門コードが20の従業員がリストされます。

<img src="img/5d66975f1795ff15.png" alt="5d66975f1795ff15.png"  width="624.00" />

以上で、pgAdminを使用したクラスタへのアクセスは完了です。


## (オプション) pgAdminでのデータベース作成
Duration: 05:00


pgAdminでは、前のステップで使用したPSQLツールだけでなく、GUIを使用してテーブルやデータを確認することができます。このセクションでは、pgAdminで新しいデータベース(northwind) を作成し、GUIの操作を確認します。

1. データベースにテーブルを作成するDDLファイル( [northwind_ddl.sql.txt](https://files.cdn.thinkific.com/file_uploads/540012/attachments/a36/29a/bd2/northwind_ddl.sql.txt))と、データを挿入するDMLファイル( [northwind_data.sql.txt](https://files.cdn.thinkific.com/file_uploads/540012/attachments/886/c8a/550/northwind_data.sql.txt))をダウンロードします。以下のリンクをクリックして新しいブラウザ・タブで開き、ブラウザの[ファイル] &gt; [別名で保管] (Chromeの場合、ブラウザによって異なります) から、テキストファイルを任意のローカルフォルダに保管してください。

<img src="img/e37a3a3cdd8e7dba.png" alt="e37a3a3cdd8e7dba.png"  width="264.50" />

2. pgAdminを開きます。左側のナビゲータで[データベース]を右クリックし、[作成] &gt; [データベース...]を選択してください。

<img src="img/79119e9e5538f6f1.png" alt="79119e9e5538f6f1.png"  width="365.50" />

3. [データベース]に northwind と入力し、[保存]ボタンをクリックしてください。

<img src="img/b354fd9ba605dfe7.png" alt="b354fd9ba605dfe7.png"  width="460.69" />

4. ナビゲータにnorthwindデータベースが表示されます。PSQLツールを再度開くか、すでに開いている場合は `\c northwind` と入力して、接続先データベースを切り替えてください。
5. 最初のステップでダウンロードしたDDLファイルをインポートして、テーブルを一括作成します。`\i ‘&lt;ダウンロードしたDDLファイルのパス&gt;'` と入力してください。(\iの後に半角スペース、ファイルパスはシングルクオート(')で囲みます。)

<img src="img/3da850d7235620ad.png" alt="3da850d7235620ad.png"  width="517.17" />

6. 同様に、ダウンロードしたDMLファイルをインポートして、データを一括挿入します。`\i ‘&lt;ダウンロードしたDMLファイルのパス&gt;'` と入力してください。
7. 左側のナビゲータで、[northwind] &gt; [スキーマ] &gt; [テーブル] と展開してください。上の手順で作成された14個のテーブルが確認できます。

<img src="img/8e341c26e6613b90.png" alt="8e341c26e6613b90.png"  width="336.00" />

8. employeeテーブルを右クリックして、[データを閲覧/編集] &gt; [すべての行]を選択してください。

<img src="img/ea439c7b2f4bc109.png" alt="ea439c7b2f4bc109.png"  width="476.21" />

9. 右側に、実行されたクエリとデータ出力が表示されます。

<img src="img/57c5dd5976e22711.png" alt="57c5dd5976e22711.png"  width="624.00" />

以上で、pgAdminでのデータベースやテーブルの作成は完了です。


## まとめ
Duration: 01:00


お疲れ様でした。YugabyteDB Managed入門ハンズオンは、これで終了です。

YugabyteDB Managedは、データベースを導入するハードウェアやOSを準備しなくても、数ステップの操作で使い始められるマネージドのデータベース・サービスです。

YugabyteDB Managedでは、CLIやGUIのツールを使って、使い慣れたSQLで操作できることを確認できたと思います。

### 次におすすめのハンズオン

以下のハンズオンも実施してみてください。

* YugabyteDB Managedチュートリアル (Cloud Shellから表示)
*  [YugabyteDB Managedの耐障害性と拡張性](https://yugabytedb-japan.github.io/codelabs/ybm-cluster-resiliency/index.html)

### 参考資料

*  [YugabyteDB Managed 公式ドキュメント](https://docs.yugabyte.com/preview/yugabyte-cloud/)
*  [Yugabyte University: YugabyteDB Managed Basics](https://university.yugabyte.com/courses/yugabytedb-managed-basics)


