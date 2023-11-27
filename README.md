# Домашнее задание к занятию «Сетевое взаимодействие в K8S. Часть 1»

### Цель задания

В тестовой среде Kubernetes необходимо обеспечить доступ к приложению, установленному в предыдущем ДЗ и состоящему из двух контейнеров, по разным портам в разные контейнеры как внутри кластера, так и снаружи.

------

### Чеклист готовности к домашнему заданию

1. Установленное k8s-решение (например, MicroK8S).
2. Установленный локальный kubectl.
3. Редактор YAML-файлов с подключённым Git-репозиторием.

------

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Описание](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) Deployment и примеры манифестов.
2. [Описание](https://kubernetes.io/docs/concepts/services-networking/service/) Описание Service.
3. [Описание](https://github.com/wbitt/Network-MultiTool) Multitool.

------

### Задание 1. Создать Deployment и обеспечить доступ к контейнерам приложения по разным портам из другого Pod внутри кластера

1. Создать Deployment приложения, состоящего из двух контейнеров (nginx и multitool), с количеством реплик 3 шт.
2. Создать Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.
3. Создать отдельный Pod с приложением multitool и убедиться с помощью `curl`, что из пода есть доступ до приложения из п.1 по разным портам в разные контейнеры.
4. Продемонстрировать доступ с помощью `curl` по доменному имени сервиса.
5. Предоставить манифесты Deployment и Service в решении, а также скриншоты или вывод команды п.4.


Создаем пространство имен под ДЗ:
```
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl get ns
NAME              STATUS   AGE
kube-system       Active   13d
kube-public       Active   13d
kube-node-lease   Active   13d
default           Active   13d
lesson2           Active   6d14h
dz3               Active   5d23h
lesson3           Active   3d23h
metallb-system    Active   2d23h
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl apply -f ~/manifests/03_dz_kuber_1.4/01_namespace.yml
namespace/dz4 created
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl get ns
NAME              STATUS   AGE
kube-system       Active   13d
kube-public       Active   13d
kube-node-lease   Active   13d
default           Active   13d
lesson2           Active   6d14h
dz3               Active   5d23h
lesson3           Active   3d23h
metallb-system    Active   2d23h
dz4               Active   11s
```

Запускаем деплоймент с тремя репликами:
```
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl apply -f ~/manifests/03_dz_kuber_1.4/02_deploy_nginx_multitool.yml
deployment.apps/dpl-nginx-multitool-dz4 created
```

Смотрим ПОДы в созданном неймспейсе:
```
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl get pods -n dz4 -o wide
NAME                                       READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
dpl-nginx-multitool-dz4-6989c86479-z2h5s   2/2     Running   0          21s   10.1.198.96   microk8s-01   <none>           <none>
dpl-nginx-multitool-dz4-6989c86479-5ghcl   2/2     Running   0          21s   10.1.63.171   microk8s-02   <none>           <none>
dpl-nginx-multitool-dz4-6989c86479-2zdlr   2/2     Running   0          21s   10.1.63.170   microk8s-02   <none>           <none>
```

Пилим Service, который обеспечит доступ внутри кластера до контейнеров приложения из п.1 по порту 9001 — nginx 80, по 9002 — multitool 8080.
```
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl apply -f ~/manifests/03_dz_kuber_1.4/03_svc_nginx_multitool.yml
service/svc-nginx-multitool-dz4 created
```

Проверяем, что сервис поднялся и эндпоинты подвязались:
```
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl get svc -n dz4 -o wide
NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE   SELECTOR
svc-nginx-multitool-dz4   ClusterIP   10.152.183.36   <none>        9001/TCP,9002/TCP   10s   app=web
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl get ep -n dz4 -o wide
NAME                      ENDPOINTS                                                        AGE
svc-nginx-multitool-dz4   10.1.198.96:8080,10.1.63.170:8080,10.1.63.171:8080 + 3 more...   50s
```

Разворачиваем ПОД для тестового мультитула:
```
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl apply -f ~/manifests/03_dz_kuber_1.4/04_deploy_multitool.yml
pod/test-multitool created
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl get pods -n dz4 -o wide
NAME                                       READY   STATUS    RESTARTS   AGE     IP            NODE          NOMINATED NODE   READINESS GATES
dpl-nginx-multitool-dz4-6989c86479-z2h5s   2/2     Running   0          8m41s   10.1.198.96   microk8s-01   <none>           <none>
dpl-nginx-multitool-dz4-6989c86479-5ghcl   2/2     Running   0          8m41s   10.1.63.171   microk8s-02   <none>           <none>
dpl-nginx-multitool-dz4-6989c86479-2zdlr   2/2     Running   0          8m41s   10.1.63.170   microk8s-02   <none>           <none>
test-multitool                             1/1     Running   0          13s     10.1.198.97   microk8s-01   <none>           <none>
```

Тестируем, что наши ПОДы нам отвечают по ожидаемым портам:
```
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl exec -n dz4 test-multitool -- curl 10.1.198.96:80
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0  58603      0 --<!DOCTYPE html> --:--:--     0
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
:--:-- --:--:-- --:--:-- 61200
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl exec -n dz4 test-multitool -- curl 10.1.198.96:8080
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0WBITT Network MultiTool (with NGINX) - dpl-nginx-multitool-dz4-6989c86479-z2h5s - 10.1.198.96 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
100   158  100   158    0     0   6980      0 --:--:-- --:--:-- --:--:--  7181
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl exec -n dz4 test-multitool -- curl 10.1.63.171:8080
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0WBITT Network MultiTool (with NGINX) - dpl-nginx-multitool-dz4-6989c86479-5ghcl - 10.1.63.171 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
100   158  100   158    0     0   3740      0 --:--:-- --:--:-- --:--:--  3761
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl exec -n dz4 test-multitool -- curl 10.1.63.171:80
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
100   612  100   612    0     0  88452      0 --:--:-- --:--:-- --:--:--   99k
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl exec -n dz4 test-multitool -- curl 10.1.63.170:80
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0  94240      0 --:--:-- --:--:-- --:--:--   99k
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl exec -n dz4 test-multitool -- curl 10.1.63.170:8080
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0WBITT Network MultiTool (with NGINX) - dpl-nginx-multitool-dz4-6989c86479-2zdlr - 10.1.63.170 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
100   158  100   158    0     0  25991      0 --:--:-- --:--:-- --:--:-- 31600
```

Тестируем, что наш Сервис нам отвечает по ожидаемым портам:
```
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl exec -n dz4 test-multitool -- curl 10.152.183.36:9001
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   612  100   612    0     0  84764      0 --:--:-- --:--:-- --:--:-- 87428<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl exec -n dz4 test-multitool -- curl 10.152.183.36:9002
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0WBITT Network MultiTool (with NGINX) - dpl-nginx-multitool-dz4-6989c86479-5ghcl - 10.1.63.171 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
100   158  100   158    0     0  27175      0 --:--:-- --:--:-- --:--:-- 31600
```

Тестируем, что наш Сервис нам отвечает по имени:
```
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl exec -n dz4 test-multitool -- curl svc-nginx-multitool-dz4:9001
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
100   612  100   612    0     0   8611      0 --:--:-- --:--:-- --:--:--  8742
usrcon@cli-k8s-01:~/manifests/03_dz_kuber_1.4$ kubectl exec -n dz4 test-multitool -- curl svc-nginx-multitool-dz4:9002
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0WBITT Network MultiTool (with NGINX) - dpl-nginx-multitool-dz4-6989c86479-z2h5s - 10.1.198.96 - HTTP: 8080 , HTTPS: 443 . (Formerly praqma/network-multitool)
100   158  100   158    0     0  20447      0 --:--:-- --:--:-- --:--:-- 22571
```

------

### Задание 2. Создать Service и обеспечить доступ к приложениям снаружи кластера

1. Создать отдельный Service приложения из Задания 1 с возможностью доступа снаружи кластера к nginx, используя тип NodePort.
2. Продемонстрировать доступ с помощью браузера или `curl` с локального компьютера.
3. Предоставить манифест и Service в решении, а также скриншоты или вывод команды п.2.

------
