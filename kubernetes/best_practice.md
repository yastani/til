# Best Practice

## イメージのテスト
dockerspecを使うと作成したimageに対してserverspecを走らせることができる。

## イメージのデプロイ

1. docker build
1. docker imageに対してテスト
1. テストが通ったらコンテナレジストリにpush

## イメージのタグ
latestは設定しない。  
設定してしまうと予期しないイメージのコンテナに切り買ってしまい、様々な差異が生じてしまうため。

## Deployment

### Rolling Updateの注意点
Podを入れ替える時、利用可能なPodが一気に減少するとレスポンスを返せなくなる可能性がある。  
なので下記のような設定を入れておいたほうがいい。
このような設定にすると新しいPodが作成され、利用可能になってから既存のPodを削除するようになる。
```
strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 10% # maximum number of Pods that can be created above the desired number of Pods
      maxUnavailable: 0% # maximum number of Pods that can be unavailable during the update process.
```

## マニフェストファイル

### 設計指針
設計指針には以下のようなパターンがある。
- システム全体を単一ディレクトリにまとめる
- システム全体を特定のサブシステム単位に分割する
- マイクロサービス単位に分割する
システムの規模感や管理するチームの役割に応じて最適化を図っていけばいい。