# 4. ユーザーとロールの設定

Red Hat Process Automation Manager のセキュリティシステムは、ユーザー、ロール、グループに基づいています。
これらの設定は、PAM を実行しているアプリケーションサーバー（この場合は Red Hat Enterprise Application Server (Red Hat EAP)）からデフォルトで提供されます。 
エンタープライズアーキテクチャで作業する場合、RHPAM をエンタープライズ LDAP システムや Red Hat Single Sign On (別名 Keycloak) のような外部プロバイダに接続することをお勧めします。

このセクションでは、Red Hat Process Automation Manager のすぐに使えるセキュリティ管理システムを使用して、ユースケースを実装するために必要な新しいユーザーとグループを作成する方法を学びます。

RHPAMでは、2種類のユーザーが存在します:

- **platform users** : これらのユーザーは、Business Central、つまりオーサリング環境と対話するユーザーです。例えば、Business Centralへのログインに使用した `pamAdmin` などです。

- **application users** : これらはプロセス駆動型のアプリケーションを構築し、プラットフォーム上で実行する際に必要なユーザーのことです。_チャージバック申請_ 処理システムの場合、プロセスと対話する2つの異なるロールが必要になります:
	- **クレジットカード所持者*
	- Pecunia Corp. で働く、 **承認権限のあるマネージャー** 

実際には、これらは個々のユーザーではなく、異なるユーザーが所属できるグループとなります。
私たちの銀行では、この2つのグループが定義されており、プロセスを自動化しながら、各グループのタスクを定義し、設定していきます。

## Business Central でのユーザー設定

Business Centralでは、ユーザーが実行できる（または実行しなければならない）タスクに応じて機能をグループ化した、アクティビティ中心のデザインアプローチを採用しています。

1. ユーザーやグループを管理する機能にアクセスするには、画面右上の `設定` メニューの `歯車` アイコン ![Gear Icon]({% image_path gear-icon.png %}){:width="600px"} を選択してください。


2. The _Settings_ menu shows all components that you can configure in your environment. In this step we will focus on the users and groups.

  ![Settings Menu]({% image_path settings-menu.png %}){:width="600px"}

3. Click the `Groups` option and you'll see the group administration menu, where you can list the groups that currently exist, update or delete them or create a new group.

  ![Groups Menu]({% image_path groups-menu.png %}){:width="600px"}

4. Click on `New Group`. The name group is `card-holder`{{copy}}.

5. After you type in the name of the group, in this case `card-holder`{{copy}}, select the `pamAdmin` and click on `Add selected users` to add it to the `card-holder` group.

  _In a real world scenario all these groups will be associated with different personas but for the sake of simplicity, your user will be part of all the groups needed, in order for you to be able to test the processes and application._

5. Now, repeat the steps and create the group `approval-manager`. 

At the end you should have two aditional groups:

- `card-holder`{{copy}}
- `approval-manager`{{copy}}

![User Groups Settings Menu]({% image_path user-groups-settings-menu.png %}){:width="600px"}  

**You have successfully completed the groups configuration of your environment.**
