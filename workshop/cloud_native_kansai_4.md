# Cloud Native Kansai #04

## エンタープライズKubernetesを再定義する - Red Hat OpenShift 4

Kubernetes実践ガイド書いてる人だった。

### OpenShiftとは

- 4にバージョンアップした
- RedHatはIBMに吸収された
- CoreOSと組み合わせて運用自動化のソリューションをしていた
  - OpenShiftと合体させて今に至る
- DockerからCRI-Oに切り替わった
- CoreOS -> Kubernetes -> App Layerの構造

### Operatorとは

- 運用自動化
- Kubernetes上にOperatorというコンテナを立ち上げる
  - アプリケーションの死活監視はOperatorが行う
  - プロセス観察および自動復旧
- Operatorはこれらの機能を利用して実装されている
  - CRD(etcd)
  - API Aggregation
- OperatorHub.io
  - コミュニティやRedHatが提供した様々なOperatorがあり、簡単に使える
  - 登録されるには自動テストなどの一定の認定がある

### まとめ
- 運用自動化はOperatorありきになってくる・・・らしい
- `learn.openshift.com` で気軽に触れる

## Packer/Ansible/Terraformを使ったAWS Fargateへのデプロイ

- 利用するサービス
  - AWS
      - ECR
      - Fargate
      - Code Build
      - Code Deploy
      - Code Pipeline
- 構成
  - githubにpushするとCode Buildでイメージ作成し、ECRに保存。Fargateへデプロイする。  

## MLOpsで必要なflowを考えてみる

### MLOpsとは？

- ML will run on Kubernetesというワードがある
- Opsを機械学習するためのPipeline
- データ収集、開発、予測を繰り返して精度を上げるモデル
- MLOpsで発生する運用負担をKubernetesで解消できるのでは？

### Kubeflow

- Kubernetesを使ってMLを使えるフレームワーク
- Kubeflowの構成
  - Ambassador
    - Kubernetes上で動作するAPI Gateway
    - Modelの提供をする
    - Envoyの構成
    - 認証認可・暗号化
  - CentralUI
  - JupyterHub
  - TFJobDashBoard
  - KubeDash
  - Soldon Core
    - 機械学習のモデルをKubernetesクラスタにデプロイする
      - TensorFlowやSparkなど
  - Argo
    - PipelineのWorkflow
    - YAMLで書ける

### まとめ
  - 機械学習のモデル作成や提供はインフラが担当することが多い
  - MLOpsのパイプラインを意識して設計しよう
  - 省力化運用はできる
  - ABEJA Platform  

## OSSによるCI/CD環境構築

### CI/CDとは

- 定義
  - 開発者はソースコード変更に専念し、自動でビルド・テスト・コード解析を行う

### Rancherとは

- Kubernetesのクラスタ構築や管理ができるプラットフォーム
- Rancher Catalog
  - カタログ画面からデプロイしたいアプリケーションおよびパラメータを入力すれば簡単にデプロイできる

### RancherによるCI/CD環境構築

- Tools
  - Jenkins
    - CI/CD Pipeline管理
  - SonarQube
    - ソースの静的解析
  - Nexus
    - コンテナイメージ管理
  - GitLab
    - ソースコード管理
    - Helm Chart管理

### 環境構築
- カタログ画面からJenkinsをデプロイする
  - Ingress, Service, LDAPなどの設定
  - デフォルトでJenkins-kubernetesプラグインが設定されている
- 同様にSonarQube、Nexus、GitLabをデプロイする
- Rancher Pipeline
  - パイプラインの編集はGUIから行える
  - YAMLでコード管理もできる
  - Slackでパイプラインの実行通知を確認できる

### まとめ
- 分岐条件はまだ未対応だがシンプルなパイプラインなら簡単に構築できる
- ツール間の連携などでは色々問題があったりもするらしい

## Cloud Runってなんやねん

- FaaSとコンテナをサービス特性にあわせて使い分け
- トラフィックや利用時間にムラがあるとどちらかに振り切るのは難しい

### Knative

- Kubernetes上でFaaS的な環境をつくれる
- リクエストが来たらComputeが起動し、一定時間動かないと停止する
- FaaSとの違いは制約が少ないこと、インフラ管理不要なこと
- 開発スコープで制約に問題なければFaaS、厳しくなるならKnative

### Cloud Run

- Full Managed Knative
- 課金体系もFaaSと同じ
- 80並列まで処理できる

### CloudEvents

- CNCFがサーバレスのEvent処理のインターフェースを定義し始めている
