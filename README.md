
```shell
## create namespace
$ oc new-project test1

## 新增 nodejs application
## 新增 java application

## reate namespace
$ oc new-project test2

## 新增 python application
## 新增 Go application
```

## 驗證

```shell
$ oc project test1
$ oc get po
NAME                                     READY   STATUS      RESTARTS   AGE
java-springboot-basic-1-build            0/1     Completed   0          13m
java-springboot-basic-6d9b7d45f5-zdqdc   1/1     Running     0          13m
nodejs-basic-1-build                     0/1     Completed   0          58m
nodejs-basic-6474868d8-zkl5j             1/1     Running     0          58m

## 確認 service name
$ oc get svc
NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
java-springboot-basic   ClusterIP   172.30.89.16   <none>        8081/TCP   13d
nodejs-basic            ClusterIP   172.30.185.9   <none>        3001/TCP   18h

## 進入 java container
$ oc rsh java-springboot-basic-6d9b7d45f5-zdqd
sh-4.4$ curl nodejs-basic:3001
Hello from Node.js Starter Application!
sh-4.4$ 

## 進入 python container
$ oc project2
$ oc get po
NAME                            READY   STATUS      RESTARTS   AGE
python-basic-1-build            0/1     Completed   0          106s
python-basic-567cb6ff48-rnr7m   1/1     Running     0          105s

$ oc rsh python-basic-567cb6ff48-rnr7m
(app-root) sh-5.1$ curl nodejs-basic.test1.svc:3001
Hello from Node.js Starter Application!
(app-root) sh-5.1$
```

## 套用deny all

```shell
$ oc project test1
$ oc apply -f deny-all.yaml 

## 回到 python container
$ oc project test2
$ oc rsh python-basic-567cb6ff48-rnr7m
(app-root) sh-5.1$ curl nodejs-basic.test1.svc:3001
curl: (28) Failed to connect to nodejs-basic.test1.svc port 3001: Connection timed out
##### no response

## 進入 java container
$ oc project test1
$ oc rsh java-springboot-basic-6d9b7d45f5-zdqd
sh-4.4$ curl nodejs-basic:3001
curl: (7) Failed to connect to nodejs-basic port 3001: Connection timed out
```

## 套用allow-same-namespace
只有相同namespace 的pod 可以通
```shell
$ oc project test1
$ oc apply -f allow-same-namespace.yaml

$ oc rsh java-springboot-basic-6d9b7d45f5-zdqd
sh-4.4$ curl nodejs-basic:3001
Hello from Node.js Starter Application!

$ oc project test2
$ oc rsh python-basic-567cb6ff48-rnr7m
(app-root) sh-5.1$ curl nodejs-basic.test1.svc:3001
curl: (28) Failed to connect to nodejs-basic.test1.svc port 3001: Connection timed out
```

## 套用 allow-from-openshift-ingress

```shell
$ oc project test1
$ oc apply -f allow-from-openshift-ingress.yaml 

$ oc project test2
$ oc rsh python-basic-567cb6ff48-rnr7m
(app-root) sh-5.1$ curl nodejs-basic.test1.svc:3001
curl: (28) Failed to connect to nodejs-basic.test1.svc port 3001: Connection timed out

## 進入 java container
$ oc project test1
$ oc rsh java-springboot-basic-6d9b7d45f5-zdqd
sh-4.4$ curl nodejs-basic:3001
curl: (7) Failed to connect to nodejs-basic port 3001: Connection timed out
```

## 套用 allow-ingress-same-ns.yaml + allow-3001.yaml

```shell
$ oc project test1
$ oc apply -f allow-ingress-same-ns.yaml
$ oc apply -f allow-3001.yaml

$ oc get networkpolicy
NAME                    POD-SELECTOR       AGE
allow-3001              app=nodejs-basic   12d
allow-ingress-same-ns   <none>             12d
```