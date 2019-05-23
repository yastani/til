# CI/CDに関する知見

## Code Commit

### BitbucketやGithubのリポジトリミラーリングには対応している？
- 2019年5月時点では対応していないのでCode Commitでソース管理しなければならなくなる

## Code Pipeline

### リポジトリに対するトリガーの条件定義はどこまで柔軟に設定できるのか？
- 例えば、Push・PR作成・Merge・Tagなど
- また、これらはブランチ単位で定義できるのか？
  - Pushしか対応していない

## Code Build

### ECRにビルドイメージをPushできるのと同じようにS3にビルドファイル(アーティファクトファイル)をUploadできるのか？
- できる

### Slack通知したい
- Cloudwatch Events -> SNS -> Lambdaでできる

## Code Deploy

### S3にビルドファイルをUploadできたとして、どこまで自動化できるのか？

#### ここまでできる
- Agentを使用して当該サーバにビルドファイルを置ける

#### できるといいな
- ベースとなるAMIを元にPackerのように新しいバージョンのAMIを作成できるのか？
- Launch Configurationを新たに生成できるのか？
- Launch Configurationを元にAutoscaling Groupを生成できるのか？
- Autoscaling Groupを対象のロードバランサーにアタッチできるのか？
- アタッチ後、サーバープロビジョンできるのか？
- プロビジョンが完了後、トラフィックマネジメントができるのか？
  - 現行イメージのサーバ内構成をコピーしたAutoscaling Groupを生成する
  - それを対象のロードバランサーにアタッチできる
  - 加えてトラフィックマネジメントもできる
  - 最新イメージを取得したくなったタイミングでAMIを作成すればよい

#### Slack通知したい
- Webhookを飛ばして何らかの方法でLambdaに渡せばできる
- Slackに通知後、デプロイしたりローリングアップデートしたりを人間が判断できる？
  - デプロイはApprovalリンクを踏んで権限があればできる
  - ローリングアップデートはchatbotなどでAPIを叩けば実行するのでそれでできる
    - https://docs.aws.amazon.com/ja_jp/codepipeline/latest/userguide/approvals-approve-or-reject.html#approvals-approve-or-reject-cli
