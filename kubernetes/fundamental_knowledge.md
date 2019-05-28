# Fundamental Knowledge

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