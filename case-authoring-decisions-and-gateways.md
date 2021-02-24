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

  As part of the rule's action (the right-hand-side, or consequence, of the rule), the case data might change. For example, when the dispute is eligible for automated chargeback, the rule will change the `FraudData` fact/case file item by setting its `automated` property to `true`. Hence, we want to use a conditional gateway to decide whether we can do automatic approval or not.

1. Select the `Check for automated chargeback` Business Rule node, and click on the gateway icon.

    ![Business Central Case X-OR Gateway]({% image_path business-central-case-xor-gateway.png %}){:width="600px"}

2. Now, convert the `parallel gateway` to an `exclusive gateway`:

    ![Convert Gateway]({% image_path convert-xor-gateway.png %}){:width="600px"}

3. Add a new task to the gateway, by selecting the `Create Task` option: 

    ![New Task on Gateway]({% image_path new-task-xor-gateway.png %}){:width="600px"}

4. Convert this task to a `Script Task`: 

    ![Convert Task to Script Task]({% image_path generic-task-to-script-task.png %}){:width="600px"}

5. Double click the `Script Task` and name it as  `Automatic Approval`. 
6. Now, follow the same steps, and add another `Script Task` connected to the gateway, and name is `Manual Approval`. 
7. These serve only as placeholders, so they do not need to have a script implementation. 

  ![Business Central Case X-OR Gateway Tasks]({% image_path business-central-case-xor-gateway-tasks.png %}){:width="800px"}

3. Give one of the _Sequence Flows_ (the arrows connecting the nodes) the name `automatic` and the other the name `manual`. Not only is naming the sequence flows after a gateway a good practice, it also helps when selecting the default flow on the next step.

4. We can now implement the conditional expressions on the _Exclusive  Gateway_ and the _Sequence Flows_ connecting the gateway to the task nodes. We're first going to define the _Default gate_ of the gateway. *I.e. the path that should be taken when no conditions are met*. Select the _Exclusive Gateway_ node, open the properties panel on the right side of the editor and set the `Default gate` property to the `manual` flow.

    ![Business Central Case X-OR Gateway Default Gate]({% image_path business-central-case-xor-gateway-default-gate.png %}){:width="800px"}

4. Select the `automatic` _Sequence Flow_. In the properties panel, click on `Implementation/Execution` and select the `Expression` radio button.

5. Set the following expression in the _Script_ tab of the expression editor:
  
    * Expression: 

      ~~~
      return (caseFile_fraudData.getAutomated()) != null && (caseFile_fraudData.getAutomated() == true);
      ~~~

    The expression will activate when the `automated` field of the `fraudData` has been set (is not `null`) and the value is `true`.

    ![Business Central Sequence Flow Expression]({% image_path business-central-sequence-flow-expression.png %}){:width="800px"}

6. Save the case definition.

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
