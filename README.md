Red Hat Process Automation Manager / Decision Manager Workshop Module 3
===
Red Hat Process Automation Manager と Red Hat Decision Manager の ハンズオンワークショップの `モジュール 3` です。
このワークショップでは、開発者や業務エキスパートに、最新のクラウドネイティブアーキテクチャのコンテキストでのルールおよびプロセス駆動型アプリケーションとマイクロサービスの紹介を行います。

Agenda
===
* イントロダクション
* ユースケースについて
* ケース管理の概要
* ユーザーとロールの設定
* ケースプロジェクトの作成
* ケースの作成 - マイルストーンとシグナル
* ケース管理アプリケーションの実行
* ケースとの統合に向けたデシジョンの準備
* ケースにデシジョンとゲートウェイを統合する
* ケースにヒューマンタスクを統合する
* おわりに

Run locally
===

```
$ git clone https://github.com/kamorisan/rhpam-rhdm-workshop-v1m3-guides-jp.git
$ cd rhpam-rhdm-workshop-v1m3-guides-jp
$ docker run -it --rm -p 8081:8080 -v $(pwd):/app-data -e CONTENT_URL_PREFIX="file:///app-data" -e WORKSHOPS_URLS="file:///app-data/_rhpam-rhdm-workshop-module3.yml" -e LOG_TO_STDOUT=true quay.io/osevg/workshopper
```

Lab Instructions on OpenShift
===

APB経由で labs-infra をインストールした場合、ラボインストラクションはすでにデプロイされていることに注意してください。

ここでは、ラボインストラクションを OpenShift クラスタに手動でデプロイするための Ansible プレイブックの例を示します。
```
- name: Create Guides Module 3
  hosts: localhost
  tasks:
  - import_role:
      name: siamaksade.openshift_workshopper
    vars:
      project_name: "guide-m3"
      workshopper_name: "RHPAM / RHDM Workshop V1 Module-3"
      project_suffix: "-XX"
      workshopper_content_url_prefix: https://raw.githubusercontent.com/kamorisan/rhpam-rhdm-workshop-v1m3-guides-jp/main
      workshopper_workshop_urls: https://raw.githubusercontent.com/kamorisan/rhpam-rhdm-workshop-v1m3-guides-jp/main/_rhpam-rhdm-workshop-module3.yml
      workshopper_env_vars:
        PROJECT_SUFFIX: "-XX"
        COOLSTORE_PROJECT: coolstore{{ project_suffix }}
        OPENSHIFT_CONSOLE_URL: "https://master.seoul-2922.openshiftworkshop.com"
        ECLIPSE_CHE_URL: "http://che-labs-infra.apps.seoul-2922.openshiftworkshop.com"
        GIT_URL: "http://gogs-labs-infra.apps.seoul-2922.openshiftworkshop.com"
        NEXUS_URL: "http://nexus-labs-infra.apps.seoul-2922.openshiftworkshop.com"
        LABS_DOWNLOAD_URL: "http://gogs-labs-infra.apps.seoul-2922.openshiftworkshop.com"

      openshift_cli: "/Users/doh/cloud-native-app-dev/oc --server https://master.seoul-2922.openshiftworkshop.com"
```
