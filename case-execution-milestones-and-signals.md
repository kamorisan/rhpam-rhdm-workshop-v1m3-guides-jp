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

     ````
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
     ````

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

## Validating your case using Case Management Showcase App

The platform provides a _Case Management Showcase_ application, which is an **example** of how a custom Case Management dashboard could be composed. Let's test your process using it.

1. Open this application by clicking on the _Application Launcher_ button on the top right of the screen. Next, click on _Case Management Showcase_.

    ![Business Central Case Management Showcase Button]({% image_path business-central-cm-showcase-button.png %}){:width="800px"}

2. Login with the same credentials that you used for the Business Central Workbench (u:`pamAdmin`, p:`redhatpam1!`).

3. Locate the blue button `Start Case` and click on it. We will now start a new instance of your Charge Dispute process.

4. A new pop-up should show up. Fill in the `Case Owner` field with value `pamAdmin`, and also the `Users` for the `Role Name` `approval-manager`. Your form should look like this:

  ![Case Management Showcase ChargeDispute List]({% image_path showcase-app-new-case-popup.png %}){:width="800px"}

5. Click on `Start`. The showcase application will list the opened case. You will see the case you've just started in the list:

    ![Case Management Showcase ChargeDispute List]({% image_path cms-chargedispute-case-list.png %}){:width="800px"}

4. Click on the case to open a detailed view of the open case. This view will, among other things, list the completed milestones, the available actions, the actions that are in progress, the completed actions and the case roles. Take some time to explore the details of your case.

    ![Case Management Showcase ChargeDispute Details]({% image_path cms-chargedispute-case-details.png %}){:width="800px"}

# Conclusion

So far we've configured the initial process, a `signal end-event` that triggers the `milestone`, and a `milestone` conditional expression which fired based on the `case data`.

In the next section we will add our rules, decisions and user-tasks to our project.
