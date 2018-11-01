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
gcloud container clusters create my-hands-on-cluster --enable-ip-alias --num-nodes=1 --zone=asia-northeast1-b --async
```

### 1.4 クラスターの確認

作成されてたクラスターを確認します。

```bash
gcloud container clusters list
```

[GKEのコンソール](https://console.cloud.google.com/kubernetes/list)でも確認することができます。

しばらくすると、STATUSがRUNNINGになります。

# 2. kubectl 入門

次に、kubectl(kubernetes-cli)について簡単に説明します。

## 2.1 GKE クラスターに接続

```bash
gcloud container clusters get-credentials my-hands-on-cluster --zone asia-northeast1-b
```

## 2.2 kubectlでいくつか操作を行う

GKEのノードの一覧を出してみましょう。

```bash
kubectl get nodes
```

また動いてるポッドの一覧を取得します。

```bash
kubectl get pods
```

※ 新規なのでリソースは空っぽです。

## 2.3 ハンズオン用の namespace の作成

今回作業用のnamespaceを作成します。

```bash
kubectl create namespace tutorial
```

```bash
kubectl get namespaces
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

[Cloud Build](https://console.cloud.google.com/cloud-build/builds) の画面でもビルドの結果が確認できます。


ビルドが終わったらイメージを確認しましょう。

```bash
gcloud container images list
```

また、ビルドされたイメージは、[Container Registry](https://console.cloud.google.com/gcr/images/ubie-sandbox)でも確認することができます。

ビルドが終わったのでディレクトリを移動します。

```bash
cd ..
```

## 3.2 デプロイする

`manifests/deployment.yaml` の `[PROJECT_ID]` の部分を使用してるプロジェクト名に書き換えてください

書き換えが終わったら、GKEに反映を行います。

```bash
kubectl apply -f manifests/deployment.yaml
```

反映したら結果を確認しましょう。

```bash
kubectl -n tutorial rollout status deployment/app1
```

動いているpodも確認します。

```bash
kubectl -n tutorial get deployments
```

```bash
kubectl -n tutorial get pods
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

```bash
kubectl -n tutorial get svc
```

### 3.2.3 Ingress経由でロードバランサーの作成

```bash
kubectl apply -f manifests/ingress.yaml
```


# 4. アプリケーションのアップデート

## 4.1 アプリケーションの変更

- helloのメッセージを変更する
- 環境変数を使用して設定するようにする

importsの中に`"os"`を加え、
`fmt.Fprintf(w, "Hello!")` を `fmt.Fprintf(w, os.Getenv("HELLO_MESSAGE"))`
に書き換えを行ってください。

## 4.2 ビルド

```bash
cd app1
```

v2としてビルド

```bash
gcloud builds submit --tag=gcr.io/$PROJECT_ID/hands-on-app-1:v2 .
```

```bash
cd ..
```

## 3.2 デプロイする
ではv2のバージョンをデプロイしましょう。

### 3.2.1 環境変数をセットする

```bash
kubectl apply -f manifests/configmap.yaml
```

### 3.2.2 イメージを更新する


`v1` を `v2` に書き換え、

`containers`の中に次の設定を追記します。
```
        envFrom:
        - configMapRef:
            name: app1-conf
```


```bash
kubectl apply -f manifests/deployment.yaml
```


ローリングアップデートが走っているのがわかります。

```
kubectl -n tutorial get pods
```

### 3.3 サービスにアクセスをする

もう一度サービスにアクセスしてみましょう。
メッセージが変わるはずです。

# 5. 2つ目のアプリケーションのデプロイ

```bash
cd app2
```

## 5.1 ビルド

```bash
gcloud builds submit --tag=gcr.io/$PROJECT_ID/hands-on-app-2:v1 .
```


```bash
cd ..
```

## 5.2 デプロイする

`[PROJECT_ID]` の部分を使用してるプロジェクト名に書き換えてください

```bash
kubectl apply -f manifests/deployment2.yaml
```

deploymentsが増えていて

```bash
kubectl -n tutorial get deployments
```

podが増えていることを確認しましょう。

```bash
kubectl -n tutorial get pods
```

## 5.3 サービスの公開

```bash
kubectl apply -f manifests/service2.yaml
```

## 5.4 パスルーティング

manifests/ingress.yamlのpathsに下記を追記します

```
      - path: /ping
        backend:
          serviceName: app2-service
          servicePort: 8082
```

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
kubectl delete -f manifests/service2.yaml
```

## 6.2 GCP リソースの削除

GKE クラスターの削除

```bash
gcloud container clusters delete my-hands-on-cluster --zone=asia-northeast1-b --async
```

静的 IP の削除

```bash
gcloud compute addresses delete hands-on-ip
```
