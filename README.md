# kernel21_platform
kernel21 Platform repository

**При выполении работы сделано:**

- Изучен процесс устновки minikube (плагины virtualbox, kvm2, none, docker) и kind.
- Выяснены причинины восстановления подов после удаления. Coredns восстанавливается с помощью ReplicationController, остальные являются static pod'ами и восстанавливаются системной службой kublet (конфигуарции находятся в /etc/kubernetes/manifests).
- Собран и запушен на docker hub образ согласно заданию. Создан yaml для pod с init контейнером.
- Получен файл конфигурации hipster shop frontend, в результате аналига логов файл пофикшен и является рабочим.