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
1. Был создан первый манифест ReplicaSet **frontend-replicaset.yaml** из методички HW2
  
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
   8. Добавлены deployment манифесты для paymentservice.
   9. Добавлен Probes для frontend


- Пункт 2
1. Добавлены deployment манифесты для paymentservice с двумя стратегиями
   1. Аналог blue-green:
      ```
      spec:
         strategy:
           type: RollingUpdate
           rollingUpdate:
             maxSurge: 0
             maxUnavailable: 1
      ```
   2. Reverse Rolling Update:
      ```
      spec:
         strategy:
           type: RollingUpdate
           rollingUpdate:
             maxSurge: 3
             maxUnavailable: 0
      ```
2 Добавлен node-exporter-daemonset.yaml.
  1. Если я правильно понял из документации и нескольких проблем на stackoverflow. За разворачивание на мастер ноде отвечает данная секция и правило
     ```
      spec:
        tolerations:
          - key: node-role.kubernetes.io/master
            operator: Exists
            effect: NoSchedule
     ```
     
## Домашнее задание №3
### Тема: Сетевая подсистема Kubernetes
### В процессе сделано.
###### Добавление проверок Pod
- из hw-1 в web-pod.yaml в секцию spec.containers добавили readinessProbe:
  ```
  ...
          readinessProbe:
            httpGet:
              port: 8000
              path: /index.html
  ...
  ```
- запустили под, он перешел в состояние
   ```
  NAME READY STATUS  RESTARTS AGE
   web  0/1   Running 0        1m32s
  ```
- из hw-1 в web-pod.yaml в секцию spec.containers добавили livenessProbe:
  ```
  ...
         livenessProbe:
            tcpSocket:
              port: 8000
  ...
  ```
###### Вопросы самопроверки
1. Почему следующая конфигурация валидна, но не имеет смысла?
   ```
   livenessProbe:
      exec:
        command:
           - 'sh'
           - '-c'
           - 'ps aux | grep my_web_server_process'
   ```
   - потому что она всегда выполнится и она ничего не проверяет. в этом выводе будет всегда как минимум сам `grep`
2. Бывают ли ситуации, когда она все-таки имеет смысл?

   - в таком виде нет. Если только не убрать сам grep из вывода: `... | grep -v grep`. В таком виде это даст нам понять существует ли данный процесс в контейнере.
###### Создание Deployment
- создали deployment на основе web-pod `web-deploy.yaml`. 
- удалили старый под.
- поднимаем pod через Deployment манифест `kubectl apply -f web-deploy.yaml`
- Поскольку мы не исправили ReadinessProbe, то поды, входящие в наш
  Deployment, не переходят в состояние Ready из-за неуспешной проверки.
- изменяем ReadinessProbe порт на 8000 и увеличиваем количество реплик до 3
- добавили описание стратегии развертывания. 
- Применили изменения `kubectl apply -f web-deploy.yaml`
###### Deployment | Самостоятельная работа
1.  Попробуйте разные варианты деплоя с крайними значениями maxSurge и maxUnavailable (оба 0, оба 100%, 0 и 100%)
    - Согласно документации с сайта kubernetes.io нельзя ставить `maxUnavailable` и `maxSurge` со значением 0 одновременно.

###### Создание Service | ClusterIP
- Создаем Service ClusterIP `web-svc-cip.yaml`
  - ClusterIP удобны в тех случаях, когда:
    - Нам не надо подключаться к конкретному поду сервиса
    - Нас устраивается случайное распределение подключений между подами
    - Нам нужна стабильная точка подключения к сервису, независимая от подов, нод и DNS-имен
- деплоим его в кластер и смотрим его ip `kubectl get service`
- заходим на vm minikube `minikube ssh` далее `sudo -i`
- делаем `curl  http://<CLUSTER-IP>/index.html` - работает
- делаем `ping <CLUSTER-IP>` - пинга нет
- делаем `iptables --list -nv -t nat` видим большой список правил связанных с kube
###### Включение IPVS
- включаем ipvs для kube-proxy "на живую" `kubectl -n kube-system edit configmaps kube-proxy`
  - затем в ключ `data.config.conf:|- mode: ""` вписываем значение ipvs и в ключ `ipvs.strictARP:` указываем true
  - удалим под `kube-proxy kubectl -n kube-system delete pod --selector='k8s-app=kube-proxy'`
  - описание работы и настройки ipvs в k8s: https://github.com/kubernetes/kubernetes/blob/master/pkg/proxy/ipvs/README.md
  - Причины включения strictARP: https://github.com/metallb/metallb/issues/153
  - входим снова на ноду, проверяем: `iptables --list -nv -t nat`
  - видим что у цепочек 0 preference
  - kube-proxy настроил все по-новому, но не удалил мусор
  - запуск kube-proxy --cleanup не помогает `kubectl -n kube-system exec kube-proxy-<pod> kube-proxy --cleanup <pod>`
  - далее начинаются танцы с бубном поскольку двигаясь по методичке мы будем по 10 раз переделывать предыдущие шаги начиная с того что убивать миникуб и разворачивать заново.
как только мы очищаем список правил и у нас перестает работать доступ в тырьнет (по крайней мере у меня здесь возникли сложности)
    - пока есть все эти правила подготавливаем необходимый инструментарий для дальнейшей работы. 
    - ставим `nano ipset ipvsadm net-tools` без этих инструментов невозможно продолжить.
    - все готово продолжаем
  - полностью очистим все правила iptables
    - создаем файл `/tmp/iptables.cleanup` с содержимым
      ```
      *nat
      -A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
      COMMIT
      *filter
      COMMIT
      *mangle
      COMMIT
      ```
    - применим конфигурацию: `iptables-restore /tmp/iptables.cleanup` немного ждем.
    - смотрим результат `iptables --list -nv -t nat`
  - лишние правила удалены, мы видим только актуальную конфигурацию, kube-proxy периодически делает полную синхронизацию правил в своих цепочках
  - проверяем наш сервис: `ipvsadm --list -n` и находим там правила распределения нагрузки.
  - снова делаем ping clusterIP, и видим что пинги пошли, все потому что ip теперь есть на интерфейсе `kube-ipvs0`
##### Работа с LoadBalancer и Ingress
###### Установка Metallb
- переходим к очередному большому куску неактуальной методичке, а именно установка Metallb
  - плюем на описание в методичке и идем на сайт [Metallb](https://metallb.universe.tf/installation/)
  - тут же мы видим причины почему необходимо выставить kube-proxy для ключа `ipvs.strictARP:` значение **_true_**
  - устанавливаем metallb `kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.13.10/config/manifests/metallb-native.yaml`
    - тут у меня начинаются очередные проблемы с развертыванием.
    - под контроллера в namespaces `metallb-system` в состоянии `ImagePullBackOff`. чешем репу ничего не понятно.
    - удаляем все из namespaces `metallb-system` ставим заново проделываем данные пункты n-1 раз.
    - идем в vm `minikube ssh` проверяем сайт откуда тянутся образы контроллера и спикера `curl quay.io` получаем ответ `Could not resolve host: quay.io`
    - резолвим гугловские днс-ники.
    - пробуем снова поставить.
    - ура!
  - проверяем работоспособность `kubectl --namespace metallb-system get all`
- создаем манифест `metallb-config.yaml`
  ```
   apiVersion: v1
   kind: ConfigMap
   metadata:
     namespace: metallb-system
     name: config
   data:
     config: |
       address-pools:
       - name: default
       protocol: layer2
       addresses:
       - "172.17.255.1-172.17.255.255"
    ```
- применяем манифест `kubectl apply -f metallb-config.yaml`
###### MetalLB | Проверка конфигурации
- делаем копию сервиса `web-svc-cip.yaml` с именем `web-svc-lb.yaml` изменяем имя на `web-svc-lb` и тип на `LoadBalancer` и применяем его.
- смотрим логи пода-контроллера MetalLB  `kubectl --namespace metallb-system logs pod/controller-XXXXXXXX-XXXXXX` пытаемся обратить внимание на назначенный IP-адрес **НО** увы его **НЕТ**.
- может мы его проглядели, ведь в таком длинном логе и правда можно не заметить. Пробуем другой вариант  `kubectl describe svc web-svc-lb`, а нет смотрите не показалось. Сервис выдает ошибку `Failed to allocate IP for "default/web-svc-lb": no available IPs`
- кругом обман.
- пробуем разные вариант манипуляций с ConfigMap. Безрезультатно.
- идем на [оф.сайт Metallb](https://metallb.universe.tf/configuration/#layer-2-configuration) создаем манифест `metallb-addresspool.yaml` по примеру применяем.
- Проверяем `kubectl describe svc web-svc-lb`
   ```
  Name:                     web-svc-lb
   Namespace:                default
   Labels:                   <none>
   Annotations:              metallb.universe.tf/ip-allocated-from-pool: first-pool
   Selector:                 app=web
   Type:                     LoadBalancer
   ...
   Events:
   Type    Reason       Age   From                Message
     ----    ------       ----  ----                -------
   Normal  IPAllocated  19s   metallb-controller  Assigned IP ["172.17.255.2"]
   ```
- пробуем открыть страничку  `curl http://172.17.255.2/index.html` ничего
- добавляем маршрут `ip route add 172.17.255.0/24 via <ip-minikube>` где **<ip-minikube>** ip-адрес виртуалки который можно узнать через `minikube ip`
- теперь можно обращаться по нему curl-ом и каждый раз попадать на другой под
- по умолчанию ipvs использует rr (Round-Robin) балансировку.
###### Задание со звездочкой * | DNS через MetalLB
  - создаем манифест сервиса, который открывает доступ к CoreDNS снаружи кластера `coredns-metallb-svc.yaml` в поддиректории `coredns` применяем.
    - не забываем, что dns работает по TCP и UDP, поэтому сервиса 2 на один и тот же IP c разными типами протокола.
    - для указания конкретного ip для LB. указываем нужный ip через **deprecated** ключ `loadBalancerIP` 
    - проверяем `nslookup coredns-svc-tcp.kube-system.svc.cluster.local 172.17.255.10`
       ```
       Server:		172.17.255.10
       Address:	172.17.255.10#53

       Name:	coredns-svc-tcp.kube-system.svc.cluster.local
       Address: 10.111.224.176
       ```
      
###### Создание Ingress
- прошелся по всем этапам методички для развертывания вручную ingress наткнулся на очередную кучу ошибок, начиная с неполучения ip адреса заканчивая недоступностью 80. плюнул на все удалил
- установил addons `minikube addons enable ingress`
- Создал файл `nginx-lb.yaml` с конфигурацией `LoadBalancer`
- теперь можно делать curl на этот ip
- подключение приложения web к ingress. создаем headless-сервис `web-svc-headless.yaml`
- создаем ingress-прокси, создав манифест с ресурсом ingress `web-ingress.yaml`
  - теперь можно обращаться к странице поду по адресу: `http://<LB_IP>/web/index.html`
  - теперь балансировка выполняется посредством nginx, а не IPVS

###### Задание со звездочкой* | Ingress для Dashboard
1. сделать доступ к kubernetes-dashboard через ingress-прокси сервис должен быть доступен через префикс /dashboard ).
   - устанавливаем дашборд: `kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v3.0.0-alpha0/charts/kubernetes-dashboard.yaml`
   - пишем манифест и деплоим ингресс `ingress-dashboard.yaml` в поддиректории `./dashboard`
- проверяем работоспособность `curl http://192.168.49.2/dashboard`
![ingress-kuber-dashboard.jpg](kubernetes-networks%2Fdashboard%2Fimages%2Fingress-kuber-dashboard.jpg)
###### Задание со звездочкой * | Canary для Ingress
1. Реализуйте канареечное развертывание с помощью ingress-nginx
- Перенаправление части трафика на выделенную группу подов должно происходить по HTTP-заголовку
  - пишем 2 манифеста ingress `ingress-web-*.yaml`, в canary ingress добавляем следующие annotation:
    - `nginx.ingress.kubernetes.io/canary: "true"` - метка, что этот ингресс является канареечным
    - `nginx.ingress.kubernetes.io/canary-weight: "20"` - тут указываем какой процент случайных запросов, будет перенаправлен в службу, указанную в канареечном входе.
    - не забываем, что без доменного имени это работать не будет. в хостах виртуальной машины прописываем имя по которому будем обращаться. `<ip-minikube> ave.canary.host`
- все сохраняем в поддиректории `.canary/`
- проверяем работоспособность 
  - смотрим что поднялись ингрессы![describe.jpg](kubernetes-networks%2Fcanary%2Fimages%2Fdescribe.jpg)
  - несколько раз прогоняем тесты curl-ом `seq 15 | xargs -Iz curl -s http://ave.canary.host/web-canary | grep HOSTNAME` ![canary.jpg](kubernetes-networks%2Fcanary%2Fimages%2Fcanary.jpg)
