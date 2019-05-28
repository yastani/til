# Docker Meetup Kansai #3

Date: 2019-05-24

## DockerCon SF2019まとめ

### Docker Enterprise 3.0

### Docker Kubernetes Services

開発からプロダクションまで一貫してKubernetesとDockerを使えるよ

### Docker/buildx

次世代のBuild kit（buildkitの機能を使える）
```
docker buildx build -t a .
```

## Dockerfile ベストプラクティス2019

Best practices for writing Dockerfiles

docker buildはもうすぐDepricationされる
ビルドにキャッシュ機能があるが、Stepに変更があると作り直される

Docker BuildKit v2を使ってみよう
可能ならすぐ使うべき、なぜならビルドの速さが段違い

### Dockerfileの改善
ビルド時間、イメージ容量、メンテナンス性　等

- キャッシュの順番に気をつける(COPYとか)
- キャッシュ破棄の影響を避けるために範囲を狭める
- 可能であれば `COPY .` を避ける
- 行をまとめる(\ &&)
- 古いパッケージキャッシュを使わない
- 不要な依存関係を削除する
  - `-no-install-recommends`
  - `rm -fr /var/lib/apt/lists/*`
- できるだけ公式パッケージを使う
- タグをlatestではなく明示する
- 公式パッケージの中でも必要最小限のものを探す
- production向けではalpineタグ付きのほうがいい
- 開発環境からDockerfileをみんなで使おう
- 依存関係の解決にはステップを分けるようにする
  - マルチステージビルド 
    - `AS builder`
    - `COPY from builder`
    - FROM句が2つ
    - 利用例
      - 実行環境と構築環境を分ける
      - 依存関係の並列化
      - プラットフォーム固有のステージ
        - 構築ステージの分け方は `docker build --target STAGE`
   - イメージのflavorを変える(global ARG)
   - Dockerfileを分けるよりflavorでコントロールしよう
   - BuildKitのマルチステージはいいぞ
     - 不要なステージを無視する

### Dockerfileの新機能
contextマウント `RUN --mount=target=.`
mount-type=cache
Secretもある

## JupyterhubのDockerspawner

### Jupyterhub
Jupyter Notebookをトークン認証ではなくマルチユーザー環境にする
Docker + JupyterNotebookよりもユーザ単位の環境をセキュアにできる

## A/B test with Docker
簡単なA/Bテストなら主流のツールでいいが、凝ったA/Bテストには向いていない。

## オンプレでPrivate Registryを使ったDockerイメージの運用について
Direct Pollは面白い

## Docker開発環境のMac対応に苦戦した話
Linux用のDocker-composeをMacで動かす
R/Wが遅い

## コンテナ使い見習いのレベル上げ
大阪でDocker書籍読書会やってるよ

## コンテナとオーケストレーションと営業と
営業視点でのコンテナオーケストレーション提案