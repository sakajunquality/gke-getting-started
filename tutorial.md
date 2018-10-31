# GKE Getting Started

## Agenda

- 1. GKE クラスターの作成
- 2. kubectl 入門
- 3. アプリケーションのデプロイ
- 4. アプリケーションのアップデート
- 5. 2つ目のアプリケーションのデプロイ

# 1. GKE クラスターの作成

GKEのクラスターを作成します。


## 1.1 ProjectID の指定

```bash
export PROJECT_ID=my-cool-project
```

- ※ 自身のプロジェクト名を指定してください
- ※ 途中でやり直す場合はこちらを確認ください


```bash
gcloud config set project $PROJECT_ID
```

## 1.2 必要なAPIの有効化

必要になるAPIを有効化します。

```bash
gcloud services enable \
  cloudapis.googleapis.com \
  container.googleapis.com \
  containerregistry.googleapis.com \
  cloudbuild.googleapis.com
```

## 1.3 GKE クラスタの作成

GKEのクラスターを作成します。ハンズオンなのでノードは1台にしてあります。

```bash
gcloud container clusters create my-hands-on-cluster --num-nodes=1 --zone=asia-northeast1-b --async
```

### 1.4 クラスターの確認

作成されてたクラスターを確認します。

```bash
gcloud container clusters list
```

# 2. kubectl 入門

kubernetes-cliについて簡単に説明します

## 2.1 GKE クラスターに接続

```bash
gcloud container clusters get-credentials my-hands-on-cluster --zone asia-northeast1-b
```

## 2.2 kubectlでいくつか操作を行う

```bash
kubectl get nodes
```

```bash
kubectl get pods
```

# 3 アプリケーションのデプロイ

では実際にアプリケーションをデプロイしてみましょう

## 3.0 ハンズオンのアプリケーションについて

アプリケーションは `app1` のディレクトリに置いてあります。

```bash
cd app1
```

main.go だけのシンプルなGoのwebサーバーです。

```bash
cat main.go
```

コンテナにするためのDockefileも用意しています。

```bash
cat Dockerfile
```

## 3.1 ビルド

Cloud Buildを使用してビルドを行います。

```bash
gcloud builds submit --tag=gcr.io/$PROJECT_ID/hands-on-app-1:v1 .
```

```bash
cd ..
```

## 3.2 デプロイする

```bash
kubectl apply -f manifests/deployment.yaml
```

## 3.2 サービスを公開する

### 3.2.1 静的IPの確保
ロードバランサーに割り当てるIPアドレスを予め予約しておきます。

```bash
gcloud compute addresses create hands-on-ip \
     --global \
    --ip-version IPV4
```

### 3.2.2 GKEのserviceの作成

```bash
kubectl apply -f manifests/service.yaml
```

### 3.2.3 Ingress経由でロードバランサーの作成

```bash
kubectl apply -f manifests/ingress.yaml
```


# 4. アプリケーションのアップデート

## 4.1 アプリケーションの変更

- helloのメッセージを変更する
- 環境変数を使用して設定するようにする

変更...

## 4.2 ビルド

```bash
cd app1
```

v2としてビルド

```bash
gcloud builds submit --tag=gcr.io/$PROJECT_ID/hands-on-app-1:v2 .
```
## 3.2 デプロイする

### 3.2.1 環境変数をセットする

```bash
kubectl apply -f manifests/configmap.yaml
```

### 3.2.2 イメージを更新する

```bash
kubectl apply -f manifests/deployment.yaml
```

# 5. 2つ目のアプリケーションのデプロイ

```bash
cd app2
```

## 5.1 ビルド

```bash
gcloud builds submit --tag=gcr.io/$PROJECT_ID/hands-on-app-2:v1 .
```

## 5.2 デプロイする

```bash
kubectl apply -f manifests/deployment.yaml
```

## 5.1 パスルーティング

```bash
kubectl apply -f manifests/ingress.yaml
```

# 6. 掃除
最後に、作成したリソースの削除を行う。

## 6.1 Kubernetes リソースの削除

```bash
kubectl delete -f manifests/ingress.yaml
```

```bash
kubectl delete -f manifests/service.yaml
```

```bash
kubectl delete -f manifests/deployment.yaml
```

```bash
kubectl delete -f manifests/config.yaml
```

## 6.2 GCP リソースの削除

```bash
gcloud container clusters delete my-hands-on-cluster --zone=asia-northeast1-b --async
```
