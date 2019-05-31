# 【大阪・梅田】Kubernetes Meetup Tokyo #19 大阪サテライト

## Slack Channel
slack.k8s.io

## The origins and future of Kubernetes
- 仮想マシンユーザをBorg方向に引き上げるにはどうすればよいか
- Google以外の利用者にBorg的なシステムを使ってもらうアプローチがKubernetes

### What is Kubernetes?
- Kubernetesをコンテナオーケストレーションツールというが、どのように動かすかが主眼である  
- オーケストレータとしての価値は確かにある  
- 一連のロジックはコントローラ  
- Kubernetesの機能は利用者の要件によって変化する  
- スケジューラとコントロールマネージャの構成で実現できるのがKubernetesのオーケストレーションロジック  
- Kubernetesのコントロールプレーンはシングルでもマルチノードでも利用できる  
- ワーカーノードはローカルデーモンと連携して可用性を実現している  

### コントローラとは？
- 利用的な状態のコンフィグ  
- 様々なステートを見るために存在している  
- 分散システムを構成する上では重要なアプローチを行える  
- シンプルな機能が連携することで複雑な動きに対応できている  

### Pod / ReplicaSet 
- Kubernetesのセルフヒーリング機能

### 自分が運用したいやり方を実装するにはどうすればよいか？
- 全てがKubernetesから提供されるのは不可能  
- Custom Resource Definitions

### Operators
- CRDで作られた仕組み  
- Runbookのようなもの  
- AWSのRDSみたいなものを作って運用していける

### Explosion of Operators
- OracleのMySQL Operatorがあり、DBベンダーがDBのAdministrationをKubernetesに乗せることで簡単にするアプローチをしていることは興味深い

### Kube flow
- マシンラーニングに関わるOperationの生産性を上げるためのKubernetes活用例

### Gimbal
- マルチクラスタ対応のHTTPロードバランサ  
- マルチテナントでの運用環境を備えている

### The Future
- 分散システムはKubernetesを前提にしていくことで活用していける  
- より多くのシステムをセルフマネージドで運用できる

### Stateful Setの考え方
- 使い所としてはDBなど  
  - DBへの影響を考えながら作っていかないといけない

### スケーラビリティ
- ノードで5000台などの場合、Podの数などを考えて設計しなければならないが、シャーディングは必要になってくると考える  
- 溜め込んでいくデータストアを分割してよりスケーラビリティにシステムを動かせるようにしていくべき  
- スケーリングをマルチクラスタでやる場合は可用性が大きく向上する

### etcdを選んだ理由
- Borgのデータベースはロジックが一体となっている  
  - しかしそれはKubernetesでは避けたかった  
- ZooKeeperをベースに開発することを考えた時にetcdを使うことにした  
- 改めて考え直すとしても引き続きetcdを使うことになると思う  
- 将来的には他のインターフェースに変えることでチョイスを増やすようにしたい  

### K3s
- ある一定の利用用途においてKubernetesの良さを活用していける  
- APIドリブンのアーキテクチャが実現できていると思っている  
- IoTを考えた場合、Kubernetesがどこにデプロイされるという考え方について  
  - デバイスにkubeletを入れてしまうのもありだとおもうが、それがフィットするのかという問題については答えが出ていない

## Kubernetes at Yahoo Japan Iaas team

### Gimbal
- L7LB Platform  
- VMからContainerへの移行を支援している  
- L7にすることで可視化できる範囲が広がる  
- Envoyを使っている  
- Routing rukesはContourを使っている

### IaaSを裏側で支えるKubernetes

#### 事例1
- Openstack on Kubernetes  
- ワークロードをVMからContainerへ変更した  
- HVとコントローラで成り立っている  
- JenkinsからOpenStackを叩こうとするとパイプラインが複雑化する  
  - 手動操作にJenkinsは気づけない
- VMからコンテナに移行することで管理サーバ数が激減した  
- CentOS＋Dockerが不安定で苦労した

#### 事例2
- L4LBのコントロール/データプレーンにKubernetesを使用している  
- 性能が出づらくTCPのトレースが難しい  
- トラブル時に全ノードでtcpdumpを取らなければならない  

## kube-awsからEKSに移行した話
- EKSに移行することでMaster Componentの管理から開放された
  - マルチテナントからシングルテナントへ変更した
  - セキュリティ境界を明確化できた
  - インフラの権限委譲を促進できた
- 単一クラスタに複数プロダクトから複数クラスタに複数プロダクトへ変えられた
- セキュリティグループで分けられる
- SREだけでは回らないのでクラスタ管理は開発へ権限委譲
  - 人的なオペレーションの不安はGitopsを使ってレビューを挟んで解消した
  - シングルテナントなら運用負荷を分散できる

## 図で理解するDescheduler
- Deschedulerとは？
  - PodをNodeへ割り当てる役割
  - Nodeリソースが偏ったりする
    - Podを移動可能にして再スケジュールする
    - 削除するPodのポリシーをファイルで定義できる
  - RemoveDeplicates
    - Podが各Nodeに最低1つあることを確認する
  - LowNodeUtilization
    - リソース使用率が引くいNodeへPodを移動する
    - thresholdsで設定した閾値を上回るとNodeのリソース使用率が高いとみなされる
  - RemovePodsVilatingInterPodAntiAffinity
    - Anti-Affinityに違反しているPodを削除する
  - RemovePodsVilatingNodeAffinity
    - NodeAffinityに違反しているPodを削除する
