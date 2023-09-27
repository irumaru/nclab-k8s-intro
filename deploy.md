# 必要なマシン
コントロールプレーンとなるUbuntuServerを3台用意  
ワーカーノードとなるUbuntuServerを2台用意  
ロードバランサーとなるUbuntuServer(コンテナでも可)を1台用意
管理・デプロイ用にUbuntuServerを1台用意 

# クラスターのデプロイ方法
## 1.必須ツールのインストール
管理用のマシンに、k0sctlとkubectlをインストールする。  
```
brew install k0sctl
brew install kubectl
```

## 2.ssh秘密鍵の用意
管理用のマシンに、コントロールプレーンとワーカーノードのsshアクセス用秘密鍵を用意する。  

## 3.api-server用LoadBalancerを用意
### 1.haproxyをインストールする
```
apt install haproxy
```
### 2.haproxyに設定を追記する
リポジトリのk0s-cluster/haproxy.cfgの内容を/etc/haproxy/haproxy.cfgの末尾に追記する。  

## 4.k0sctlコマンドでクラスターをデプロイ
```
k0s-cluster$ k0sctl apply --config cluster.yaml
```
※コントロールプレーンとワーカーノードとロードバランサーのIPアドレスは適切な値へ変更する。  

## 5.k8s管理用シークレットを管理用マシンへ設定
```
k0s-cluster$ k0sctl kubeconfig --config cluster.yaml > ~/.kube/config
```

# k8s関連エコシステムのデプロイ

## 1.Metallbのデプロイ
### 1.チャートを展開する
```
metallb/helmfile$ helmfile template > template.yaml
```
### 2.展開したチャートをデプロイする
```
metallb/helmfile$ kubectl apply -f template.yaml
```
### 3.IPアドレスプールの追加
```
metallb$ kubectl apply -f l2-config.yaml
```

# アプリケーションのデプロイ
```
http-test$ kubectl apply -f namespace.yaml
http-test$ kubectl apply -f deployment.yaml
http-test$ kubectl apply -f service.yaml
```
