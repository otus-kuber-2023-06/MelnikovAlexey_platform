# MelnikovAlexey_platform
MelnikovAlexey Platform repository

## Домашнее задание №1
- [x] **Основное ДЗ:**
Разберитесь почему все pod в namespace **kube-system** восстановились после удаления
#### Ответ:
Причина почему все pod-ы в namespace **kube-system** восстановились, заключается в том что все pod-ы за исключением coredns
являются [static pod-ами](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/) и управляются напрямую [kubelet-ом](https://kubernetes.io/ru/docs/concepts/overview/components/#kubelet).
Описание данных подов находится
````
~$ ls -l  /etc/kubernetes/manifests
total 16
-rw------- 1 root root 2470 Jul 10 11:24 etcd.yaml
-rw------- 1 root root 4076 Jul 10 11:24 kube-apiserver.yaml
-rw------- 1 root root 3395 Jul 10 11:24 kube-controller-manager.yaml
-rw------- 1 root root 1441 Jul 10 11:24 kube-scheduler.yaml
````
Pod **coredns** описан в деплойменте неймспейса kube-system.
Посмотреть его описание можно следующей командой.

`kubectl describe deployments coredns -n kube-system`

```
C:\Windows\System32>kubectl describe deployments coredns -n kube-system
Name:                   coredns
Namespace:              kube-system
CreationTimestamp:      Mon, 10 Jul 2023 21:24:26 +1000
Labels:                 k8s-app=kube-dns
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               k8s-app=kube-dns
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
....
```
В данном случае ReplicaSet установлен на 1. В случае каких либо сбоев [ReplicaController](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/#what-is-a-replicationcontroller)
автоматически восстановит pod в количестве одного экземпляра.

- [x] **Задание со \*** Выясните причину, по которой pod **frontend** находится в статусе
  **Error**

#### Ответ:
Для работоспособности pod frontend необходимо добавить недостающие переменные и присвоить им значения
```
      env:
        - name: PRODUCT_CATALOG_SERVICE_ADDR
          value: "productcatalogservice:3550"
        - name: CURRENCY_SERVICE_ADDR
          value: "currencyservice:7000"
        - name: CART_SERVICE_ADDR
          value: "cartservice:7070"
        - name: RECOMMENDATION_SERVICE_ADDR
          value: "recommendationservice:8080"
        - name: CHECKOUT_SERVICE_ADDR
          value: "checkoutservice:5050"
        - name: SHIPPING_SERVICE_ADDR
          value: "checkoutservice:5050"
        - name: AD_SERVICE_ADDR
          value: "adservice:9555"
```
**p.s.** значения переменных взято из манифеста GoogleCloudPlatform/microservices-demo модуля frontend приложенного в методичке к ДЗ.

## В процессе сделано:
- Пункт 1
1.  Был создан и описан Doсkerfile согласно основному заданию.
2. На основе описанного Dockerfile был создан образ [avecake/mynginx:1](https://hub.docker.com/layers/avecake/mynginx/1/images/sha256-43ee60d1f4d74edc531a502c7781dcbbed09fec84b43f13c2ac5f4d81cec600e?context=repo) и размещен в публичном репозитории hub.docker.com
3. на основе образа был написан манифест pod с label **web**  _web-pod.yaml_


- Пункт 2
1. на основе [Doсkerfile](https://github.com/GoogleCloudPlatform/microservices-demo/blob/main/src/frontend/Dockerfile) из репозитория [GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo) был создан образ [avecake/frontend:0.0.1](https://hub.docker.com/layers/avecake/frontend/0.0.1/images/sha256-683b23a64b4537af04e4705631d619822d31f5928e68424b3b65fd8ab9a9ae02?context=repo)  и размещен в публичном репозитории hub.docker.com
2. находясь в директории **.\kubernetes-intro** командой `kubectl run frontend --image avecake/frontend:0.0.1 --restart=Never --dry-run -o yaml > frontend-pod.yaml` был создан манифест **frontend-pod.yaml** на основе образа описанного в пункте выше.
3. командой `kubectl logs frontend` были выявлены причины почему pod **frontend** находится в статусе **Error**
4. Создан манифест **frontend-pod-healthy.yaml** в директории **.\kubernetes-intro**
5. В манифест **frontend-pod-healthy.yaml** добавлены недостающие переменные для того чтобы pod во время запуска находился в статусе **Running**

## Как запустить проект:
- Чтобы запустить основное ДЗ необходимо выполнить команду `kubectl apply -f web-pod.yaml` в директории **.\kubernetes-intro**
- Для данного пода необходимо пробросить порты командой `kubectl port-forward --address 0.0.0.0 pod/web 8000:8000`


- Чтобы запустить задание со * необходимо выполнить команду `kubectl apply -f frontend-pod-healthy.yaml` в директории **.\kubernetes-intro**

## Как проверить работоспособность:
-  Перейти по ссылке [http://localhost:8000](http://localhost:8000) должна отобразиться страница.

-  Выполнить команду `kubectl get pod frontend`
```
PS ...\kubernetes-intro> kubectl get pod frontend
NAME       READY   STATUS    RESTARTS   AGE
frontend   1/1     Running   0          58m
```

## PR checklist:
- [x] Выставлен **kubernetes-intro** с темой домашнего задания