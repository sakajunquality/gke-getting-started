# GKE Getting Started Part.2

## Agenda

Tutorial Part.1 を終えている前提にします。
もし終了した場合は、 `1. GKE クラスターの作成` を再度お願いします。

- 1. 簡易Webアプリケーションの作成
- 2. docker-composeからの移行
- 3. skaffoldの紹介
- 4. skaffoldを利用した開発

# 1. 簡易Webアプリケーションの作成

Part 1のapp1とほぼ同じアプリケーションを用意しました。
app3ディレクトリにあります。

## 1.1 アプリを確認します

```bash
cd ./app3
```

## 1.2 docker-composeでの起動してみる

docker-compose.yamlの `[my-project-id]` をご自身のプロジェクトIDに変更してください

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

# 3. skaffoldの紹介


# 4. skaffoldを利用した開発

## 4.1 GKEのクレデンシャルを確認

```bash
kubectl config current-context
```

もし取得できていない場合は、、、

```bash
skaffold ru
```

## 4.2 runを試す

```bash
skaffold run
```

## 4.2 devを試す

```bash
skaffold dev
```

## 4.3 (おまけ) Cloud Build でビルドを行うようにする
