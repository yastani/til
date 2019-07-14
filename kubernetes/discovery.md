# Discovery & LB

## ネットワークについて

### IPアドレスの割当
* Podに割り当てられるIP
  - 同一Podとして存在するコンテナにはすべて同じIPアドレスが割り当てられているため指定は `localhost`
  - 異なるPodにあるコンテナへ通信する場合はPodが持つIPアドレスを指定しなければならない

### 内部ネットワーク
* Kubernetesクラスタはクラスタ構築時にノードに対してPodが利用するための内部ネットワークを自動的に構築する
  - これはCNI(Container Network Interface)というPluggableなモジュールの実装によるもの
  - 基本的にはノードごとに異なるネットワークセグメントを構成する
  - また、ノード間トラフィックの通信確立をさせるためにVXLANやL2 Routingを利用して転送させる
  - これらは利用者が意識する必要の無い設定である
  
### Serviceを利用することで得られるメリット
* Podへのトラフィックロードバランシング
* サービスディスカバリおよびクラスタ内DNS

## Podへのトラフィックロードバランシング
* Serviceは受信したトラフィックを複数のPodにロードバランシングする機能を提供する
* ロードバランシングのエンドポイントも提供する
  - これには外部の仮想IPアドレスやクラスタ内でのみ使用できるClusterIPなど様々な種類を提供できる

### ClusterIP
* selectorに対象となるPodのLabelを間違えている場合 `kubectl describe svc` を実行してもEndpointsに宛先のIPが表示されない

## Cluster内DNS / サービスディスカバリ
* サービスディスカバリ
  - 特定条件の対象となるメンバの列挙
    - Serviceに属するPodの列挙
  - 名前からエンドポイントを判別する
    - Service NameからEndpointsを返却する

### 環境変数を利用したサービスディスカバリ
* `docker --links` と同様に環境変数が保存される
  - envをgrepする
    - `kubectl exec -it [Deployment] env | grep -i clusterip`

### DNSのAレコードを利用したサービスディスカバリ
* Kubernetesではアドレス管理から開放される為に自動で払い出されたIPアドレスに紐付いたDNS名を使用するのが基本
* 正式なFQDNは【Service】.【Namespace】.【svc.cluster.local】
  - ServiceがコリジョンしないようにNamepsaceが含まれている
  - ただし、コンテナ内の `/etc/resolve.conf` に汎用的な記述が含まれているため短縮形でも名前解決ができる
  - これは異なるNamespaceだと無効なため、FQDNで名前解決を行う必要がある

### DNSのSRVレコードを利用したサービスディスカバリ
* SRVレコード
  - PortとProtocolを利用してサービスの提供をしているエンドポイントをDNSで解決する仕組み
  - レコードの形式は【_Service Port】.【_Protocol】.【Service】.【Namespace】.【svc.cluster.local】

### クラスタ内DNSとクラスタ外DNS
* dnsPolicyでPodのDNSサーバ設定を明示しない限り、クラスタ内DNSを通じて名前解決を行う
* クラスタ内DNSはServiceのEndpointに関するレコード(*.cluster.local)しか登録されていない
  - それ以外のレコードは外部DNSへ再起問い合わせを行うようになっている

## ClusterIP Service
* ClusterIP宛の通信は各ノード上で実行しているシステムコンポーネントのkube-proxyがPodに転送を行う
* 作成済みのServiceに対してClusterIPを変更する場合、削除してから再作成しなければならない

## ExternalIP Service
* ExternalIPといっても `type: ExternalIP` を指定するわけではない
* `spec.ExternalIPs` にはKubernetes NodeのIPを指定する
  - また、ExternalIPsに利用するIPは全てのKubernetes Nodeが持つIPを記述する必要はない
  - 利用できるIPはノードの情報から確認できる
  - `type: NodePort` を指定すればGCEに割り当てられたIPアドレスを利用できる 

## NodePort Service
* NodePortは全てのKubernetes Nodeが持つIPアドレスとPortで受信したトラフィックをコンテナに転送することで外部との疎通性を確立する
* 厳密な挙動としてはListen時に `0.0.0.0/0:Port` を使用して全てのIPアドレスでBindしている
* `spec.ports[].port` はClusterIPが受け付けるPort番号を指定する
* `spec.ports[].targetPort` は転送先コンテナのPort番号を指定する
* `spec.ports[].nodePort` はKubernetes Nodeが受け付けるPort番号を指定する 
* 注意点
  - NodePortが利用できるPort番号は、多くのKubernetes環境において `30000-32767` となっている
    - これはKubernetes Masterの設定を変更すればカスタマイズ可能
  - 複数のNodePort Serviceで同じPortを利用することはできない

### Node間通信の排除
* DaemonSetなど1ノードにつき1つのPodしか配置されない設定の場合、あえて他のKubernetes NodeにあるPodへ転送させたくないことがある
  - その場合、 `spec.externalTrafficPolicy` を使用すれば実現できる
    - 設定値は `Cluster` を `Local` にする
    - 後者の場合、ターゲットのノードに該当するPodが無いとレスポンスを返せなくなるので注意が必要
    - ただし、同じノードに2つ以上のPodがある場合はその2つのPodへ均等に割り振られる

## LoadBalancer Service
*  Endpointに割り当てる仮想IPアドレスの指定は `spec.LoadBalancerIP` で静的に指定できる

### Firewall Ruleの設定
* `spec.LoadBalancerSourceRanges` に接続許可したいIPアドレスの範囲を指定できる
  - これが未指定の場合は `0.0.0.0/0` が設定される
* LoadBalancer側でのアクセス制御が存在しない環境であればNetworkPolicyの利用を検討すること
  - これを利用するとKubernetes Nodeのiptablesを利用してアクセス制御を行える
  - ただし、スケーラビリティやレイテンシへの影響が出やすいため極力使用しないほうがいい

## Headless Service(None)
* 特徴として、ロードバランシングするためのIPアドレスが提供されない
  - また、DNS Round Robinを使ったエンドポイントを提供する
    - これは転送先のPodが持つIPアドレスがクラスタ内DNSから返ってくる形で負荷分散が行われるため、クライアント側でのキャッシュに注意が必要
* StatefulSetがHeadless Serviceを利用している場合、Podの名前でIPアドレスをディスカバリすることが可能となる
* Headless Service作成の条件
  - Serviceの `spec.type: ClusterIP`が設定されていること
  - Serviceの `spec.clusterIP: None`が設定されていること
  - Serviceの `metadata.name` がStatefulSetの `spec.serviceName` と同じであること

## ExternalName Service
* Serviceの名前解決に対して外部ドメイン宛のCNAMEを返す
* SaaSなどを利用する際に、アプリケーション内で外部エンドポイントを設定してしまうと切替時にアプリケーションの設定変更が必要になる
  - ExternalNameを利用することでアクセス内の切替はExternalNameを変更するだけでよくなる
  - これにより疎結合性を高めることができる
* ClusterIPからExternalNameに切り替える場合、 `spec.clusterIP` を明示的に空にする必要がある

## None-Selector Service
* Kubernetes内部に対して自由な宛先のロードバランシングを構成できる
* 主に利用するケースは `type: ClusterIP` の時
* これを利用する際はServiceリソースとEndpointリソースを個別に作成する必要がある
  - また、ServiceにSelectorを指定してはならない

## Ingress
* ServiceはL4ロードバランサーを提供するリソース
* IngressはL7ロードバランサーを提供するリソース
* Ingressの実装は複数ある
  - 代表的なものとしてGKE Ingress ControllerとNging Ingress Controllerがある

### Nginx Ingress
* これはクラスタ内にIngress用のPodをデプロイする
  - Ingress用のPodがHTTPS終端やパスベースルーティングといったL7相当の処理を行えるよう負荷に応じてPodのレプリカ数をオートスケーリングさせる必要がある
* Nginx Ingressを利用するにあたって起動させる必要があるもの
  - nginx-ingress-controllerのDeployment
  - nginx-ingress-controllerのService
  - default-http-backendのDeployment
  - default-http-backendのService
* 負荷を処理できるようにHPA(Hprizontal Pod Autoscaler)の利用も検討すべき
* ルールに一致しない場合にデフォルトで転送させるページを用意しておくこと
* Service AとService Bそれぞれとそれらに紐づくIngressがあった場合、衝突する可能性がある
  - これを避けるためにIngress Classのアノテーションを付与する必要がある
    