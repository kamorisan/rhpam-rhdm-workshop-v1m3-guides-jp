# 9. ケースにデシジョンとゲートウェイを統合する

このセクションでは、ケース管理の定義の中でデシジョンのアセットを使用する方法を学びます。

## ユースケース

このセクションでは、定義したビジネスルールに基づいてチャージバック申請を自動承認するシナリオの自動化に取り組んでいきます。チャージバック処理の自動化について、もう少し理解してみましょう:

### チャージバック申請の自動承認

請求エラーをめぐるクレジットカードのチャージバック申請は、顧客に有利に解決される可能性が高いです。
これは、カードの使用中に収集されたさまざまなデータが原因です。
例えば、トランザクションの金額、または顧客のカード種別によって、チャージバック申請の自動承認の対象となる可能性があります。

プロセスは以下のようになります:

1. カード所持者がチャージバック申請を開始します

2. ケースの情報をビジネスルールで自動評価し、チャージバック申請自動承認の判定を行います

3. カード発行者は、チャージバック対象金額をあなたの口座に入金します

では、自動化プロジェクトに戻りましょう。

チャージバック申請の処理の種類を決定するために、`automated-chargeback` のガイド付きルールで実装されている、チャージバック申請自動処理のルールを使用します。

NOTE: _問題が見つかり、前のステップを完了した状態でプロジェクトをインポートしたい場合は、`ccd-project` プロジェクトを削除し、以下の URL を使用してインポートし直してください。: [https://github.com/kamorisan/rhpam-rhdm-workshop-v1m3-labs-step-3.git](https://github.com/kamorisan/rhpam-rhdm-workshop-v1m3-labs-step-3.git)._

### ケースの中でビジネスルールを使う

チャージバック申請の承認を自動で行うかどうかの判断は、チャージバック申請が始まってからの最初のステップなので、`Dispute received` がトリガーされた直後にステップを追加することにします。
マイルストーンはアクションを実行するものではなく、目標を達成したことを示すものであることに注意してください。
しかし、これらのマイルストーンノードにはタスク等をリンクすることができます。
これらのノードはマイルストーンが完了した後に開始されます。

1. （以前に作業をした） _ccd-project_ の _chargeDispute_ アセットを開きます。

2. "Dispute Received” マイルストーンをクリックして、新しいタスクを追加します: 

   ![Business Central Guided Rule Automated Chargeback Ruleflow Group]({% image_path dispute-milestone-new-task-node.png %}){:width="450px"}

3. さて、新しいタスクを `ビジネスルール` タスクに変換します(タスク左下の 歯車アイコンをクリックして選択します)。

   ![Business Central Guided Rule Automated Chargeback Ruleflow Group]({% image_path convert-new-task-node-to-business-rules.png %}){:width="450px"}

4. ビジネスルールタスクをダブルクリックして、名前を `Check for automated chargeback` に変更します。
   
5. 右パネルの `ビジネスルール` プロパティで、 `実装/実行` セクションのルールフローグループの値を更新します:

      - ルールフローグループ: `automated-chargeback`

6. 作業を保存します。それでは、ルールに照らし合わせて評価するケースデータを送信してみましょう。

7. The automated chargeback rule evaluates the `CreditCardHolder` and the `FraudData`. So, we need to insert these 2 _case file items_ in the `Working Memory` of the rules engine. Click on the "Assignments" option of your business rule:チャージバック申請の自動承認ルールは `CreditCardHolder` と `FraudData` を評価します。そのため、ルールエンジンの `Working Memory` にこれら2つのケースファイル項目を挿入する必要があります。ビジネスルールタスクの `割当` オプションをクリックします。

	![Edit rules task data input/output]({% image_path edit-rules-task-data.png %}){:width="800px"}

1. 以下のように設定します:

    - データ入力と割り当て:

    | Name            | Data Type     | Source       |
    |:---------------|:-------------|:-------------|
    | brCreditCardHolder  | CreditCardHolder |caseFile_creditCardHolder |
    | brFraudData | FraudData  | caseFile_fraudData |

    - データ出力と割り当て:

    | Name            | Data Type     | Source       |
    |:---------------|:-------------|:-------------|
    | brCreditCardHolder  | CreditCardHolder |caseFile_creditCardHolder |
    | brFraudData | FraudData  | caseFile_fraudData |

    - 以下のようになっているはずです:

    ![Business Rules Task Assignment]({% image_path business-rules-task-data-assignment.png %}){:width="600px"}

9. 作業を保存します。現時点で、ケースは以下のようになっています。

  ![Business Central Case First Business Rule Node]({% image_path business-central-case-first-business-rule-node.png %}){:width="600px"}



では、判定結果を元に、チャージバック申請の自動承認処理を追加してみましょう。



## ゲートウェイを使う

  ルールのアクション(ルールの右側、つまり結果)の一部として、ケースデータが変更されることがあります。例えば、チャージバック申請がが自動承認の対象となる場合、ルールは `FraudData` の `automated` プロパティを `true` に設定することで、`FraudData` のファクト/ケースファイル項目を変更します。したがって、自動承認を行うかどうかは条件付きゲートウェイを使って判断をすることにします。

1. `Check for automated chargeback` のビジネスルールノードを選択し、ゲートウェイのアイコンをクリックする。

    ![Business Central Case X-OR Gateway]({% image_path business-central-case-xor-gateway.png %}){:width="600px"}

2. ここで、`並行ゲートウェイ` を `排他的ゲートウェイ` に変換する。

    ![Convert Gateway]({% image_path convert-xor-gateway.png %}){:width="600px"}

3. 新しいタスクをゲートウェイに追加するには、`作成 Task` オプションを選択します。

    ![New Task on Gateway]({% image_path new-task-xor-gateway.png %}){:width="600px"}

4. このタスクを `スクリプトタスク` に変換する。

    ![Convert Task to Script Task]({% image_path generic-task-to-script-task.png %}){:width="600px"}

5. `スクリプトタスク` をダブルクリックし、`Automatic Approval` という名前を付けます。
6. 同様の手順で、ゲートウェイに接続されている別の `スクリプトタスク` を追加し、名前を `Manual Approval` とします。
7. これらはプレースホルダーとしての仮のタスクなので、スクリプトの実装は必要ありません。

  ![Business Central Case X-OR Gateway Tasks]({% image_path business-central-case-xor-gateway-tasks.png %}){:width="800px"}

3. _シーケンスフロー_ (ノードを結ぶ矢印)には、片方に `automatic` 、もう片方には `manual` という名前をつけます。ゲートウェイからのシーケンスフローに名前をつけることは、次のステップでデフォルトフローを選択する際に必要になります。

4. これで、_排他的ゲートウェイ_ と、ゲートウェイとタスクノードを接続する _シーケンスフロー_ に条件式を実装することができるようになりました。まず、ゲートウェイの _デフォルトルート_ （条件が満たされていないときのパス）を定義します。_排他的ゲートウェイ_ ノードを選択し、エディタの右側にあるプロパティパネルを開き、`デフォルトルート` プロパティを`manual` フローに設定します。

    ![Business Central Case X-OR Gateway Default Gate]({% image_path business-central-case-xor-gateway-default-gate.png %}){:width="800px"}

4. _automatic_ シーケンスフロー_を選択します。プロパティパネルで `実装/実行` をクリックして `式` のラジオボタンを選択する。

5. 式エディタで以下の式を設定します。
  
    * 式: 

      ~~~
      return (caseFile_fraudData.getAutomated()) != null && (caseFile_fraudData.getAutomated() == true);
      ~~~

    この式は、`fraudData` の `automated` フィールドに `null` ではなく `automated` が設定されていて値が `true` の場合にアクティブになります。

    ![Business Central Sequence Flow Expression]({% image_path business-central-sequence-flow-expression.png %}){:width="800px"}

6. ケース定義を保存します。

## Testing the case definition

Let's try the business decision and nodes within the case we just updated.

1. Deploy the project to the execution server by clicking on the _Deploy_ button from the _Asset Library_ perspective.

2. Start a new case instance like we did in the previous lab, using the KIE Server Swagger UI. Use the same input data. 

    * URL: `POST /server/containers/{containerId}/cases/{caseDefId}/instances`
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

3. Back on Business Central, Open the diagram of the Case/Process Instance and note that the dispute you've entered requires manual approval.
  ![Case With Placeholders Manual Approval]({% image_path case-with-placeholders-manual-approval.png %}){:width="800px"}

3. Start a new case instance, but this time set the Credit Card Holder's status to `Gold`. 

      * containerId: `ccd-project`

      * case definition: `ccd-project.ChargeDispute` 

      * Body: 

        ````
        {
          "case-data" : {
            "creditCardholder": {
                "CreditCardHolder": {
                  "age": 42,
                  "status": "Gold"
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


6. This should cause the rules to make the dispute eligible for automatic processing. Open the diagram of the Case/Process Instance and observe that the case has indeed taken the path of automatic processing.

  ![Case With Placeholders Automatic Approval]({% image_path case-with-placeholders-automatic-approval.png %}){:width="800px"}

You have just learned how to leverage the Decisions and Rules you authored in the previous scenario in your case definition. You have seen how the state of the data, in this case the Card Holder's status, triggers rules. You've seen how the rules manipulate the state of the data, in this case setting the `automatic` field of the `FraudData` to `true`, which can drive decisions and flow directions within our case.

Apart from changing data, decisions and rules can also infer and create new data, as well as remove data from the case instance. Through decisions and rules, the data-driven approach of Case Management (in contrast to the flow driven approach of traditional BPM) allows for the implementation of very dynamic, data-driven, case logic.
