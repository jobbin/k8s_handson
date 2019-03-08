# k8s_handson
Handson of k8s.

## 【 Step 0 】RDSの作成

## 【 Step 1 】ローカルDBを利用したWordpressの構築

### ローカルDBの作成

1. ローカルDB(MySQL)接続時の情報を保存するため、Secretを作成
```
# kubectl create secret generic local-mysql-secret-info --from-literal=password=${Your local db password}
```

1. MySQLのPVを作成する
```
# kubectl apply -f mysql-pv.yml
persistentvolume/mysql-pv created
```

1. MySQLのPVCを作成する
```
# kubectl apply -f mysql-pvc.yml
persistentvolumeclaim/mysql-pvc created
```
1. MySQLのDeploymentを作成する
```
# kubectl apply -f mysql.yml
deployment.extensions/mysql created
```
1. Wordpressからアクセスできるように、MySQLのServiceを作成する
```
# kubectl apply -f mysql-service.yml
service/mysql created
```
1. MySQLの諸確認
```
# kubectl get deploy,pv,pvc,svc
```

### Wordpressの作成

1. WordpressのPVを作成する
```
# kubectl apply -f  wordpress-pv.yml
```
1. WordpressのPVCを作成する
```
# kubectl apply -f  wordpress-pvc.yml
```
1. WordpressのDeploymentを作成する
```
# kubectl apply -f  wordpress.yml
```
1. 外部からアクセスできるように、WordpressのServiceを作成する
```
# kubectl apply -f  wordpress-service.yml
```
1. Wordpressの諸確認
```
# kubectl get deploy,pv,pvc,svc
```
1. a
```
# kubectl get svc
```

### 【 Step 2 】ローカルDBからRDSへ切り替え

1. RDSの接続情報を設定する
```
export SECRET_NAME=mysql-secret-info
export DB_USER=wordpress
export DB_HOST={Your RDS intranet host}
export DB_PASSWORD={Your DB password}
```

1. RDSの接続情報をSecretに保存する
```
# kubectl create secret generic ${SECRET_NAME} \
--from-literal=DB_USER=${DB_USER} \
--from-literal=DB_HOST=${DB_HOST} \
--from-literal=DB_PASSWORD=${DB_PASSWORD}
```

1. WordpressのDeploymentを作り直す
```
kubectl apply -f wordpress.yml
```

1. WordpressのServiceを作り直す
```
kubectl apply -f wordpress-service.yml
```

1. 外部からアクセスしてみる

## Ingressを利用したBlue/Green Deployment

1. Blue環境のIngressを作成する
```
# kubectl apply -f ingress_blue.yml
```

1. Blue環境へアクセスしてみる
```
curl -H "Host:wordpress.io" {IP}
```

1. Green環境のNginxを作成する
```
# kubectl create deployment nginx --image nginx
deployment.apps/nginx created
```

1. Ingressからアクセスできるように、NginxのServiceを作成する
```
# kubectl expose deployment nginx --type NodePort --port 80
service/nginx exposed
```

1. Blue環境とGreen環境へのアクセスはそれぞれ50%にできるようなIngressを作成する
```
# kubectl apply -f ingress-blue-green.yml
ingress.extensions/ingress configured
```

1. Green環境へ切り替える
```
# kubectl apply -f ingress-blue-green.yml
ingress.extensions/ingress configured
```
