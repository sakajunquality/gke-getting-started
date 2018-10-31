# GKE Getting Started

## Agenda

- GKEクラスターの作成
- kubectl入門
- アプリケーションのデプロイ
- アプリケーションのアップデート
- ２つ目のアプリケーションのデプロイ

# 1. GKEクラスターの作成

## 1.0 gcloudについて

## 1.1 ProjectID の指定

```bash
gcloud config set project PROJECT_ID
```

## 1.2 必要なAPIの有効化

```bash
gcloud services enable compute.googleapis.com container.googleapis.com cloudbuild.googleapis.com
```

## 1.3 GKE クラスタの作成

```bash
gcloud container clusters create my-hands-on-cluster --num-nodes=1 --zone=asia-northeast1-b --async
```

# 2. kubectl入門

## 2.0 kubectlについて

## 2.1 GKEクラスターに接続

```bash
gcloud ...
```

## 2.2 kubectlでいくつか操作を行う

```bash
kubectl ...
```

# 3 アプリケーションのデプロイ

## 3.0 ハンズオンのアプリケーションについて

```bash
ls ...
```

## 3.1 ビルド

```bash
gcloud builds submit ...
```

## 3.2 デプロイする

```bash
kubectl ...
```

## 3.2 サービスを公開する

```bash
kubectl ...
```

# 4. アプリケーションのアップデート

## 3.1 ビルド

```bash
gcloud builds submit ...
```

## 3.2 デプロイする

```bash
kubectl ...
```

# 5. ２つ目のアプリケーションのデプロイ

## 5.1 ビルド

```bash
gcloud builds submit ...
```

## 5.2 デプロイする

```bash
kubectl ...
```

## 5.1 パスルーティング

```bash
kubectl ...
```

# 6. 掃除

## kubernetesのリソースの削除
```bash
kubectl ...
```
