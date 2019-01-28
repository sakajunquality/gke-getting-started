# GKE Getting Started Part.2

## Agenda

- 0. はじめに
- 1. 簡易Webアプリケーションの作成
- 2. docker-composeからの移行
- 3. skaffoldの紹介
- 4. skaffoldを利用した開発
- 5. 片付け
- 6. まとめ

# 0. はじめに
Tutorial Part.1 を終えている前提にします。
もし終了した場合は、 `1. GKE クラスターの作成` を再度お願いします。


```bash
cloudshell launch-tutorial -d tutorial.md
```

このPart. 1でが終わればと途中でチュートリアルをやめ、このチュートリアルに戻ってきてください。

```bash
cloudshell launch-tutorial -d tutorial2.md
```

# 1. 簡易Webアプリケーションの作成

Part 1のapp1とほぼ同じアプリケーションを用意しました。
app3ディレクトリにあります。

## 1.1 アプリを確認します

```bash
cd ./app3
```

## 1.2 docker-composeでの起動してみる

docker-compose.yamlの `[PROJECT_ID]` をご自身のプロジェクトIDに変更してください

```bash
docker-compose up
```

## 1.3 docker-composeで起動したアプリに接続しましょう

Cloud Shellのポートフォワーディング機能で `8089` 番ポートに変更してみます。

## 1.4 docker-composeの終了

フォアグラウンドで動かしているので `Ctrl + c` で終了できます。

```bash
docker-compose down
```

# 2. docker-composeからの移行

[kubernetes/kompose](https://github.com/kubernetes/kompose) を使用することで、 `docker-compose.yaml` から Kubernetes用のマニフェストを生成することができます。
また、今回使用する [skaffold](https://github.com/GoogleContainerTools/skaffold) についてもインストールを行います。

## 2.1 komposeのインストール

バイナリをダウンロードしてパスが通っている場所に移動します。

```bash
curl -L https://github.com/kubernetes/kompose/releases/download/v1.17.0/kompose-linux-amd64 -o kompose && chmod +x kompose && sudo mv ./kompose /usr/local/bin/kompose
```

## 2.2 skaffoldのインストール

同様に、バイナリをダウンロードしてパスが通っている場所に移動します。

```bash
curl -Lo skaffold https://storage.googleapis.com/skaffold/releases/v0.21.1/skaffold-linux-amd64 && chmod +x skaffold && sudo mv skaffold /usr/local/bin
```

## 2.2. マニフェストの生成

```bash
skaffold init --compose-file docker-compose.yaml
```

## 2.3 マニフェストの確認

`app-deployment.yaml` と `app-service.yaml` というファイルが作られていて、それぞれ、

- `extensions/v1beta1.Deployment`
- `v1.Service`

のマニフェストが作成されています。
また、 `skaffold.yaml` というファイルも生成されていますが、これについては後ほど説明します。

# 3. skaffoldの紹介

# 4. skaffoldを利用した開発

## 4.1 GKEのクレデンシャルを確認

```bash
kubectl config current-context
```

もし取得できていない場合は、、、

```bash
gcloud container clusters get-credentials my-hands-on-cluster --zone us-west1-b
```

念の為確認します

```bash
kubectl config current-context
```

## 4.2 skaffoldの設定の確認
前項 2.3 にて生成された `skaffold.yaml` がskaffoldの設定ファイルです。

## 4.3 runを試す

run を実行することで、イメージのビルド・デプロイを行います。

```bash
skaffold run
```

## 4.4 ポートフォーワードで確認
今回はローカルではなく、GKE上で動いているため `kubectl` コマンドを使用して先程作成したサービスにポートフォワードします。

```bash
kubectl port-forward svc/app 8089:8089
```

そして、docker-composeの際と同様にCloud Shellのポートフォワード機能で確認します。
確認が終わったら Ctrl+cで止めます。

## 4.5 devを試す

今度は dev を試してみましょう

```bash
skaffold dev
```

## 4.6 ポートフォワードで確認

4.4項と同じ方法で確認を行います。

`Watching for changes every 1s...` とコンソールに出ている通り、
devではソースコードの変更を検知して自動ででビルドとデプロイを行ってくれます。

## 4.7 devを終了する

Ctrl+cで終了できますが、devで作成したものをまとめて削除してくれます。

# 5. 片付け

最後にGKEのクラスターを削除します。

```
gcloud container clusters delete my-hands-on-cluster --zone=us-west1-b --async
```

# 6. まとめ

環境起因のトラブルを避けるためすべてGCP上でハンズオンを行いましたが、
最近のDocker CEのKubernetesやminikubeを使うことでローカルでもskaffoldを使用することができます。
また、普段はminikubeを使って開発しつつ、他の人に確認してほしいときにGKEのクラスターに上げるということも可能です。
