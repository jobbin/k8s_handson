# k8s_handson
Handson of k8s.

## RDSの作成

## Container

### MySQL(local)の作成

1. secret作成
```
# kubectl create secret generic local-mysql-secret-info --from-literal=password=${Your local db password}
```

1. あ
```
# kubectl apply -f mysql-pv.yml
persistentvolume/mysql-pv created
```

1. pvc作成
```
# kubectl apply -f mysql-pvc.yml
persistentvolumeclaim/mysql-pvc created
```
1. mysql作成
```
# kubectl apply -f mysql.yml
deployment.extensions/mysql created
```
1. service
```
# kubectl apply -f mysql-service.yml
service/mysql created
```
1. 初確認
```
# kubectl get deploy,pv,pvc,svc
NAME                          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/mysql   1         1         1            1           3m
NAME                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS    CLAIM               STORAGECLASS   REASON    AGE
persistentvolume/mysql-pv   20Gi       RWO            Recycle          Bound     default/mysql-pvc                            8m
NAME                              STATUS    VOLUME     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/mysql-pvc   Bound     mysql-pv   20Gi       RWO                           7m
NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/kubernetes   ClusterIP   172.19.0.1     <none>        443/TCP    1d
service/mysql        ClusterIP   172.19.2.226   <none>        3306/TCP   38s
```

### Wordpressの作成

1. a
```
# kubectl apply -f  wordpress-pv.yml
```
1. a
```
# kubectl apply -f  wordpress-pvc.yml
```
1. a
```
# kubectl apply -f  wordpress.yml
```
1. a
```
# kubectl apply -f  wordpress-service.yml
```
1. a
```
# kubectl get deploy,pv,pvc,svc
```
1. a
```
# kubectl get svc
```

### RDS化

1. Secretに保存する情報を設定する
```
export SECRET_NAME=mysql-secret-info
export DB_USER=wordpress
export DB_HOST={Your RDS intranet host}
export DB_PASSWORD={Your DB password}
```

1. Secretを作成する
```
# kubectl create secret generic ${SECRET_NAME} \
--from-literal=DB_USER=${DB_USER} \
--from-literal=DB_HOST=${DB_HOST} \
--from-literal=DB_PASSWORD=${DB_PASSWORD}
```

## Ingressを利用したBlue/Green Deployment

1. a
```
# kubectl apply -f ingress_1.yml
```

1. Address払い出されたら
```
curl -H "Host:wordpress.io" {IP}
```

1. a
```
# kubectl create deployment nginx --image nginx
deployment.apps/nginx created
```

1. a
```
# kubectl expose deployment nginx --type NodePort --port 80
service/nginx exposed
```

1. b
```
# kubectl apply -f ingress-blue-green.yml
ingress.extensions/ingress configured
```

1. b
```
# kubectl apply -f ingress-blue-green.yml
ingress.extensions/ingress configured
```

