# Workloads

## Pod
- Podは1つ以上のコンテナで構成されている
- 単一Pod内に含まれる複数のコンテナはネットワーク的に隔離されていない
- コンテナのIPアドレスはPodのIPアドレスを共有している
- なのでPort被りで動作しなくなる

### デザインパターン
- Podのデザインパターンには大きく三種類ある
  - サイドカー
    - メインとなるコンテナに機能追加する役割
  - アンバサダー
    - 外部システムとのやり取りを代理する
  - アダプタ
    - 外部からのInboundアクセスに対するインターフェースを担う

#### サイドカー
- 役割の例
  - 特定の変更を検知したら動的にどこかの設定を変更する
  - Gitリポジトリとローカルストレージを同期する
  - アプリログをオブジェクトストレージに転送する

#### アンバサダー
- 役割の例
  - CloudSQL Proxy
  - Redis Proxy
    - Localhostで接続することでバランシングの役割を持つコンテナが本来の接続先へ効率よくブリッジしてくれる

#### アダプタ
- 役割の例
  - Prometheus
    - MySQL Exporter
    - Redis Exporter
      - Prometheusが求めるフォ＝マットでメトリクスを整形して渡してくれる

## Deployment
- apply時に `--record` を付けておくとアップデート時の履歴を保存できる
  - 変更履歴の確認
    - `kubectl rollout history deployment DEPLOYMENT_NAME`
  - ロールバックの実行
    - `kubectl rollout undo deployment DEPLOYMENT_NAME [--to-revision (0|1|N)]`
- 更新の一時停止
  - `kubectl rollout pause deployment DEPLOYMENT_NAME`
- 更新の再開
  - `kubectl rollout resume deployment DEPLOYMENT_NAME`

### アップデート戦略
`Recreate` と `RollongUpdate` がある

#### Recreate
- 一度、全てのPodを削除してから新しいPodを作成するためダウンタイムが発生する
- そのため余剰リソースを使わず、切り替えは早まる

#### RollingUpdate
- アップデート時の挙動を制御できる
  - maxUnavailable
    - アップデート中に許容される減らしても構わないPod数を指定する
  - maxSurge
    - 超過が許されるPod数を指定する
- 値の指定方法は `数` または `パーセンテージ` で指定できる
  - デフォルト値は `25%`
- その他設定できるパラメータ
  - minReadySeconds
    - PodがReadyになってからDeploymentリソースがPodの起動を完了したと判断するまでの最低秒数
  - revisionHistoryLimit
    - ReplicaSetの数
    - ロールバック可能な履歴数
  - progressDeadlineSeconds
    - Recreate / RollongUopdate処理を行う際のタイムアウト時間
      - タイムアウト時間を超過した歳に自動でロールバックされる

## DaemonSet

### ReplicaSetとの違い
- ReplicaSetは使用可能なNode上に指定された数のPodをリソース状況に合わせて配置する
  - つまり各NpdeにおけるPodの数が均一化されるわけではない
  - また、各Nodeに必ずPodが配置されるわけでもない
- DaemonSet
  - DaemonSetは各NodeにPodを1つずつ配置するリソースである
  - なので、レプリカ数を指定することはできない
  - また、1NodeにPodを2つ配置することもできない
  - ただし、配置したくないNodeがある場合は `nodeSelector` `AntiAffinity` などで除外できる

### ユースケース
- Fluentd
  - Podが出力するログをホスト単位で収集する
- Datdog
  - Podのリソース状況やノードのモニタリング
- これらのように全ノードで必ず動作させておきたいプロセスのために使用する

### アップデート戦略
`RollongUpdate` と `OnDelete` がある

#### OnDelete
- マニフェストファイルを変更してイメージの差し替えをしても既存のPodをアップデートしない
- 何らかの要因でPodが停止して再作成されない限りはアップデートしない
- 任意のタイミングでアップデートを行う必要があるため、セルフヒーリングを促すために `kubectl delete pod` を実行する必要がある

## StatefulSet

### ReplicaSetとの違い
- 作成されるPodのSuffixに数字のインデックスが付与される
- PersistentVolumeを使ってデータ永続化の仕組みを持っている
- Pod名は変わらない

### スケーリング
- スケールさせることは可能
- 追加されたPodのSuffixがインデックスされていく
- 削除時はインデックスの値が大きいものから削除されていく
- Pod作成時にReadyのステータスになってから次のPodを作成する
  - ただし、`spec.podManagementPolicy` を `Parallel` にすればReplicaSetのように同時並列でPodを作成することもできる

### アップデート戦略
`RollongUpdate` と `OnDelete` がある

## Job
コンテナを利用して一度限りの処理を実行するためのリソース。

### ユースケース
- Podが停止することを前提にしている
- 長時間の実行が予想される「特定サーバとのrsync」や「オブジェクトストレージへのアップロード」など

### 挙動
- `kubectl get jobs` などで確認するとJobが正常に終了した場合、SUCCESSFULがカウントされていく
- restartPolicy
  - Never
    - Podの障害時に新規Podを作成する
  - OnFailure
    - 同じPodを利用してJobを再開する

### 並列化
- オプション
  - spec.completions
    - 成功回数の指定
  - spec.parallelism
    - 並列度の指定
  - spec.backoffLimit
    - 失敗を許容する回数
- 一度だけ実行したいタスク
  - `completions` `parallelism` `backoffLimit` をそれぞれ1に指定する
- N並列で実行したいタスク
  - `completions: 5` `parallelism: 3` `backoffLimit: 5` のように指定する
- 1つずつ実行するワークキュー
  - `completions: -` `parallelism: 1` `backoffLimit: -` のように指定する
- N並列で実行するワークキュー
  - `completions: -` `parallelism: 3` `backoffLimit: -` のように指定する

## CronJob
Cronのようにスケジュールされた時間にJobを作成する

### 同時実行の制御
- spec.concurrencyPolicy
  - Allow
    - 同時実行を制限しない
  - Forbid
    - 前のJobが終了するまで次のJobを実行しない
  - Replace
    - 前のJobが終了していなかった場合、そのJobをキャンセルして次のJobを実行する

### 実行開始期限の制御
- spec.startingDeadlineSeconds
  - 例えば毎時0分に実行させたいJobの開始許容時間を「0分〜5分」まで可能とする場合は `300sec` を設定すればよい

### 履歴管理
- spec.successfulJobsHistoryLimit
  - 成功したJobを保存する数
- spec.failedJobsHistoryLimit
  - 失敗したJobを保存する数
- これらは `kubectl get jobs` で確認できる