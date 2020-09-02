# kernel21_platform
kernel21 Platform repository

## Домашняя работа 1 (kubernetes-intro)

**При выполении работы сделано:**

- Изучен процесс устновки minikube (плагины virtualbox, kvm2, none, docker) и kind.
- Выяснены причинины восстановления подов после удаления. Coredns восстанавливается с помощью ReplicationController, остальные являются static pod'ами и восстанавливаются системной службой kublet (конфигуарции находятся в /etc/kubernetes/manifests).
- Собран и запушен на docker hub образ согласно заданию. Создан yaml для pod с init контейнером.
- Получен файл конфигурации hipster shop frontend, в результате аналига логов файл пофикшен и является рабочим.

## Домашняя работа 2 (kubernetes-controllers)

**При выполении работы сделано:**

- Запущен kind
~~~~~~
kind create cluster --config kind-config.yaml
~~~~~~
- Был создан ReplicaSet, Deployment для сервисов frontend, paymentservice.
- **Почему обновление ReplicaSet не повлекло обновление запущенных pod ?**  Replicaset не умеет обновлять template контейнеров.
- С использованием параметров maxSurge и maxUnavailable были реализованы сценарии развертывания blue-green и Reverse Rolling Update.
- Был изучен и проверен механиз корректности работы pod с помощью readinessProbe и livenessProbe.
- Был применен yaml c DaemonSet для node-exporter, при этом он был развернут всех нодах кластера. Было выяснено, что на развертывание на мастер-нодах влияет параметр tolerations.

## Домашняя работа 3 (kubernetes-controllers)

**При выполении работы сделано:**

- Создан Service Account bob, ему дана ему роль admin в рамках всего кластера
- Создать Service Account dave без доступа к кластеру.
- Создан Namespace prometheus
- Создан Service Account carol в namespace prometheus
- Всем Service Account в Namespace prometheus дана возможность делать get, list, watch для pods всего кластера
- Создан Namespace dev
- Создан Service Account jane в Namespace dev
- jane дана роль admin в рамках Namespace dev
- Создан Service Account ken в Namespace dev
- ken дана роль view в рамках Namespace dev

## Домашняя работа 4 (kubernetes-networks)

**При выполении работы сделано:**

- Добавлены проверки Pod (ReadinessProbe, LivenessProbe)
- Создан объект Deployment
- Добавлен сервис в кластер (ClusterIP)
- Включен режим балансировки IPVS
- Установлен MetaILB в Layer2 режиме
- Добавлен сервис LoadBalancer
- Установлен ingress-контроллер и прокси ingress-nginx
- Созданы правила Ingress

#### Вопрос:
***Почему следующая конфигурация валидна, но не имеет смысла? Бывают ли ситуации, когда она все-таки имеет смысл?***
```
                    livenessProbe:
                        exec:
                            command:
                                - 'sh'
                                - '-c'
                                - 'ps aux | grep my_web_server_process'
```
#### Ответ:
В выводе grep будет присутсвовать сама команда grep, ее можно убрать через добавление '| grep -v grep'
Но даже при этом наличие процесса не говорит о его корректной работе. Смысл имеет, если данный процесс сам будет являться результатом предварительной работы при развертывании.

## Домашняя работа 5 (kubernetes-volumes)

**При выполении работы сделано:**

- Развернут StatefulSet c minio в kind
- Создан файл секретов minio-secret.yaml и изменен minio-statefulset.yaml для использования секретов из Secret

## Домашняя работа 6 (kubernetes-templating)

**При выполении работы сделано:**

- Установлен helm 3
- Был установлен Cert-manager
- Установлен Harbor
- Рассмотрен процесс создания своего chart'a
- Рассмотрен helm secret
- Рассмотрен kubecfg
- Рассмотрен kustomize

