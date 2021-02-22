# 5. ケースプロジェクトの作成

このセクションでは、以下の内容について説明します:

1. ケースを作成するための業務の前提条件
2. このケースの役割と変数

## 概要
プロセスを自動化する場合の最初のステップは、プロセスの様々なステップと構造を明示的に定義することです。

クレジットカード所持者がチャージバックの申請を行うと、どうなるのか？
チャージバック申請の処理は一筋縄ではいきません。
事前に定義された、構造化された流れの中でプロセスを進行していくことはありません。
プロセスインスタンスの実行中に行われた *人間の意思決定* に応じて、そして最も重要なのは、ケースのデータに応じて、それはチャージバック申請を解決するために、異なるステップと異なる _登場人物_ の間を行ったり来たりします。

_NOTE: 本モジュールのセクション2.ユースケースについて にて、ユースケースの概要と登場人物の詳細について紹介しているので、適宜確認してみてください。_

これは動的で、構造化されていない、データ駆動型のプロセスなので、モデル化するための最良の方法は、`ケース` としてモデル化することです。

この特定の `ケース` では、チャージバック申請を解決するというゴールがありますが、そのゴールに至るまでの手順や順序を予測することはできません。
私たちが知っているステップの一つは、_クレジットカード発行者_ がクレジットカード所有者とカード加盟店から情報を収集し、_ケースファイル_ に保存することです。

_NOTE: 実際には、***クレジットカード発行者*** はクレジットカード処理業者（アクワイアラー）と取引を行い、カード加盟店とは直接取引を行いませんが、簡単のために加盟店を考慮に入れています。_

![Business Central CC Dispute Diagram Users]({% image_path business-central-cc-dispute-diagram-users.png %}){:width="400px"}

## オーサリングツール

ケースとプロセスをモデル化するために、Business Central はWebベースのBPMN2デザイナーを提供しています。プロセスとケースのデザイナーについて詳しく見てみましょう:

![Business Central Designer Explained - Stunner]({% image_path business-central-stunner-explained.png %}){:width="800px"}

それぞれの要素について、もう少し理解しておきましょう:

1. _モデリングキャンバス_: これはあなたの描画ボードです。キャンバス上にさまざまなオブジェクトをドロップした後、オブジェクトを移動させたり、連結したりすることができます。キャンバス上のオブジェクトをクリックすると、拡張可能なプロパティウィンドウでそのプロパティを設定することができます(3)。また、オブジェクトの名前を変更したり、接続するオブジェクトを作成したり、オブジェクトを他のタスクタイプに変形させたりすることもできます。

2. _ツールバー_: ツールバーには、デザイナーが提供する膨大な数の機能が含まれています。これらには、キャンバス上に存在するオブジェクトに対して実行できる操作が含まれています。個々の操作は、選択されている内容に応じて無効化または有効化されます。例えば、オブジェクトが選択されていない場合、切り取り/貼り付け/削除の操作は無効になっており、オブジェクトを選択すると有効になります。ツールバーのアイコンにカーソルを合わせると、操作の説明テキストが表示されます。

3. _プロパティパネル_: デザイナーの右側にあるこの拡張可能なセクションでは、プロセスとオブジェクトの両方のプロパティを設定することができます。キャンバス内のオブジェクトをクリックすると、このパネルがリロードされ、そのオブジェクトタイプに固有のプロパティが表示されます。キャンバス自体（オブジェクト上ではなく）をクリックすると、パネルには一般的なプロセスプロパティが表示されます。

4. _オブジェクトライブラリパネル_: デザイナーの左側にある拡張可能なセクションは、BPMN2（デフォルト）要素ツリーを示しています。これには、プロセスやケースを組み立てるのに使用できるRHPAM BPMN2のテンプレートが含まれています。各セクションのサブグループを展開すると、オブジェクトをドラッグ＆ドロップすることでキャンバス(1)に配置することができます。

5. _ビュー タブ_: 現在、デザイナーは3つのタブを提供しています。モデル、概要、ドキュメントです。モデルはデフォルトのタブです。概要は、アセットの履歴、コメント、その他のメタデータを表示します。ドキュメント タブには、プロセス定義から生成されたドキュメントが表示されます。

6. _アラート タブ_: デザイナー の下部には、Business Central がプロジェクトのビルド中に見つかったプロセスやコードの検証に関する警告やエラーに関するアラートを表示するセクションがあります。


## ケースの変数と役割

We have defined the _Business Object Model_ and the _Business Decisions_ in the previous lab. If you've completed the labs of Module 2,  you can use your existing project.

NOTE: _If you'd prefer to start off fresh you can delete your `ccd-project` project and re-import it using this URL: [https://github.com/RedHat-Middleware-Workshops/rhpam-rhdm-workshop-v1m3-labs-step-1.git](https://github.com/RedHat-Middleware-Workshops/rhpam-rhdm-workshop-v1m3-labs-step-1.git)_

### Creating your first Case Definition

To create your first Case Definition:

1. Go to your library view and click on _Add Asset_. From the asset catalog select `Case Definition`, configure the following values:

  * Name: `ChargeDispute`
  * Package: `com.myspace.ccd_project`

  ![Business Central Asset New Case ]({% image_path ccd-add-case-definition.png %}){:width="600px"}

  ![Business Central New Case Details ]({% image_path ccd-project-add-case-definition-charge-dispute.png %}){:width="600px"}


#### Defining Case Variables

We will first define our case variable. These variables will be used to store the case data during the execution the case.

1. In the properties panel on the right-hand-side of the screen, scroll down to the bottom and expand the `Case Management` section. In the `Case File Variables` table, add the following variables:

	| Name            | Data Type     |
	| --------------- |:-------------:|
	| customerStatus  | String |
	| totalFraudAmount| Float  |
	| fraudData | FraudData      |
	| approvedChargeback | Boolean |
	| creditCardholder | creditCardholder |

    At the end your variable definitions should look like this:

    ![Business Central Variable Definitions]({% image_path case-file-variables.png %}){:width="400px"}

2. Save your asset by clicking on the `save` button.

#### Defining Roles

In the Credit Card Dispute case, we can identify different roles. The mapping of the users and/or groups to these case roles is done when the case instance is started, and can be changed afterwards.

1. On the properties panel look for the `Case Roles` table (just above the `Case File Variables` definition). Add the following roles:

  | Name             | Cardinality |
  | ---------------- |:-----------:|
  | owner            | 1           |
  | approval-manager | 1           |

The cardinality refers to the number of actors that can be mapped to a role.

![Business Central Case Roles]({% image_path case-roles.png %}){:width="400px"}

Save your asset by clicking on the `save` button.

----- 

We have defined the basic information to get started with our case. Now, let's understand how we can drive our case based on the existing data and define some milestones.
