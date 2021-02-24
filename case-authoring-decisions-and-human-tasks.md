# 10. ケースにヒューマンタスクを統合する

さて、この最後のセクションでは、以下のタスクを実施します。

1. ケースモデルをさらに充実させる方法
2. ケースモデルにヒューマンタスクを統合する。

## クレジットカード・チャージバック申請のケース

![Business Central CC Dispute Diagram Users]({% image_path business-central-cc-dispute-diagram-users.png %}){:width="400px"}

前のセクションで見たように、ビジネスルールを通じて、どのチャージバック申請が自動承認の対象となり、どのチャージバック申請が手動処理を必要とするかを定義しました。

## チャージバック申請の自動承認

これで、私たちのケースでは、チャージバック申請が自動的に承認されるのか、それとも手動の承認ステップが必要なのかを判断できるようになったので、
実際の承認ロジックと、チャージバック申請が承認されたか否かをトレースする _マイルストーン_ を実装することができます。

まず、_マイルストーン_ とその条件を作成してみましょう。
ケースファイルには `approvedChargeback` という `Boolean` 型 の _ケースファイル項目_ が含まれています。
このケースファイル項目をマイルストーンの条件式で使用します。

1. 以下の特性を持つ新しい _マイルストーン_ ノードを作成します。

    Name: `Chargeback Approved`
    アドホックの自動開始: `true`
    データ入力と割り当て:
      Name: `Condition`
      データタイプ: `Boolean`
      ソース: `CaseData(data.get("approvedChargeback") == true)`

2. Create a second _Milestone_ with the following characteristics:

    Name: `Dispute Rejected`  
    Condition: `CaseData(data.get("approvedChargeback") == false)`  

    Condition Data Type: `Boolean`

    Adhoc autostart: `true`

      ![Business Central Milestones Approved Rejected]({% image_path case-milestones-approved-rejected.png  %}){:width="800px"}

    Setting the `Adhoc autostart` property activates these milestone nodes when the case is started. The milestones are completed when their condition is met.

3. We now need to define and configure the logic that sets the `approvedChargeback` case file item to `true` or `false`. In the case of _automatic_ processing, we want to implement this logic using rules. In the case of _manual_ processing, we want to allow a user to set this value via a so called _User Task_ or _Human Task_. We've already provided the rules for the automatic approval, which are defined in a DRL (Drools Rule Language) file called `automatic-approval.drl`.

    ![Case Automatic Approval Rule]({% image_path automatic-approval-rule-ss.png %}){:width="800px"}

    As showed in the screenshot above, the rule sets the `approvedChargeback` case file item to `true` when `automated` field of `FraudData` is set to `true`. Obviously, this is a very basic rule which in essence approves the chargeback for every dispute that is eligible for automated approval. Nevertheless, it demonstrates how rules can be used to manipulate data in the case file, and how more advanced rules can make use of this functionality.

    Also note that the name of the _ruleflow-group_ is set to `automatic-approval`. We need this _ruleflow-group_ and use it in our _Business Rule Task_.

4. To use the rule, change the `Automatic Approval` script task into a _Business Rule Task_ and set its _ruleflow-group_ property to `automatic-approval`.

    ![Change Script Task to Business Rule Task]({% image_path change-script-task-to-business-rule-task.png %}){:width="800px"}

    ![Change Script Task to Business Rule Task]({% image_path case-rule-task-automatic-approval.png %}){:width="800px"}

5. Configure the following Data Assignments for the Automatic Approval Business Rule Task:

    * Data Inputs and Assignments

    | Name  | Data Type | Source |
    |:--:|:--:|:--:|
    | htCreditCardHolder | CreditCardHolder | caseFile_creditCardHolder |
    | htFraudData | FraudData | caseFile_fraudData |

    * Data Outputs and Assignments

    | Name  | Data Type | Target |
    |:--:|:--:|:--:|---|---|
    | htApprovedChargeback | Boolean | caseFile_approvedChargeback |
    | brFraudData | FraudData | caseFile_fraudData |

    ![User Task Manual Approval Data IO]({% image_path user-task-manual-approval-data-io.png %}){:width="800px"}

6. For the manual approval part of the flow, we first want to apply the credit risk scoring rules that we've defined in the previous module. This will create the risk scoring information for the user to assess the risk of the dispute. Convert the `Manual Approval` script task into a _Business Rule Task_ and set its _ruleflow-group_ to `risk-evaluation`. Name it `Credit Risk Evaluation`.

    ![Case Rule Risk Evaluation]({% image_path case-rule-risk-evaluation.png %}){:width="800px"}

7. Configure the following Data Assignments for the `Credit Risk Evaluation` Business Rule Task:

    * Data Inputs and Assignments

    | Name  | Data Type | Source |
    |:--:|:--:|:--:|
    | brCreditCardHolder | CreditCardHolder | caseFile_creditCardholder |
    | brFraudData | FraudData | caseFile_fraudData |

    * Data Outputs and Assignments

    | Name  | Data Type | Target |
    |:--:|:--:|:--:|
    | brCreditCardHolder | CreditCardHolder | caseFile_creditCardholder |
    | brFraudData | FraudData | caseFile_fraudData |

    ![Business Rule Data Input Output]({% image_path business-rule-data-input-output-mapping.png %}){:width="800px"}

8. Next, we want to define the actual user task. Create a _User Task_ node linked to the `Credit Risk Evaluation` rule task. Click on the `Manual Task` and add a new task. Convert the generic task into a User Task. 

	![User Task Manual Approval Data Input Output]({% image_path new-user-task.png %}){:width="600px"}


10. Configure the task as follows:

    * Name: `Manual Approval`  
    * Task Name: `manual_approval`  
    * Actors: `pamAdmin`  

    Also, configure the following data assignments:

    * Data Inputs and Assignments

    | Name  | Data Type | Source |
    |:--:|:--:|:--:|---|---|
    | htCreditCardHolder | CreditCardHolder | caseFile_creditCardHolder |
    | htFraudData | FraudData | caseFile_fraudData |

    * Data Outputs and Assignments

    | Name  | Data Type | Target |
    |:--:|:--:|:--:|---|---|
    | htApprovedChargeback | Boolean | caseFile_approvedChargeback |

    ![User Task Manual Approval Data Input Output]({% image_path user-task-manual-approval-data-io.png %}){:width="600px"}

    ![User Task Manual Approval Data Input Output]({% image_path case-usertask-manual-approval.png %}){:width="600px"}

9. Let's automatically generate the user task that will allow us to interact with this human task in Business Central.

   ![User Task Manual Approval Data Input Output]({% image_path generate-forms.png %}){:width="350px"}

9. Let's add the end nodes in our case. Add a _Terminating End Event_ after both the `Chargeback Approved` and `Dispute Rejected` milestone. This makes sure that the process, and all open tasks and milestones are terminated when either one of these milestones is met.
  ![Case Full Implementation]({% image_path case-full-implementation.png %}){:width="800px"}

10. Save the process.

You've now completed the full implementation of the Credit Card Dispute case.

# Trying out your case

It's time to try out your latest version of the Credit Card Dispute process! Let's Deploy the project to the Execution Server and see it working.

1. To deploy it, go back to the _Assets Library_ view and click on the _Deploy_ button to deploy the project.

1. Start a case with the data that would require manual approval. Any case with a Credit Card Holder having a _Silver_ status will do.
      
    * API: `POST /server/containers/{containerId}/cases/{caseDefId}/instances` *Starts a new case instance for a specified definition*
    * containerId: `ccd-project`
    * case definition: `ccd-project.ChargeDispute` 
    * Body: 

      ````
      {
        "case-data" : {
          "creditCardholder": {
              "CreditCardHolder": {
                "age": 42,
                "status": "Silver"
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

3.  Open the process instance diagram of the case and observe that the process is waiting in the `Manual Approval` user task.

    ![Case Wait State Manual Approval]({% image_path case-wait-state-manual-approval.png %}){:width="800px"}

4. In the workbench, go to _Menu -> Track -> Task Inbox_. If everything is correct, the user task of the case you've just started should be in your inbox:

    ![Case Task Inbox]({% image_path case-task-inbox.png %}){:width="800px"}

5. Click on the task to open it. You will see some of the details of the dispute, including the _Risk Rating_ that is determined by our rules. Enable the _Approve Chargeback_ checkbox to approve the dispute, or keep it disabled to reject it. Click on the _Complete_ button to complete the task:

    ![Case User Task Approve]({% image_path case-user-task-approve.png %}){:width="800px"}

6. Go back to the _Process Instance_ list and notice that your process/case is no longer listed. In the filter section on the left-hand-side of the screen, enable _Completed_

    ![Process Instance List Completed Filter]({% image_path process-instance-list-completed-filter.png %}){:width="800px"}

7. You will see your process/case in the list. Select it to open the process instance details screen. Open the _Diagram_ tab. Note that the dispute has been approved, as the _Chargeback Approved_ milestone has been completed.

    ![Case Completed Dispute Approved]({% image_path case-completed-dispute-approved.png %}){:width="800px"}

    **Note**: The `Dispute Rejected` milestone is greyed out as well in the diagram. This is however due to the _Termination_ node after the `Chargeback Approved` milestone, which has terminated all active nodes in the process, including the `Dispute Rejected` milestone. You can see in the diagram that the _Termination_ node after the `Dispute Rejected` milestone was not reached.

You have successfully finished this part of the workshop. You've learnt how to build a full case definition containing milestones, signals, business rules, gateways, user tasks and terminators. You've seen how Case Management differs from traditional BPM in the sense that cases are dynamic and fully data-driven, relying on rules, data and events to drive the case execution.
