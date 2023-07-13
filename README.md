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
- [x] Выставлен label **kubernetes-intro** с темой домашнего задания


## Домашнее задание №2

## В процессе сделано:
  - Пункт 1
1. Был создан первый манифест ReplicaSet [frontend-replicaset.yaml](kubernetes-controllers%2Ffrontend-replicaset.yaml) из методички HW2
  
   1. Исправлены ошибки в манифесте. Отсутствовала секция важная секция selector. 
   2. В манифест так же добавлены переменные (env) которые необходимы подам для работоспособности.
   3. Через команду  `kubectl apply -f frontend-replicaset.yaml` запустили репликасет. Убедились что поднялась одна реплика пода **frontend** командой `kubectl get rs frontend`
      ```
      PS...\MelnikovAlexey_platform>  kubectl get rs frontend
      NAME       DESIRED   CURRENT   READY   AGE
      frontend   1         1         1       46s
      ```
   4. Изменили количество реплик с 1 на 3 через команду `kubectl scale replicaset frontend --replicas=3`. Убедились что количество реплик пода **frontend** изменилось на 3 командой `kubectl get rs frontend` 
      ```
      PS...\MelnikovAlexey_platform>  kubectl get rs frontend
      NAME       DESIRED   CURRENT   READY   AGE
      frontend   3         3         3       46s
      ```
   5. Через команды  `kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w` убедились что репликасет отслеживает изменения состояний подов и поднимает новые реплики в случае если под по какой-то причине зафейлился. В нашем случае они были специально удалены.
      ```
      PS...\MelnikovAlexey_platform> kubectl delete pods -l app=frontend | kubectl get pods -l app=frontend -w
      NAME             READY   STATUS              RESTARTS   AGE
      frontend-8x8m2   0/1     ContainerCreating   0          2s
      frontend-bjsb5   0/1     ContainerCreating   0          2s
      frontend-xv6bd   0/1     ContainerCreating   0          2s
      frontend-xv6bd   1/1     Running             0          3s
      frontend-8x8m2   1/1     Running             0          3s
      frontend-bjsb5   1/1     Running             0          4s
      ```
   6. Изменен манифест чтобы сразу поднималось 3 реплики пода.
   7. Изменена версия образа с которого репликасет поднимает под. После применения манифеста командой `kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'` убеждаемся что образ с которого будут создаваться реплики подов изменился на указанную нами в манифесте версию
      ```
      PS...\MelnikovAlexey_platform\kubernetes-controllers> kubectl get replicaset frontend -o=jsonpath='{.spec.template.spec.containers[0].image}'
      avecake/frontend:0.0.2
      ```
      Проверяем образ с которого запущен под командой `kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'`
      ```
      PS...\MelnikovAlexey_platform\kubernetes-controllers> kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'
      avecake/frontend:0.0.1 avecake/frontend:0.0.1 avecake/frontend:0.0.1
      ```
      образ подов не был изменен. Образ пода будет изменен когда репликасет его пересоздаст. В нашем случае мы удаляем под вручную командой `kubectl delete pods -l app=frontend`. После этого снова проверяем с какого образа был развернут под.
      ```
      PS...\MelnikovAlexey_platform\kubernetes-controllers> kubectl get pods -l app=frontend -o=jsonpath='{.items[0:3].spec.containers[0].image}'
      avecake/frontend:0.0.2 avecake/frontend:0.0.2 avecake/frontend:0.0.2
      ```