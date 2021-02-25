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
    アドホックの自動開始: `true`
    データ入力と割り当て:
      Name: `Condition`
      データタイプ: `Boolean`
      ソース: `CaseData(data.get("approvedChargeback") == false)`

      ![Business Central Milestones Approved Rejected]({% image_path case-milestones-approved-rejected.png  %}){:width="800px"}

     `アドホックの自動開始` プロパティを設定すると、ケースが開始されたときにマイルストーンノードがアクティブになります。マイルストーンは条件を満たすと完了します。

3. 次に、ケースファイルの `approvedChargeback` の項目を `true` または `false` に設定するロジックを定義・設定する必要があります。自動承認の場合は、ルールを使ってこのロジックを実装したいと思います。手動処理の場合は、_ユーザータスク_ または _ヒューマンタスク_ と呼ばれる方法で、ユーザがこの値を設定できるようにしたいと考えています。自動承認のためのルールはすでに提供されており、`automatic-approval.drl` というDRL(Drools Rule Language)ファイルで定義されています。

    ![Case Automatic Approval Rule]({% image_path automatic-approval-rule-ss.png %}){:width="800px"}

    上のスクリーンショットに示すように、このルールは `FraudData` の `automated` フィールドが `true` に設定されている場合、`approvedChargeback` ケースファイルの項目を `true` に設定します。これは非常に基本的なルールであり、自動承認の対象となるすべてのチャージバック申請に対してこれを承認するものです。このことは、ルールを使用してケースファイル内のデータを操作する方法と、より高度なルールでこの機能を利用する方法を示しています。

    また、_ruleflow-group_ の名前が `automatic-approval` に設定されていることにも注意してください。この _ruleflow-group_ が、_ビジネスルールタスク_ を使用するために必要になります。

4. ルールを使用するには、`Automatic Approval `スクリプトタスクを _ビジネスルールタスク_ に変更し、_ruleflow-group_ プロパティを`automatic-approval`に設定します。

    ![Change Script Task to Business Rule Task]({% image_path change-script-task-to-business-rule-task.png %}){:width="800px"}

    ![Change Script Task to Business Rule Task]({% image_path case-rule-task-automatic-approval.png %}){:width="800px"}

5. `Automatic Approval` ビジネスルールタスクに次のデータ割り当てを設定します。

    * データ入力と割り当て

    | Name  | データタイプ | ソース |
    |:--:|:--:|:--:|
    | htCreditCardHolder | CreditCardHolder | caseFile_creditCardHolder |
    | htFraudData | FraudData | caseFile_fraudData |

    * データ出力と割り当て

    | Name  | データタイプ | ターゲット |
    |:--:|:--:|:--:|---|---|
    | htApprovedChargeback | Boolean | caseFile_approvedChargeback |
    | brFraudData | FraudData | caseFile_fraudData |

    ![User Task Manual Approval Data IO]({% image_path user-task-manual-approval-data-io.png %}){:width="800px"}

6. フローの手動承認の部分では、まず、前のモジュールで定義したリスク算出ルールを適用したいと思います。これにより、ユーザーがチャージバック申請のリスクを評価するためのリスクスコアリング情報が作成されます。`Manual Approval` スクリプトタスクを _ビジネスルールタスク_ に変換し、_ruleflow-group_ を `risk-evaluation` に設定します。これを `Credit Risk Evaluation` と命名します。

    ![Case Rule Risk Evaluation]({% image_path case-rule-risk-evaluation.png %}){:width="800px"}

7. `Credit Risk Evaluation` ビジネスルールタスクに以下のデータ割り当てを設定します:

    * データ入力と割り当て

    | Name  | データタイプ | ソース |
    |:--:|:--:|:--:|
    | brCreditCardHolder | CreditCardHolder | caseFile_creditCardholder |
    | brFraudData | FraudData | caseFile_fraudData |

    * データ出力と割り当て

    | Name  | データタイプ | ターゲット |
    |:--:|:--:|:--:|
    | brCreditCardHolder | CreditCardHolder | caseFile_creditCardholder |
    | brFraudData | FraudData | caseFile_fraudData |

    ![Business Rule Data Input Output]({% image_path business-rule-data-input-output-mapping.png %}){:width="800px"}

8. 次に、実際のユーザタスクを定義します。`Credit Risk Evaluation` ルールタスクにリンクした _ユーザータスク_ ノードを作成します。`Credit Risk Evaluation` をクリックし、新しいタスクを追加し、汎用タスクをユーザタスクに変換します。

	![User Task Manual Approval Data Input Output]({% image_path new-user-task.png %}){:width="600px"}


10. ユーザータスクを以下のように設定します:

    * Name: `Manual Approval`  
    * タスク名: `manual_approval`  
    * アクター: `pamAdmin`  

    また、以下のデータ割り当てを設定します。

    * データ入力と割り当て

    | Name  | データタイプ | ソース |
    |:--:|:--:|:--:|---|---|
    | htCreditCardHolder | CreditCardHolder | caseFile_creditCardHolder |
    | htFraudData | FraudData | caseFile_fraudData |

    * データ出力と割り当て

    | Name  | データタイプ | ターゲット |
    |:--:|:--:|:--:|---|---|
    | htApprovedChargeback | Boolean | caseFile_approvedChargeback |

    ![User Task Manual Approval Data Input Output]({% image_path user-task-manual-approval-data-io.png %}){:width="600px"}

    ![User Task Manual Approval Data Input Output]({% image_path case-usertask-manual-approval.png %}){:width="600px"}

9. Business Central でこのユーザータスクを処理するためのフォームを自動生成してみましょう。

   ![User Task Manual Approval Data Input Output]({% image_path generate-forms.png %}){:width="350px"}

10. 次に、エンドノードを追加してみましょう。`Chargeback Approved` と `Dispute Rejected` の両方のマイルストーンの後に `終了終端イベント` を追加します。これにより、いずれかのマイルストーンが満たされたときにプロセスとすべてのオープンタスクとマイルストーンが終了するようになります。
  ![Case Full Implementation]({% image_path case-full-implementation.png %}){:width="800px"}

10. 作業を保存してください。

これでクレジットカードのチャージバック申請処理の案件の実装は完了です。

# ケースをテストする

クレジットカードチャージバック申請処理の最新版を試してみましょう! 
プロジェクトを実行サーバにデプロイして、動作を確認してみましょう。

1. デプロイするには、_アセットライブラリ_ ビューに戻り、_デプロイ_ ボタンをクリックしてプロジェクトをデプロイします。

1. 手動での承認が必要なデータでケースを開始します。クレジットカード所有者のステータスが _Silver_ であれば、どのようなケースでも手動処理となります。
      
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

3.  ケースのプロセスインスタンスダイアグラムを開き、`Manual Approval` のユーザタスクでプロセスが待機していることを確認します。

    ![Case Wait State Manual Approval]({% image_path case-wait-state-manual-approval.png %}){:width="800px"}

4. Business Central で、_メニュー -> トラック -> タスク受信箱_ を選択します。全てが正常であれば、開始したばかりの案件のユーザータスクが受信箱に入っているはずです。

    ![Case Task Inbox]({% image_path case-task-inbox.png %}){:width="800px"}

5. タスクをクリックして開いてください。ルールで決定された _Risk Rating_ を含む、チャージバック申請の詳細が表示されます。チャージバックを承認するには、_Approve Chargeback_ チェックボックスを有効にし、拒否するには無効にしてください。_完了_ ボタンをクリックしてタスクを完了させます。

    ![Case User Task Approve]({% image_path case-user-task-approve.png %}){:width="800px"}

6. _プロセスインスタンス_ リストに戻ると、先ほどのプロセス/ケースが表示されなくなっているはずです。画面左側のフィルターセクションで、_完了_ を有効にします。

    ![Process Instance List Completed Filter]({% image_path process-instance-list-completed-filter.png %}){:width="800px"}

7. リストの中に自分のプロセス/ケースが表示されます。それを選択すると、プロセスインスタンスの詳細画面が開きます。ダイアグラム_タブを開きます。この画面では、_Chargeback Approved_ マイルストーンが完了しているので、チャージバック申請が承認されていることに注意してください。

    ![Case Completed Dispute Approved]({% image_path case-completed-dispute-approved.png %}){:width="800px"}

    **Note**: ダイアグラムでは `Dispute Rejected` マイルストーンもグレーアウトしています。しかし、これは `Chargeback Approved` マイルストーンの後の _Termination_ ノードが `Dispute Rejected` マイルストーンを含むプロセス内のすべてのアクティブなノードを終了させたためです。この図を見ると、`Dispute Rejected` マイルストーンの後の _終了終端_ ノードに到達していないことがわかります。

あなたはこのワークショップのこのモジュールを終了しました。
マイルストーン、シグナル、ビジネスルール、ゲートウェイ、ユーザータスク、ターミネーターを含む、ケース定義を構築する方法を学びました。
ケース管理が従来のBPMとどのように違うのか、ケースが動的で完全にデータ駆動型であり、ルール、データ、イベントに依存してケースの実行を駆動するという点を確認しました。
