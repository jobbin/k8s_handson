# k8s_handson
Handson of k8s.

## 【 Step 0 】RDSの作成

## 【 Step 1 】ローカルDBを利用したWordpressの構築

### ローカルDBの作成

- ディレクトリ 1-wordpress_local_db の配下に進む


- ローカルDB(MySQL)接続時の情報を保存するため、Secretを作成
```
# kubectl create secret generic local-mysql-secret-info --from-literal=DB_PASSWORD=${Your local db password}
```

- MySQLのPVを作成する
```
# kubectl apply -f mysql-pv.yml
persistentvolume/mysql-pv created
```

- MySQLのPVCを作成する
```
# kubectl apply -f mysql-pvc.yml
persistentvolumeclaim/mysql-pvc created
```

- MySQLのDeploymentを作成する
```
# kubectl apply -f mysql.yml
deployment.extensions/mysql created
```
- Wordpressからアクセスできるように、MySQLのServiceを作成する
```
# kubectl apply -f mysql-service.yml
service/mysql created
```
- MySQLの諸確認
```
# kubectl get deploy,pv,pvc,svc
```

### Wordpressの作成

- WordpressのPVを作成する
```
# kubectl apply -f  wordpress-pv.yml
```
- WordpressのPVCを作成する
```
# kubectl apply -f  wordpress-pvc.yml
```
- WordpressのDeploymentを作成する
```
# kubectl apply -f  wordpress.yml
```
- 外部からアクセスできるように、WordpressのServiceを作成する
```
# kubectl apply -f  wordpress-service.yml
```
- Wordpressの諸確認
```
# kubectl get deploy,pv,pvc,svc
```
- WordpressのServiceのIPを確認する
```
# kubectl get svc
```

### 【 Step 2 】ローカルDBからRDSへ切り替え

- ディレクトリ 2-wordpress_rds の配下に進む

- RDSの接続情報を設定する
```
export SECRET_NAME=mysql-secret-info
export DB_USER=wordpress
export DB_HOST={Your RDS intranet host}
export DB_PASSWORD={Your DB password}
```

- RDSの接続情報をSecretに保存する
```
# kubectl create secret generic ${SECRET_NAME} \
--from-literal=DB_USER=${DB_USER} \
--from-literal=DB_HOST=${DB_HOST} \
--from-literal=DB_PASSWORD=${DB_PASSWORD}
```

- WordpressのDeploymentを作り直す
```
kubectl apply -f wordpress.yml
```

- WordpressのServiceを作り直す
```
kubectl apply -f wordpress-service.yml
```

- 外部からアクセスしてみる

## Ingressを利用したBlue/Green Deployment

- Blue環境のIngressを作成する
```
# kubectl apply -f ingress_blue.yml
```

- IngressのIPを確認する
```
kubectl get ingres
```

- ローカル環境にて、Blue環境へアクセスしてみる
```
curl -H "Host:wordpress.io" {Ingress_IP}
```

- Green環境のNginxを作成する
```
# kubectl create deployment nginx --image nginx
deployment.apps/nginx created
```

- Ingressからアクセスできるように、NginxのServiceを作成する
```
# kubectl expose deployment nginx --type NodePort --port 80
service/nginx exposed
```

- Blue環境とGreen環境へのアクセスはそれぞれ50%にできるようなIngressを更新する
```
# kubectl apply -f ingress-blue-green.yml
ingress.extensions/ingress configured
```

- ローカル環境にて、Blue/Green環境へアクセスしてみる
```
curl -H "Host:wordpress.io" {Ingress_IP}
```

- Green環境へ切り替える
```
# kubectl apply -f ingress-blue-green.yml
ingress.extensions/ingress configured
```

- ローカル環境にて、Green環境へアクセスしてみる
```
curl -H "Host:wordpress.io" {Ingress_IP}
```
