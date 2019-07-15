# Config & Storage
コンテナに対して設定ファイルやパスワードなどの機密情報をInjectしたり、永続化ボリュームを提供するためのリソース。

## 環境変数の利用
* 下記5つの情報源から環境変数を埋め込める
  - 静的設定
  - Podの情報
  - コンテナの情報
  - Secretリソースの機密情報
  - ConfigMapリソースの設定値

### 静的設定
* `spec.containers[].env` に静的な値として環境変数を設定する
* コンテナのTimezoneを変更する場合は下記設定が一番簡単
```
env:
  - name: TZ
    value: Asia/Tokyo
```

### Podの情報
* どのノードで起動しているか、PodのIPアドレス、起動時間などは「fieldRef」で確認できる
* 具体的なコマンドとしては `kubectl get pods POD_NAME -o yaml`

### コンテナの情報
* コンテナのリソースに関する情報は「resourceFieldRef」から参照できる
* Podには複数のコンテナ情報が含まれているためfieldRefではコンテナ単位での参照ができない

### ConfigMapの設定値
* 単純なKey-Valueの値はConfigMapで管理できる

### 環境変数利用時の注意点
* KubernetesではCommandやArgsで実行するコマンドを指定する際、通常のような環境変数の利用はできない
* 環境変数の呼び出し方としては `${}` ではなく `$()` となる
* もし、OSからしか参照できない環境変数を利用したい場合は `spec.containers[].command` をentrypoint.sh などのシェルスクリプトにしてシェルスクリプト内で処理をすること

## Secret

### Secretの分類
* Generic(type: Opaque)
  - 通常のユーザ名やパスワードといった認証情報の秘匿にはこれを使用する
* TLS(type: kubernetes.io/tls)
  - Ingressなどで証明書などを参照可能にする場合に使用する
* Docker Registry(type: kubernetes.io/dockerconfigjson)
  - Docker Registryの認証情報として使用する
* Service Account(type: kubernetes.io/service-account-token)
  - Podに対してサービスアカウントのトークンや証明書をマウントする際に使用する

### GenericタイプのSecret
* 作成方法は下記の4種類
  - `--from-file`
    - ファイルから値を参照する場合に使用する
    - ファイル名がKeyとなるため拡張子は外しておくこと
  - `--from-env-file`
    - 単一ファイルから一括で作成する場合に使用する
    - Dockerで「--env-file」オプションを使用してコンテナを起動していた場合はこれを使うことでそのままSecretに移行できる
  - `--from-literal`
    - kubectlを使って直接値を渡す場合に使用する
  - `-f`
    - マニフェストから作成する場合に使用する
    - base64encodeした値をマニフェストに埋め込むこと
    - エンコード時に改行が入らないように注意

### TLSタイプのSecret
* 基本的に秘密鍵と証明書のファイルから作成する
* kubectlコマンドで作成する場合
  - `kubectl create secret tls --save-config TLS_NAME --key KEY_PATH --cert CERT_PATH`

### Docker RegistryタイプのSecret
* kubectlコマンドで作成する場合
  - `kubectl create secret --save-config docker-registry AUTH_NAME \
    --docker-server=REGISTRY_SERVER \
    --docker-username=REGISTRY_USER \
    --docker-password=REGISTRY_PASS \
    --docker-email=REGISTRY_USER_EMAIL`
* マニフェストで作成することも出来なくはないが、dockercfg形式のjsonがbase64encodeされているためコマンドでの作成をしたほうが簡単
* DockerHubなどのプライベートリポジトリからイメージを取得する場合
  - Secretを事前に作成する
  - Pod定義にある `spec.imagePullSecrets` にdocker-registryタイプのSecretを作成する必要がある

### 動的なSecretの更新
* Volumeマウントを利用したSecretは一定期間ごとにkube-apiserverに変更確認を行う
  - 変更があればファイルを入れ替える
  - デフォルトの更新間隔は60秒
    - 調整する場合はkubeletの `--sync-frequency` オプションを設定すればよい
  - Podの再作成は発生しないため瞬断も起こらない
* 環境変数を利用したSecretでは起動時に環境変数が決定してしまうため動的な更新は行えない

## kubesec
* KubernetesのSecretを安全に管理するためのOSS
* GnuPG、Google Cloud KMS、AWS KMSを利用してSecretの暗号化を行える
* ファイル全体を暗号化するわけではなく、Secretの構造を保ちつつ値だけ暗号化できるので可読性に優れている

## SecretとConfigMapの使い分け
* 大きな違いは機密情報を扱うか否か
* SecretのデータはKubernetes Masterが利用している分散KVSのetcdに保存される
  - SecretのデータがKubernetes Node上に永続化されないようtmpfsに保持される

## ConfigMap
* 設定情報などをKey-Valueで保存できるリソース
* nginx.confやhttpd.confなどの設定ファイル自体も保存できる
* 一つのConfigMapに格納可能なデータサイズの上限は1MB

### ConfigMapの作成
* 作成方法は下記の3種類
  - `--from-file`
    - ファイルから値を参照する場合に使用する
    - ファイル名がKeyとなるため拡張子は外しておくこと
    - `kubectl create configmap --save-config CONFIGMAP_NAME --from-file=./conf/nginx.conf`
  - `--from-literal`
    - kubectlを使って直接値を渡す場合に使用する
    - `kubectl create configmap --save-config web-config \
      --from-literal=connection.max=100 \
      --from-literal=connection.min=10`
  - `-f`
    - マニフェストから作成する場合に使用する
    - base64encodeは必要ない

### ConfigMapの渡し方
* ConfigMapで定義した特定のKeyを渡す場合は `spec.containers[].env` の `valueFrom.configMapKeyRef` を使う
* 環境変数名に `-` や `_` は変数の参照が難しくなるため使わないほうがいい

### 動的なConfigMapの更新
* Volumeマウントを利用したConfigMapは一定期間ごとにkube-apiserverに変更確認を行う
  - 変更があればファイルを入れ替える
  - デフォルトの更新間隔は60秒
    - 調整する場合はkubeletの `--sync-frequency` オプションを設定すればよい
  - Podの再作成は発生しないため瞬断も起こらない
* 環境変数を利用したConfigMapでは起動時に環境変数が決定してしまうため動的な更新は行えない

## PersistentVolumeClaim
* Volume、PersistentVolume、PersistentVolumeClaimの違い
  - Volume
    - 予め用意された利用可能なボリューム(ホスト領域、NFS、iSCSI)などをマニフェストに指定して利用できるようにする
    - 利用者はKubernetes上で新規ボリュームの作成や既存ボリュームの削除はできない
  - PersistentVolume
    - 外部の永続ボリュームを提供するシステムと連携する
    - 新規ボリューム作成や既存ボリュームの削除ができる
  - PersistentVolumeClaim
    - 作成されたPersistentVolumeリソースの中からアサインをするためのリソース
    - Dynamic Provisioning機能を利用するとPersistentVolumeを動的に作成することもできる

## Volume
* Volumeプラグインは様々な種類がある
  - emptyDir
    - Pod用の一時的なディスク領域として利用できる
    - PodがTerminateされると削除される
  - hostPath
    - Kubernetes Node上の領域をコンテナにマッピングする
    - ホスト上の任意の領域をマウントできる
    - type
      - Directory
      - DirectoryOrCreate
      - File
      - Socket
      - BlockDevice
    - セキュリティの観点からhostPathを使用できないKubernetes環境もある
  - downwardAPI
    - Podの情報などをファイルとして配置する
    - fieldRefやresourceFieldRefと使い方は同じ
  - projected
    - Secret, ConfigMap, downwardAPI, serviceAccountTokenのボリュームマウントを1箇所のディレクトリに集約する
    - Secretの認証情報とConfigMapの設定ファイルを1箇所のディレクトリに配置したい場合に利用できる
  - nfs
  - iscsi
  - cephfs

## PersistentVolume
* 基本的にネットワーク越しにディスクをアタッチするタイプのディスクとなる
* GCPのプラグインとして「GCE Persistent Disk」、AWSのプラグインとして「AWS Elastic Block Store」がある

### PersistentVolumeの作成
* 下記のような項目を設定できる
  - ラベル
    - type、environment、speedなどのラベルを付与するのがおすすめ
    - ラベルが無いとPersistentVolumeから割り当てる際に特定のボリュームを指定するのが難しくなる
  - 容量
    - PersistentVolumeClaimの要求に対して、用意されているPersistentVolumeは最も近い容量のものを割り当てる
  - アクセスモード
    - ReadWriteOnce(RWO)
      - 単一ノードから読み書き可能
    - ReadOnlyMany(ROX)
      - 複数ノードから読み取り可能
    - ReadWriteMany(RWX)
      - 複数ノードから読み書き可能
    - GCP、AWS、OpenStackで提供されるブロックストレージサービスではReadWriteManyはサポートされていない
  - Reclaim Policy
    - PersistentVolumeの利用が終わった後の処理方法を制御するポリシー
      - Delete
        - PersistentVolumeの実体を削除する
        - GCP、AWS、OpenStackなどで確保される外部ボリュームのDynamic Provisioningで利用されることが多い
      - Retain
        - PersistentVolumeの実体を消さずに保持する
        - その際、他のPersistentVolumeClaimから再マウントされることはない
      - Recycle 
        - PersistentVolumeのデータを削除(`rm -rf ./*`)
        - 他のPersistentVolumeから再マウントできる
        - 将来のKubernetesで廃止検討されているため、Dynamic Provisioningを利用したほうがいい
  - マウントオプション
  - StorageClass
  - 各プラグイン特有の設定

## PersistentVolumeClaim
* 永続化領域の要求を行うリソース
* PersistentVolumeClaimを作成する際の設定項目
  - ラベルセレクタ
  - 容量
  - アクセスモード
  - StorageClass

### Dynamic Provisioning
* PVを事前に作成し、PVCの要求を元にディスク領域を割り当てる順序における手間を省いた
* PVCが発行されたタイミングで動的にPVを作成して割り当てられる
* アクセスモードは制限される

### GCEにおけるStorageClass
* ユーザがPVCを使ってDynamic ProvisioningでPVを要求する際にどんなディスクがほしいのかを指定するために利用される
* StorageClassを選択するということは外部ボリュームの種別を選択するのと同義
* GKEの場合は下記4つのパラメータを指定できる
  - type
    - pd-standard
    - pd-ssd
  - replication-type
    - none
    - regional-pd
  - zone
  - zones

## volumeMountsで利用可能なオプション
* ReadOnlyマウント
  - 様々なspec.volumesで定義したvolumesをコンテナにマウントする際にreadonlyオプションを付与できる

## subPath
* Volumeマウントする際に特定のディレクトリをルートとしてマウントする
