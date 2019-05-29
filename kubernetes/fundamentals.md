# Fundamentals

## MasterとNode
KubernetesはMasterとNodeの二種類に大別される。  

### Masterの役割
- API Endpointの提供
- コンテナのスケジューリング
- コンテナのスケーリング

### Nodeの役割
- Dockerのホストとしてコンテナが起動するインスタンス

## Manifest file
- マニフェストファイルの形式は `YAML` または `JSON`
- kubectlがマニフェストファイルの情報を元にしてMasterにAPIリクエストを送ってKubernetesを操作する

## Kubernetesリソース
- 大きく分けて5種類ある
  - Workloads
  - Discovery & LB
  - Config & Storage
  - Cluster
  - Metadata

### Morkloads
Cluster上にコンテナを起動させるためのリソース。
- Pod
- ReplicationController
- ReplicaSet
- Deployment
- DaemonSet
- StatefulSet
- Job

### Discovery & LB
コンテナのサービスディスカバリやクラスタ外部からアクセス可能なエンドポイントを提供するためのリソース。
- Service
  - ClusterIP
  - ExternalIP
  - NodePort
  - LoadBalancer
  - Headless
  - ExternalName
  - None-Selector
- Ingress

### Config & Storage
設定、秘匿データをコンテナに埋め込む、または永続ボリュームを提供するためのリソース。
- Secret
- ConfigMap
- PersistentVolumeClaim

### Cluster
Cluster自体の振る舞いを定義するためのリソース。
- Node
- Namespace
- PersistentVolume
- ResourceQuota
- ServiceAccount
- Role
- ClusterRole
- RoleBinding
- ClusterRoleBinding
- NetworkPolicy

### Metadata
Cluster内にある他のリソースを制御するためのリソース。
- LimitRange
- HorizontalPodAutoscaler
- PodDisruptionBudget
- CustomResourceDefinition

### Namespace
Namespaceは仮想的なKubernetes Clusterを分離する機能。  
初期状態では下記のNamespaceが存在する。
- kube-system
  - Clusterのコンポーネントやアドオンがデプロイされる
- kube-public
  - 全てのユーザが利用できるConfigMapなどを配置する
- default
  - ユーザが任意のリソースを登録する際に使われる

用途としてはkubernetes ClusterをIstioのようなサービスメッシュで共用利用するようなケースに限られる。  
また、RBAC(Role Based Access Control)でCluster操作に関する権限をNamespace毎に制御することもできる。  
そしてNetworkPolicyと組み合わせることでNamespace間の通信を制御できる。

### CLIツール
代表的なCLIツールとして「kubectl」があり、これはMasterにAPIリクエストを手動で送る場合に使用される。  
このkubectlがMasterと通信するために接続先や認証情報などが必須となるが、これはkubeconfigの情報を参照して接続を行う。  

#### kubeconfig
kubeconfigもマニフェストファイルと同じフォーマットとなっており、下記のような認証要素に分かれている。
- クラスタ認証
- ユーザ認証
- コンテキスト
クラスタ認証には接続先の情報、ユーザ認証には認証情報を定義し、コンテキストはクラスタとユーザのペアおよびNamespaceの指定を定義している。  

コンテキストやNamespaceの切り替えはkubectlコマンドで行うこともできるが、 `kubectx` `kubens` などのOSSを導入するとコマンド入力を簡略化できる。

#### stern
`stern` というOSSを導入するとスケールされたPodのログを自動で色分けして一画面に時系列で集約表示してくれる。

### Metadata
Metadataには `annotations` `labels` がある。  

#### Annotation
AnnotationはGKEやEKSなど特定環境に応じたロードバランサー連携などの定義を指定できる。

#### Label
LabelはPod単位でラベルを付与することでグルーピングしやすくしたり、何らかの処理をする際の条件分岐に役立てることができる。

### topコマンド
`kubectl top node` といったコマンドを使用することでリソース使用量を確認することができる。

### ポートフォワーディング
ローカルから特定のPod(MySQLなど)に接続したいときは `kubectl port-forward` コマンドを用いるのが手っ取り早い。
