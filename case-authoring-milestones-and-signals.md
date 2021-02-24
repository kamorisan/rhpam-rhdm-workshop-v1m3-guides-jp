# 6. ケースの作成 - マイルストーンとシグナル

このセクションでは、以下の内容について説明します:

1. マイルストーンとは何か、どのような時に使うのか

2. マイルストーンを使ってケースをモデル化する方法

3. シグナルを使ってマイルストーンをアクティブにする方法。

4. マイルストーンを完了させる方法。

## ケースのマイルストーン

ケースインスタンスの進捗をトレースするためには、顧客に関連するマイルストーンを定義する必要があります。
マイルストーンは、重要な目標が達成されたことをクレジットカード所有者、ケースオーナー、ケースワーカーに知らせるために定義する必要があります。
これらの目標の中には、並行して達成できるものもあります。
マイルストーンを完了した後、必要に応じてプロセスに戻って再度マイルストーンをトリガーすることも可能です。
また、マイルストーンについては任意なので、条件が成立しない場合は実行されません。
ケース定義のマイルストーンは、マイルストーンノードで表されます。

We've identified the following milestones in our Credit Card Dispute case:
クレジットカードのチャージバック申請案件では、以下のようなマイルストーンを定義します。

1. チャージバック申請の開始
2. チャージバック承認
3. チャージバック拒否

これらは、チャージバック申請の進捗をトレースするための達成可能な目標です。
これらには特に順番はありませんので、ケースファイルのデータに何か変更があった場合は、いずれかの目標に戻ってくることができます。

次に、ケース定義の中でマイルストーンをモデル化する方法について説明します。

## マイルストーンの定義

ケースのマイルストーンをモデル化するには:

1. `ChargeDispute` プロセスを開きます。

2. オブジェクトライブラリパネルから _スクリプト_ タイプのノードを選択します（パレットの _タスク_ セクションにあります）。それをキャンバス上に配置します。
    ![Business Central Variable Definitions]({% image_path business-central-case-script-task.png %}){:width="800px"}
    
3. 追加したタスクをダブルクリックして、名前を `Log Case Started` に設定します。

4. _スクリプトノード_ のプロパティパネルで、次のように入力します:

    | Name            | Value     |
    | --------------- |:-------------:|
    | スクリプト  | `System.out.println("Case started");` |
    | アドホックの自動開始  | True |

    設定は以下のようになります:
      ![Business Central Designer Script Task Properties]({% image_path ccd-project-log-case-started-node-properties.png %}){:width="450px"}
    
    **Note:** このケースでは _開始イベント_ ノードが定義されていません。スクリプトノードの `アドホックの自動開始` を有効にすると、ケースが開始されたときにノードがアクティブになり、自動的に実行されます。
    
      ![Business Central Designer Script Task]({% image_path ccd-project-log-case-started-node.png %}){:width="450px"}

5. **Log Case Started** タスクをクリックして、そのすぐ横に表示される赤丸をクリックします。これにより、タスクに接続される形で、キャンバスに _終了_ タイプの _終了イベント_ が追加されます。

     ![Business Central Designer Convert End Task]({% image_path business-central-change-end-event.png %}){:width="450px"}

6. この終了タスクを終了シグナルタスクに変換します。上の画像のように、終了タスクをクリックして、左下隅の歯車のアイコンをクリックします。`変更・終了シグナル` オプションを選択します。終了イベントノードは以下のようになります:

     ![Business Central Designer Script Task End Event]({% image_path ccd-project-end-signal-dispute_received.png %}){:width="450px"}

7. シグナルを _Dispute Received_ に設定して、ケースが開始されたことのロギングを完了すると、シグナルは `Dispute Received` というマイルストーンをトリガします。シグナルのスコープを _プロセスインスタンス_ に設定します。

    | Name            | Value     |
    | --------------- |:-------------:|
    | シグナル  | `Dispute Received` |
    | シグナルのスコープ  | プロセスインスタンス |

    ![Business Central Designer Script Task End Event]({% image_path ccd-project-signal-properties.png %}){:width="400px"}

8. キャンバスにマイルストーンノードを追加します。
  
    ![Business Central Milestone Node]({% image_path business-central-milestone-in-palette.png %}){:width="300px"}

9.  それをダブルクリックして、その名前を `Dispute Received` に設定します。シグナル終了イベントがこのマイルストーンをトリガするためには、イベントの _シグナル_ プロパティはマイルストーンの _名前_ と全く同じ名前でなければなりません（ここでは両方とも _Dispute Received_ です）。

10. 次に、ノードを選択し、プロパティパネル（キャンバスの右側）を開き、`データ割り当て` オプションを展開します。

11. `割当` フィールドをクリックします。これで _データ入力／出力と割り当て_ エディタが開きます。データ入力の `Condition` が既にリストアップされているはずです。_ソース_ フィールドで `Expression` を選択し、以下の条件式の値を入力(または貼り付け)します。

    | Name           | Data Type     | Value     |
    | --------------- |:-------------:| |:-------------:|
    | Condition | Boolean |  CaseData(data.get(\"fraudData\") != null) |

    **Note:** マイルストーンの `Condition` を設定しています。これは、`fraudData` のケースファイル項目が `null` ではないときに、マイルストーンノードをトリガーします。マイルストーンは、その条件が満たされたときに **完了** します。 

    ![Business Central Designer Milestone Dispute Assignments]({% image_path milestone-input-condition.png %}){:width="600px"}

    キャンバスは以下のようになっているはずです:

    ![Business Central Designer Milestone Dispute Received]({% image_path ccd-project-milestone-dispute_received.png%}){:width="350px"}

12. プロセス/ケースの定義を保存してください。

先ほどスクリプトタスクで見たように、開始イベントのないアドホックノードは `アドホックの自動開始` プロパティで設定することができます。
これにより、ケースが開始されたときに自動的にノードがアクティブになります。
アドホックノードをトリガーしたりアクティブにしたりするもう一つの方法は、ノードにシグナルを送ることです。
この場合、_終了シグナルイベント_ は _マイルストーン_ ノードをトリガーし、それをアクティブにします。
マイルストーンは、条件式（Expression）とデータの状態に基づいて完了します。
言い換えれば、今作成したマイルストーンはデータ駆動型であるということです。

ここまでで、初期プロセスと、`マイルストーン` のトリガーとなる `終了シグナルイベント` と、`ケースデータ` に基づいて発生する `マイルストーン` の条件式を設定しました。

次のセクションでは、 API を通じてケースを実行し、Business Central を通じてケースの進捗状況を追跡します。