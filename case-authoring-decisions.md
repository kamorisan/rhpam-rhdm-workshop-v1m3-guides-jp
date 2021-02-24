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

## ガイド付きデシジョンテーブルへの ruleflow-group の追加

1. Business Centralのアセットリストで、`risk-evaluation` のガイド付き決定テーブルを開きます。
2. `列` タブをクリックします。
  ![Business Central GDST Column tab]({% image_path gdst-column.png %}){:width="600px"}
3. 次に、`Insert Column` ボタンをクリックします。チェックボックス `高度なオプションを含む` を選択してください。
4. メニューに新しいオプションが表示されます。`属性列を追加` オプションを選択し、`次へ` をクリックする。
  ![Business Central GDST RuleFlow Group Option]({% image_path gdst-adv-options.png %}){:width="600px"}

5. `Ruleflow-Group` を選択し、`完了` をクリックします。
  ![Business Central GDST RuleFlow Group Option]({% image_path gdst-new-option.png %}){:width="600px"}

6. 列タブで `属性列` メニューを開き、ruleflow-group: `risk-evaluation` を入力します。
  ![Business Central GDST Ruleflow Group Config]({% image_path gdst-ruleflow-config.png %}){:width="600px"}

7. `モデル` タブに戻り、ruleflow groupが、`risk-evaluation` に設定されていることを確認します。設定されていない場合は、手動で入力してください。
  ![Business Central GDST Ruleflow Group Config]({% image_path gdst-ruleflow-model-config.png %}){:width="600px"}


## 新しい DRL ルールの追加

チャージバック申請の承認を自動承認とするのか、それとも手動で承認する必要があるのかを定義するルールが必要になります。
このルールは単純なロジックを持っていますが、ケースファイル上で自動で決定を行い、必要なときに操作する方法を示しています。

1. Business Central で、`DRLファイル` タイプの新しいアセットを追加します。
2. DRLファイルの名前は、 `automatic-approval` とし、`OK` を選択します。
3. 以下のコードをルールに使用します：

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

ruleflow-group が `automatic-approval` に設定されており、`then` 句が ケースファイル に存在する FraudData オブジェクトを変更して `approvedChargeback` 属性を `true` に設定していることに注意してください。


----- 

これで、すべてのルールを3つの異なる ruleflow-group に設定しました。これは、`ソース` タブに切り替えてルールコードを評価することで確認できます。

では、これらのルールを `ChargeDispute` のケースで使ってみましょう。


