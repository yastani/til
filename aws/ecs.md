# ECS
Elastic Container Service

## 概念

### Task / TaskDefinition
Taskとはコンテナの集合体のこと。  
Task DefinitionとはどんなTaskを生成するかの定義を指す。

後者について下記のような定義をしておくことでTaskというコンテナの集合体を生成できる。
- どのイメージを使ってコンテナを立てるのか
- リソース割り当ての配分はどうするか
- 環境変数やポートなどの設定はどうするか

### Service
前述したTask / TaskDefinitionを依頼することを指す。  
つまりTaskDefinitionを参照してTaskをどのように生成するかを自動管理してくれる。

### Cluster
TaskとServiceをグルーピングする。  
区分としてはアプリケーション単位または環境単位など。  
ex)
- yourproject-front-dev
- yourproject-front-prd
- yourproject-back-dev
- yourproject-back-prd

### Fargate
ECSのデータプレーンとして使用できるサーバレスコンテナの技術。  
Taskに割り当てたリソースの量に応じた秒単位の課金。  
データプレーンにFargateを選択することでEC2インスタンスの管理が不要となり、柔軟なスケーリングが実現できる。

### ざっくりアーキテクチャ
- DockerImageはECRに上げる
- 上がったImageとTagの情報をTaskDefinitionに定義する
- ServiceがTaskを生成する
- ALBまたはNLBを立ててバランシングする