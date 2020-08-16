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