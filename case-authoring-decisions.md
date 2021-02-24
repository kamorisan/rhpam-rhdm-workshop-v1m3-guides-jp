# 8. ケースとの統合に向けたデシジョンの準備

このセクションでは、ケースやプロセスと統合できるように、さまざまなタイプのデシジョンを設定する方法を学びます。

_このセクションは、 "Module 2 - Red Hat PAM でビジネスルールとデシジョンを設計する" で作成したプロジェクトを使用している方には必須のセクションです。そうでない場合は、ここでの手順はすでにプロジェクトに実装されています。_

## 本プロジェクトにおける自動判定

クレジットカードのチャージバック申請処理のケースを開始すると、自動処理と手動処理の2つの異なるシナリオを経ることができます。

本プロジェクトでは、"automated-chargeback” という名前のガイド付きルールと、"risk-evaluation” という名前のガイド付きデシジョンテーブルの2つのルールで自動化をしています。

プロセスやケースがこれらのルールを使用できるようにするために、`ruleflow-group` という追加のプロパティを使う必要があります。
このプロパティは多数のルールを1つのグループにまとめることができる属性であり、その後、デシジョンノードを経由してプロセス/ケースから `ruleflow-group` をアクティブにすることで、これらのルールの実行を制御することができます。

ケース内から使用できるようにルールを設定してみましょう。

## ガイド付きルールへの ruleflow-group 設定の追加

1. Business Centralのアセットリストで、`automated-chargeback` を開きます。
2. `（オプションの表示...）` をクリックします。
  ![Business Central Guided Rule Automated Chargeback]({% image_path automated-chargeback-guided-rule-more-options.png %}){:width="800px"}

3. 次に、`+` アイコンをクリックして新しいオプションを追加します:
  ![Business Central Guided Rule Automated Chargeback New Option]({% image_path automated-chargeback-guided-rule-new-options.png %}){:width="800px"}

4. ポップアップウィンドウで、 `属性`: `ruleflow-group` を選択します。 

5. 新しいフィールド `ruleflow-group` が表示されるはずです。このフィールドに `automated-chargeback` という値を入力してください。
   ![Business Central Guided Rule Automated Chargeback New Option]({% image_path automated-chargeback-guided-rule-configured.png %}){:width="800px"}

6. ルールを保存してください。

## Adding the ruleflow-group to the Guided Decision Table

1. In the Business Central assets list, open the `risk-evaluation` Guided Decision Table
2. Click on the `Columns` tab
  ![Business Central GDST Column tab]({% image_path gdst-column.png %}){:width="600px"}
3. Next, click the `Insert Column` button. Select the checkbox `Include advanced options`
4. New options will show on the menu. Select the `Add an Attribute column` option, and click next
  ![Business Central GDST RuleFlow Group Option]({% image_path gdst-adv-options.png %}){:width="600px"}

5. Select `Ruleflow-Group`, and click `Finish`;
  ![Business Central GDST RuleFlow Group Option]({% image_path gdst-new-option.png %}){:width="600px"}

6. Still on the column tab, open the `Attributes Column` menu. Insert the ruleflow-group name: `risk-evaluation`, and check the `Hide Column` check box.
  ![Business Central GDST Ruleflow Group Config]({% image_path gdst-ruleflow-config.png %}){:width="600px"}

7. Go back to the `Model` tab and confirm your rules are set to the right ruleflow group:
  ![Business Central GDST Ruleflow Group Config]({% image_path gdst-ruleflow-model-config.png %}){:width="600px"}


## Add a new DRL Rule

We will need a rule to define if a chargeback can be automatically made, or if it needs to be manually approved. This rule has a simple logic, but it demonstrates how you can make automatic decision made on case file and manipulate it when needed. 

1. In the Business Central, add a new asset of type `DRL File`
2. Name it `automatic-approval`
3. Use the following code in your rule:

~~~ 
package com.myspace.ccd_project;

import com.myspace.ccd_project.FraudData;
import org.kie.api.runtime.process.CaseData;

rule "automatic-approval"
ruleflow-group "automatic-approval"
no-loop
when
    $caseFile: CaseData()
    $fraudData: FraudData(automated == true) from $caseFile.getData("fraudData")
then
    System.out.println("Automatically approving chargeback.");
    modify($caseFile) {
        add("approvedChargeback", true);
    }
end
~~~

Note our ruleflow-group is set to `automatic-approval`, and that the `then` clause is modifying our FraudData object existing in our caseFile, setting the attribute `approvedChargeback` to `true`.


----- 

With this, we have configured all the rules and into three different ruleflow groups. You can confirm that by switching to the `Source` tab and evaluating the rules code.

Now, let's go ahead and use these rules in our `ChargeDispute` case.


