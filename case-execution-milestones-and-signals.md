# 7. ケース管理アプリケーションの実行

このセクションでは、先ほど作成したケースプロジェクトをテストします:

1. プロジェクトをデプロイします

2. ケース管理の Rest API について

3. Business central を使用してケースの進捗状況をトレースします

## ケースプロジェクトの特別な設定

プロジェクトをデプロイする前に、プロジェクトがケースプロジェクトとして正しく設定されていることを確認してください。

1. プロジェクトの `設定` を開き、オプション `デプロイメント`、 `一般設定` にアクセスします。
2. ランタイムストラテジーが `ケース別` に設定されていることを確認してください。
  ![Project  General Settings]({% image_path ccd-project-general-settings.png%}){:width="700px"}

2. 次に、 `マーシャリングストラテジー` メニューを開きます。プロジェクトに両方のストラテジーが設定されていることを確認してください。

    * `org.jbpm.casemgmt.impl.marshalling.CaseMarshallerFactory.builder().withDoc().get();`
    * `new org.jbpm.document.marshalling.DocumentMarshallingStrategy();`

  ![Project Marshalling Strategies]({% image_path ccd-project-marshalling-strategies.png%}){:width="700px"}

必要に応じて、プロジェクトの設定を上記のものに合わせて修正し、プロジェクトを保存します。

## プロジェクトのデプロイ

では、新しいケースを起動してみましょう。ケースをテストをするためには、実行サーバにデプロイする必要があります。

1. Business Central で、プロジェクトの _アセットライブラリ_ ビューを開きます。
2. 画面の右上隅にある _デプロイ_ ボタンをクリックします。これにより、プロジェクトがパッケージ化され、実行サーバにデプロイされます。
3. ワークベンチには、ビルドとデプロイが成功したことを示す2つの緑色の通知バーが表示されます。

## REST API 経由で新規にケースを開始する 

プロセスエンジンの柔軟性を確認するために、Rest API 経由でケースの新しいインスタンスを起動してみましょう。KIE Serverでは、利用可能なAPIを一通り提供しているので、試してみることができます。
まずはKIE ServerのSwagger UIを開いてみましょう。

1. Business Centralメニューで、デプロイメニューの、`実行サーバー` にアクセスします。

2. 次に、リモートサーバーをクリックして、そのURLを新しいタブで開きます。

   ![KIE Server URL]({% image_path kie-server-url.png%}){:width="600px"}

3. Swagger UIにアクセスしてみましょう。あなたの URLを "http://your-url/docs" のように変更します。次のようになります。 http://insecure-rhpam7-kieserver-rhpam-user1.apps.cluster-e3df.e3df.example.opentlc.com/docs
4. **Case Instances** セクションを確認します。このセクションの、以下のAPIをクリックします:
   * `POST /server/containers/{containerId}/cases/{caseDefId}/instances` *Starts a new case instance for a specified definition*

5. `Try it Out` をクリックし、以下のデータを入力します：

   * containerId: `ccd-project`

   * case definition: `ccd-project.ChargeDispute` 

   * Body: 

     〜〜〜
     {
       "case-data" : {
         "creditCardholder": {
             "CreditCardHolder": {
               "age": 42,
               "status": "Standard"
             }
         }, 
           "fraudData": {
             "FraudData": {
               "totalFraudAmount": 49
             } 
           }
       },
       "case-user-assignments" : {
         "owner" : "pamAdmin",
         "approval-manager" : "pamAdmin"
       },
       "case-group-assignments" : { },
       "case-data-restrictions" : { }
     }
     〜〜〜

     ![Swagger UI Start Case 1]({% image_path swagger-ui-start-case-1.png %}){:width="800px"}

6. Response Bodyが、"CASE-0000000001” のような、自動生成されたケースIDとなっている、201 Response を取得する必要があります。

**TIP:** Business CentralとKIE Server Swagger UIを2つの異なるタブで開いておくと、作業がしやすくなります。

ここで、このケース インスタンスの進行状況をよりよく可視化するために、Business Central でケース インスタンスを確認してみましょう。

## Business Central でケースインスタンスをトレースする

Business Central は、各プロセスインスタンスとタスクの詳細を表示するモニタービューを提供しており、これらの項目と対話することもできます。

- プロセスの開始、中止、監視。
- ヒューマンタスク（クレーム、開始など）のリスト化、可視化、管理
- プロセスの各ステップのログの確認

プロセスエンジン（別名：Kie Server）を管理・監視しているBusiness Centralを使って、起動したケースインスタンスの詳細を見てみましょう。

1. Business Central タブに戻り、管理メニューの `プロセスインスタンス` を開きます。

   ![Process Instances 1]({% image_path bc-process-instances.png %}){:width="600px"}

    1つのインスタンスがリストアップされているはずです。Swagger UI経由でさらにインスタンスを起動すると、さらに多くのインスタンスが表示されるはずです。表示されているケース・インスタンスをクリックします。

    ![Business Central Case Instances Charge Dispute]({% image_path business-central-case-instances-chargedispute.png %}){:width="800px"}

2. _ダイアグラム_ タブをクリックすると、ケースのダイアグラムが表示されます。マイルストーンの色がグレーになっていることに注意してください。これは、( _終了シグナルイベント_ によって)トリガーされただけでなく、( `true` に評価された _Condition_ によって)完了したことを意味しています。

    ![Start Charge Dispute Diagram]({% image_path start-chargedispute-diagram.png %}){:width="800px"}

10. _プロセス変数_ をクリックすると、ケースの現在の状態/値の概要が表示されます。`fraudData` が設定されていることに注意してください。

    ![Start Charge Dispute Case Variables]({% image_path start-chargedispute-case-variables.png %}){:width="800px"}

## ケース管理のショーケースアプリを使って、ケースを検証する

このプラットフォームは _Case Management Showcase_ アプリケーションを提供しており、カスタムのケース管理ダッシュボードをどのように構成することができるかの **例** です。これを使ってプロセスをテストしてみましょう。

1. 画面右上の _Application Launcher_ ボタンをクリックし、_ケース管理ショーケース_ をクリックします。

    ![Business Central Case Management Showcase Button]({% image_path business-central-cm-showcase-button.png %}){:width="800px"}

2. Business Central ワークベンチ で使用したのと同じ資格情報 (user:`pamAdmin`, password:`redhatpam1!`) でログインします。

3. 青いボタン `ケース開始` をクリックしてください。これで、あなたのチャージバック申請処理の新しいインスタンスが開始されます。

4. 新しいポップアップウィンドウが表示されるはずです。`ケース所有者` フィールドに値 `pamAdmin` を入力し、`ロール名` `approval-manager` には `Users` を入力してください。フォームは次のようになります:

  ![Case Management Showcase ChargeDispute List]({% image_path showcase-app-new-case-popup.png %}){:width="800px"}

5. `スタート`をクリックします。ショーケースアプリケーションが開いているケースを一覧表示します。リストの中には、今始めたばかりのケースが表示されます:

    ![Case Management Showcase ChargeDispute List]({% image_path cms-chargedispute-case-list.png %}){:width="800px"}

6. ケースをクリックすると、開いているケースの詳細ビューが表示されます。このビューには、完了したマイルストーン、利用可能なアクション、進行中のアクション、完了したアクション、ケースの役割などが表示されます。ケースの詳細について確認してみてください。

    ![Case Management Showcase ChargeDispute Details]({% image_path cms-chargedispute-case-details.png %}){:width="800px"}

# おわりに

ここまでで、初期プロセスと、`マイルストーン` のトリガーとなる `終了シグナルイベント` と、`ケースデータ` に基づいて発生する `マイルストーン` の条件式を設定した。

次のセクションでは、ルール、デシジョン、ユーザータスクをプロジェクトに追加します。
