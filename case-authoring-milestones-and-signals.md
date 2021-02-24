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

## Defining Milestones

To model the milestones of the case:

1. Open the `ChargeDispute` process.

2. Select from the Object Library Panel a node of type _Script Node_ (located in the _Tasks_ section of the palette). Place it on the canvas.
    ![Business Central Variable Definitions]({% image_path business-central-case-script-task.png %}){:width="800px"}
    
3. Double click the task you just added, and set the name as: `Log Case Started`. 

4. On the properties panel for the _Script Node_ enter the following:

    | Name            | Value     |
    | --------------- |:-------------:|
    | Script  | `System.out.println("Case started");` |
    | AdHoc Autostart  | True |

    Your configuration should look like this:
      ![Business Central Designer Script Task Properties]({% image_path ccd-project-log-case-started-node-properties.png %}){:width="450px"}
    
    **Note:** that we have not defined a _Start_ node in our case. By enabling the `Adhoc autostart` on the script node, the node will be activated and executed automatically when the case is started.
    
      ![Business Central Designer Script Task]({% image_path ccd-project-log-case-started-node.png %}){:width="450px"}

5. Click on the **Log Case Started** task, and click on the red circle that appeared right next to it. This will add an _End Event_ of type _End Signal_ to the canva, already connected to your task. 

     ![Business Central Designer Convert End Task]({% image_path business-central-change-end-event.png %}){:width="450px"}

6. Convert this End Task to and End Event Task. As represented on the image above, click on the End Task, and click on the engine icon on the left corner. Select the "Convert into End Signal" option. Your end node should look like this:

     ![Business Central Designer Script Task End Event]({% image_path ccd-project-end-signal-dispute_received.png %}){:width="450px"}

7. Set its Signal to _Dispute Received_, so once you've completed the logging that the case has started, the signal will trigger a Milestone called `Dispute Received`. Set its Signal Scope to _Process Instance_.

    | Name            | Value     |
    | --------------- |:-------------:|
    | Signal  | `Dispute Received` |
    | Signal Scope  | Process Instance |

    ![Business Central Designer Script Task End Event]({% image_path ccd-project-signal-properties.png %}){:width="800px"}

5. Add a Milestone node to the canva.
  
    ![Business Central Milestone Node]({% image_path business-central-milestone-in-palette.png %}){:width="300px"}

6.  Double click it, and set its name to `Dispute Received`. In order to allow the _Signal End Event_ to trigger this _Milestone_, the _Signal_ property of the event should have the exact same name as the _Name_ of the _Milestone_ (which we've done - both are _Dispute Received_). 

9. Now, select the node, open the properties panel (on the right side of the canva), and unfold the "Data Assignments" options. 

10. Click on the `Assignments` field. This will open the _Data Input/Output Assignments_ editor. The data-input `Condition` should already be listed. In the _Source_ field, select `Expression`, and type (or paste) the condition expression value below. 

    | Name           | Data Type     | Value     |
    | --------------- |:-------------:| |:-------------:|
    | Condition | Boolean |  CaseData(data.get(\"fraudData\") != null) |

    **Note:** We are setting the `Condition` of the Milestone. This condition will trigger the Milestone node when the `fraudData` case file item is not `null`. Note that triggering the Milestone only *activates* it, in other words, it doesn't complete the milestone. A Milestone is *completed* when its Condition is met.  

    ![Business Central Designer Milestone Dispute Assignments]({% image_path milestone-input-condition.png %}){:width="600px"}

    You should have something like this:

    ![Business Central Designer Milestone Dispute Received]({% image_path ccd-project-milestone-dispute_received.png%}){:width="350px"}

6. Save your process/case definition.

As we saw earlier with the Script Task the ad hoc nodes with no incoming connections can be configured with the `Adhoc autostart` property, which is a property of the node itself. This will activate the node automatically when the case is started. Another way of triggering/activating an _ad hoc_ node is by signalling it. In this case our _Signal End Event_ triggers our _Milestone_ node and activate it. The milestone will be completed basedonconditional expressions and the state of the data. In other words, the milestone we've just created is data-driven.

So far we've configured the initial process, a `signal end-event` that triggers the `milestone`, and a `milestone` conditional expression which fired based on the `case data`.

In the next section we will execute our case, interact with the engine API and track its progress via Business Central.