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

## Домашняя работа 7 (kubernetes-operators)

- Результат выполнения:

***kubectl get jobs***:

``` 
$ kubectl get jobs

NAME                         COMPLETIONS   DURATION   AGE
backup-mysql-instance-job    1/1           1s         7m4s
restore-mysql-instance-job   1/1           3m22s      8m25s
```

- Результат выполнения:

***$ export MYSQLPOD=$(kubectl get pods -l app=mysql-instance -o jsonpath="
{.items[*].metadata.name}")***

***$ kubectl exec -it $MYSQLPOD -- mysql -potuspassword -e "select * from test;" otus-database***

```
mysql: [Warning] Using a password on the command line interface can be insecure.
+----+-------------+
| id | name        |
+----+-------------+
|  1 | some data   |
|  2 | some data-2 |
+----+-------------+
```

## Домашняя работа 8 (kubernetes-monitoring)

- Создаем новый ns и разворачиваем prometheus-operator через helm3 с values для ingress:
```
kubectl create ns monitoring
helm upgrade --install  prometheus-operator stable/prometheus-operator -n monitoring -f prometheus-operator.values.yaml
```
- Ждем старта всех pods
```
kubectl get pods -n monitoring
NAME                                                      READY   STATUS    RESTARTS   AGE
alertmanager-prometheus-operator-alertmanager-0           2/2     Running   0          45s
prometheus-operator-grafana-7769fc4f77-lq28g              2/2     Running   0          72s
prometheus-operator-kube-state-metrics-69fcc8d48c-9gvmd   1/1     Running   0          72s
prometheus-operator-operator-5cf9888d76-szdxf             2/2     Running   0          72s
prometheus-operator-prometheus-node-exporter-s2j65        1/1     Running   0          72s
prometheus-prometheus-operator-prometheus-0               3/3     Running   1          35s

```
- Разворачиваем:
1. ConfigMap с конфигурацией nginx.
2. Deployment с nginx и sidecar nginx-exporter контейнерами:
  * nginx с 80 портом с поддержкой nginx-статуса доступного на порту 8080 по адресу http://127.0.0.1:8080/basic_status
  * nginx-exporter, отдающий метрики в формате prometheus на порту 9113
  * три реплики
3. Service для доступа к pods.
4. CR ServiceMonitor для мониторинга сервисов.

```
kubectl apply -f nginx-configMap.yaml
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-nginx-service.yaml
kubectl apply -f nginx-serviceMonitor.yaml
```
- Получаем пароль для grafana
```
kubectl get secret -n monitoring prometheus-operator-grafana -o yaml
apiVersion: v1
data:
  admin-password: cHJvbS1vcGVyYXRvcg==
  admin-user: YWRtaW4=
  ldap-toml: ""
----
```
```
$echo "YWRtaW4=" | base64 -d
admin
$echo "cHJvbS1vcGVyYXRvcg==" | base64 -d
prom-operator
```

- Заходим на http://prometheus.lan/targets
![](img/kubernetes-monitoring-prom.png)
- Заходим на http://grafana.lan, импортируем dashboard.json для nginx prometheus exporter
![](img/kubernetes-monitoring-grafana.png)

## Домашняя работа 9 (kubernetes-logging)

**При выполении работы сделано:**

- Был установилен EFK
- Провели наблюдение на отработку дашборды при отказе ноды с ElasticSearch
- Была создана дашборда в Kibana
- Была создана дашборда с Loki в garafana.

## Домашняя работа 10 (kubernetes-gitops)

**При выполении работы сделано:**

- Был установлен Flux, Flagger
- Посмотрели обновление манифестов при разверывании нового образа docker
- Провели канареечное развертывание с помощью Flagger

Репозиторий: <https://gitlab.com/kernel218/microservices-demo>

## Домашняя работа 11 (kubernetes-vault)

**При выполении работы сделано:**

- Был установлен Vault
- Сгенерировали сертификаты
- Настроили работу vault по HTTPS

```console
$ helm status vault
WARNING: "kubernetes-charts.storage.googleapis.com" is deprecated for "stable" and will be deleted Nov. 13, 2020.
WARNING: You should switch to "https://charts.helm.sh/stable"
NAME: vault
LAST DEPLOYED: Tue Nov 10 15:24:05 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Thank you for installing HashiCorp Vault!

Now that you have deployed Vault, you should look over the docs on using
Vault with Kubernetes available here:

https://www.vaultproject.io/docs/


Your release is named vault. To learn more about the release, try:

  $ helm status vault
  $ helm get manifest vault
```

```console
$ kubectl exec -it vault-0 -- vault operator init --key-shares=1 --key-threshold=1
Unseal Key 1: 9JWOacoVgI9dwG12YIH9U7EeHVlE5nZTFAxt7w5qzNY=

Initial Root Token: s.ACIsIsQFVc3fB28VQfYJ4pph
```

```console
$ kubectl exec -it vault-0 -- vault status
Key             Value
---             -----
Seal Type       shamir
Initialized     true
Sealed          false
Total Shares    1
Threshold       1
Version         1.5.4
Cluster Name    vault-cluster-40d72e93
Cluster ID      728cc4ca-d52b-6628-96fe-020efcda1777
HA Enabled      true
HA Cluster      https://vault-0.vault-internal:8201
HA Mode         active
$ kubectl exec -it vault-1 -- vault status
Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           1
Threshold              1
Version                1.5.4
Cluster Name           vault-cluster-40d72e93
Cluster ID             728cc4ca-d52b-6628-96fe-020efcda1777
HA Enabled             true
HA Cluster             https://vault-0.vault-internal:8201
HA Mode                standby
Active Node Address    http://10.44.2.6:8200
$ kubectl exec -it vault-2 -- vault status
Key                    Value
---                    -----
Seal Type              shamir
Initialized            true
Sealed                 false
Total Shares           1
Threshold              1
Version                1.5.4
Cluster Name           vault-cluster-40d72e93
Cluster ID             728cc4ca-d52b-6628-96fe-020efcda1777
HA Enabled             true
HA Cluster             https://vault-0.vault-internal:8201
HA Mode                standby
Active Node Address    http://10.44.2.6:8200
```

```console
$ kubectl exec -it vault-0 -- vault login
Token (will be hidden): 
Success! You are now authenticated. The token information displayed below
is already stored in the token helper. You do NOT need to run "vault login"
again. Future Vault requests will automatically use this token.

Key                  Value
---                  -----
token                s.ACIsIsQFVc3fB28VQfYJ4pph
token_accessor       F3rNMqLt2J08EWG8JyUMMd04
token_duration       ∞
token_renewable      false
token_policies       ["root"]
identity_policies    []
policies             ["root"]
$ kubectl exec -it vault-0 -- vault auth list
Path      Type     Accessor               Description
----      ----     --------               -----------
token/    token    auth_token_ddbd8546    token based credentials
```

```console
$ kubectl exec -it vault-0 -- vault read otus/otus-ro/config
Key                 Value
---                 -----
refresh_interval    768h
password            asajkjkahs
username            otus
$ kubectl exec -it vault-0 -- vault kv get otus/otus-rw/config
====== Data ======
Key         Value
---         -----
password    asajkjkahs
username    otus
```

```console
$ kubectl exec -it vault-0 -- vault auth list
Path           Type          Accessor                    Description
----           ----          --------                    -----------
kubernetes/    kubernetes    auth_kubernetes_aff4ece0    n/a
token/         token         auth_token_ddbd8546         token based credentials
```

#### Вопрос:
***Вопрос: что делает эта конструкция sed ’s/\x1b[[0-9;]m//g’ ?***

#### Ответ:
убирает цвет в выводе консоли(убирает escape-последовательность)

#### Вопрос:
***Почему мы смогли записать otus-rw/config1 но не смогли otus-rw/config ?***

#### Ответ:
Потому что в политиках определены правила

```
path "otus/otus-ro/*" {
  capabilities = ["read", "list"]
}
path "otus/otus-rw/*" {
  capabilities = ["read", "create", "list"]
}
```
*Поправим правила:*

```
path "otus/otus-ro/*" {
  capabilities = ["read", "list"]
}
path "otus/otus-rw/*" {
  capabilities = ["read", "create", "update", "list"]
}
```

```console
$ kubectl exec -ti vault-agent-example -c nginx-container  -- cat /usr/share/nginx/html/index.html
<html>
<body>
<p>Some secrets:</p>
<ul>
<li><pre>username: otus</pre></li>
<li><pre>password: asajkjkahs</pre></li>
</ul>

</body>
</html>
```

``` console
$ kubectl exec -it vault-0 -- vault write pki_int/roles/devlab-dot-ru allowed_domains="devlab.ru" allow_subdomains=true max_ttl="720h"
Success! Data written to: pki_int/roles/devlab-dot-ru
$ kubectl exec -it vault-0 -- vault write pki_int/issue/devlab-dot-ru common_name="gitlab.devlab.ru" ttl="24h" 
Key                 Value
---                 -----
ca_chain            [-----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUHVVm+JS5vk31bi8J+pIaiVjjZYQwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhhbXBsZS5ydTAeFw0yMDExMTAxMzQzNThaFw0yNTEx
MDkxMzQ0MjhaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAJb/cl4oB1q9
C3IxC0ITM2uxuVL9RWamyeF9NWkDMUixKATNVoXO5Fm3Nv0aJE+WzNz5sBpURFPs
ocqoi3vlFiT++CvA1nxwG7c3MhfdyzymAIeoWxRNztzRCbCNcLcSAiEp99z9BqBg
qK2tOWZTyTrEGxPrRNpmiHDUGnj+eGQSjIbCu4s8FacOIwgQrOlhFbTvhFyS/dPD
OK79oU12xM/bmrYKRsTX069nLcX0VSjiq49ZIK1CPR+99qkYaQpPXNd0dSDUYApf
GB107PS6ThVPSQu1KfgnPAJluu+EJa0cp37zZZKNCa2rbVhLMFKRORlLy0q3l58g
L6lPPX9D0IMCAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUVP8BGgPlrbzRzlUadcMp2IgE7AIwHwYDVR0jBBgwFoAU
AruKZQIV4dQV97zD8zOzjohmhKwwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
JpAAcaGKm6N4ryv0bvbCtIf46DaSWhCbCLQzvmHZMfvn4fvP2jAjknRznKYlFte/
pZvnqJyOnWi8pRRV5FAQ8nhr8MzFPwsxjtaD6x2A0qxA7S1cctUzO4xxCHWB/hGQ
Dx5H+b70f6nPWtUI1CiUHVNTatwutTCaGpdre+lh+zGYVeZbt9Za4HAc19vTZlhX
1Dzsbu3LZhLu+l55Wa3eFTb6cUnkp+i08AYdhN/+XyU8d+sKphmLB3AUzOH4R/bh
3EUQNzF2cx8D8eXRWh9wvc+mjoOzdR5/ms5QIz0y9lVRApvP/YYs9lgN2hP0Tvch
th6ktjdHuU076s3rV1kxvg==
-----END CERTIFICATE-----]
certificate         -----BEGIN CERTIFICATE-----
MIIDZTCCAk2gAwIBAgIUOQOWN+dkPa4qn+napZy7R/BGTvgwDQYJKoZIhvcNAQEL
BQAwLDEqMCgGA1UEAxMhZXhhbXBsZS5ydSBJbnRlcm1lZGlhdGUgQXV0aG9yaXR5
MB4XDTIwMTExMDEzNTQwMVoXDTIwMTExMTEzNTQzMFowGzEZMBcGA1UEAxMQZ2l0
bGFiLmRldmxhYi5ydTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAPET
CQ/gjpJ1m64OOrv9qBP671IH3dvVU1kzaXqX/hi2QUA9jwx92AdRxqHw+G6NxSng
+3lusfW5R9p4yYPNzhF8RJ5EkWM3qiyDCH0ps6vLFbN71Qb748y8FjQQFRuvHUv/
Y9QvYEjRsTTBJyDLGjxjSLnTJELX1Q3DoghvSHMm1ObBs6FGn5jrMzVm+XmkIRr5
JDF1R3EJWbufUQfafivAEatZ2E8by6kvMgZN/Mudy1l96ijl8AlsBgSmzrSArhDW
A1yW+KGhXOg/82JdFDEzHFAnpfbZ2ePbuMBy3j/NbsJ3vSvNJ1sYMM1rWJ3zHpPS
MUApIC16fzAVZ2H/xbMCAwEAAaOBjzCBjDAOBgNVHQ8BAf8EBAMCA6gwHQYDVR0l
BBYwFAYIKwYBBQUHAwEGCCsGAQUFBwMCMB0GA1UdDgQWBBQ8BWH1n3yc0l/JQny+
JgC4Pt7U5zAfBgNVHSMEGDAWgBRU/wEaA+WtvNHOVRp1wynYiATsAjAbBgNVHREE
FDASghBnaXRsYWIuZGV2bGFiLnJ1MA0GCSqGSIb3DQEBCwUAA4IBAQBgmgqpXSKO
/RhzUEpIclcuYqyx/d1dzuLD4Qp/1nvH1QemBpvtM4qOwI7wc5V1RwGYIZNiOatS
CkuWZ6ePsxVrLScbiBwbHmeHpS5KEzwnZXmdHhnp1scuv3cTdPHj0ZWgjs2qyTz6
BDqgZYYO85Ioq1N2TA2c2T43KULDa4Gml46Bmxy7dKDeN29ISl7BAItDCChUTtsR
DWIlw4cKJG3DCdwA7eqvN9prt5es4/JmlhjLLT6d6Hqe78hoek5VKhsu0xPa/CEo
9EJL2d33j3uSyGXFMJiEF7bBx65AMg944x/ojyqkiCHp1I6rZtIG/0kwIVobUgMu
li/XZCZ745WR
-----END CERTIFICATE-----
expiration          1605102870
issuing_ca          -----BEGIN CERTIFICATE-----
MIIDnDCCAoSgAwIBAgIUHVVm+JS5vk31bi8J+pIaiVjjZYQwDQYJKoZIhvcNAQEL
BQAwFTETMBEGA1UEAxMKZXhhbXBsZS5ydTAeFw0yMDExMTAxMzQzNThaFw0yNTEx
MDkxMzQ0MjhaMCwxKjAoBgNVBAMTIWV4YW1wbGUucnUgSW50ZXJtZWRpYXRlIEF1
dGhvcml0eTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBAJb/cl4oB1q9
C3IxC0ITM2uxuVL9RWamyeF9NWkDMUixKATNVoXO5Fm3Nv0aJE+WzNz5sBpURFPs
ocqoi3vlFiT++CvA1nxwG7c3MhfdyzymAIeoWxRNztzRCbCNcLcSAiEp99z9BqBg
qK2tOWZTyTrEGxPrRNpmiHDUGnj+eGQSjIbCu4s8FacOIwgQrOlhFbTvhFyS/dPD
OK79oU12xM/bmrYKRsTX069nLcX0VSjiq49ZIK1CPR+99qkYaQpPXNd0dSDUYApf
GB107PS6ThVPSQu1KfgnPAJluu+EJa0cp37zZZKNCa2rbVhLMFKRORlLy0q3l58g
L6lPPX9D0IMCAwEAAaOBzDCByTAOBgNVHQ8BAf8EBAMCAQYwDwYDVR0TAQH/BAUw
AwEB/zAdBgNVHQ4EFgQUVP8BGgPlrbzRzlUadcMp2IgE7AIwHwYDVR0jBBgwFoAU
AruKZQIV4dQV97zD8zOzjohmhKwwNwYIKwYBBQUHAQEEKzApMCcGCCsGAQUFBzAC
hhtodHRwOi8vdmF1bHQ6ODIwMC92MS9wa2kvY2EwLQYDVR0fBCYwJDAioCCgHoYc
aHR0cDovL3ZhdWx0OjgyMDAvdjEvcGtpL2NybDANBgkqhkiG9w0BAQsFAAOCAQEA
JpAAcaGKm6N4ryv0bvbCtIf46DaSWhCbCLQzvmHZMfvn4fvP2jAjknRznKYlFte/
pZvnqJyOnWi8pRRV5FAQ8nhr8MzFPwsxjtaD6x2A0qxA7S1cctUzO4xxCHWB/hGQ
Dx5H+b70f6nPWtUI1CiUHVNTatwutTCaGpdre+lh+zGYVeZbt9Za4HAc19vTZlhX
1Dzsbu3LZhLu+l55Wa3eFTb6cUnkp+i08AYdhN/+XyU8d+sKphmLB3AUzOH4R/bh
3EUQNzF2cx8D8eXRWh9wvc+mjoOzdR5/ms5QIz0y9lVRApvP/YYs9lgN2hP0Tvch
th6ktjdHuU076s3rV1kxvg==
-----END CERTIFICATE-----
private_key         -----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEA8RMJD+COknWbrg46u/2oE/rvUgfd29VTWTNpepf+GLZBQD2P
DH3YB1HGofD4bo3FKeD7eW6x9blH2njJg83OEXxEnkSRYzeqLIMIfSmzq8sVs3vV
BvvjzLwWNBAVG68dS/9j1C9gSNGxNMEnIMsaPGNIudMkQtfVDcOiCG9IcybU5sGz
oUafmOszNWb5eaQhGvkkMXVHcQlZu59RB9p+K8ARq1nYTxvLqS8yBk38y53LWX3q
KOXwCWwGBKbOtICuENYDXJb4oaFc6D/zYl0UMTMcUCel9tnZ49u4wHLeP81uwne9
K80nWxgwzWtYnfMek9IxQCkgLXp/MBVnYf/FswIDAQABAoIBAQCLd+LHP7fb/ZRq
dyr9tXs2y/cGsyxkUR9ePMMqPKKxc0d+vd5zcJ65ZVMQP1PKydQmLVXvY94q9d0f
BMA4s6kjLoyYL70Y9IxMIiaYGrcqjVxpsRuGZdXdjXce+arskDvXytHbYOlIV6A4
kAJuE3KDO0FI2GFjFnDY/LRSQudcThxlRfStXd6H6veY0Z9l/PrMrw50pPBq4qBE
Tg0LEPwhj30N6/V8ZMnK0bCRwLiNAQqYLLqKJ4t2EID7L7HGwYUGTw2F7soELrhS
a2g2BaxgC2J9UafDVteAqyYRQa3uZ6/zB1C2ijKxtLgwshAKOIBejYHHwFqRsaDw
dZ34BO65AoGBAPS8GFxyNhT3nP2PYhZVvj3FKAwAIV56qY34WZ3yHa1sSBsCb8UB
XwhRn/s7ZbAzo9nlOmr1O77ybejxRUmGs6e+E4j1u8dUlSTc33BTd979XGt/UXAz
JbkyKZNQc17evKS5EDUC/gFWCxcOsnX12LjrYBgf9fJcuGiFBpKbVyUfAoGBAPwr
zpPviCjgeudqSHx8fNswGfQsTvQjV4RchpjOOIQ3uuXs/MTPVs6zklnfavlrFd18
uL8QAB6ef2ZGkBhsGxeYjdPdeqIjwS5f7TtSl1QFMOnPmaT+/QXU2U8umbQ4JQW7
i8O53loxUiWLa1IhHVSI4ZWCJIngvtRwEbYauJjtAoGBAMcnZZ+dJVtsoGlKS+Sn
A7faf5s8Y+sxYFbyeWLpirL8gbTRB8lGM2JeohRcooR/kV+YhTBSvbrGJyC/bcXG
gt4G9Hiol5U+xFuKDZ2nns1sWc/0fH4UcSdCpciGWEwkb1iQbJrnA3Js5Xtu71TE
qgbZK4qWP5tpTntnfRDCrmi7AoGAGRJx85t5Ojc3gRK8KkRmVZSuv+w33WY2KV7Z
sw+t5tdzqbCqYRcMVnjcMDtac3oGLoNcCwMYP/MaT5zsbsEw4GO2lj4LF1vetTGs
cJ2BlkT93AFcEV+Y4J+NC6ZiedyrMaq39rngNa95r2nxPbU1KVaCt069O0gxMQYD
fMujVvECgYEAwAtEPoSlr9irA1QMiQokP77V1xxYDOh0xUPtpxgx9oBzLaKIkU8B
Tg0JP0JYPB7fEfDHVQs+QeCrpehZChpX54zg3dXxje0qfBDtE6ir/GoWUxhS23Af
kRn2xO27PxsoN7VFlPNnUByxY5AC3sI6mngdfFYhtUYDFss69s7aclA=
-----END RSA PRIVATE KEY-----
private_key_type    rsa
serial_number       39:03:96:37:e7:64:3d:ae:2a:9f:e9:da:a5:9c:bb:47:f0:46:4e:f8
```

## Домашняя работа 12 (kubernetes-storage)

- Подготовим кластер k1s из 2 нод:
```console
git clone https://github.com/maniaque/k1s.git
cd k1s/vagrant/twin/
vagrant up
vagrant ssh -c 'cat /home/vagrant/.kube/config' > ~/.kube/config
```
- Установим CSI Host Path Driver:
```console
git clone https://github.com/kubernetes-csi/csi-driver-host-path.git
./csi-crd_deploy.sh
./csi-driver-host-path/deploy/kubernetes-1.18/deploy.sh
```
- Применим манифесты
```console
$ kubectl apply -f ./hw/csi-storageclass.yaml -f ./hw/csi-pvc.yaml -f ./hw/csi-app.yaml
storageclass.storage.k8s.io/csi-hostpath-sc created
persistentvolumeclaim/storage-pvc created
pod/storage-pod created
```
- Запишем данные в volume
```console
$ kubectl exec -it storage-pod /bin/sh
# touch /data/hello-world    
# ls -la /data
total 8
drwxr-xr-x 2 root root 4096 Nov 11 15:19 .
drwxr-xr-x 1 root root 4096 Nov 11 15:17 ..
-rw-r--r-- 1 root root    0 Nov 11 15:19 hello-world
```
- Создадим снапшот
```console
kubectl apply -f hw/csi-snapshot.yaml
volumesnapshot.snapshot.storage.k8s.io/csi-snapshot created
```
- Удалим pod, pv, pvс
```console
kubectl delete -f ./hw/csi-app.yaml
pod "storage-pod" deleted
kubectl delete -f ./hw/csi-pvc.yaml
persistentvolumeclaim "storage-pvc" deleted

kubectl get pvc
No resources found in default namespace.
kubectl get pv
No resources found in default namespace.
```

- Восстановимся из snapshot'а
```
kubectl apply -f ./hw/csi-restore.yaml
```
- Восстанавливаем pod и проверяем данные
```
kubectl apply -f ./hw/csi-app.yaml
```
```console
$ kubectl exec storage-pod -- ls -la /data
total 8
drwxr-xr-x 2 root root 4096 Nov 11 15:19 .
drwxr-xr-x 1 root root 4096 Nov 11 15:17 ..
-rw-r--r-- 1 root root    0 Nov 11 15:19 hello-world
```

## Домашняя работа 13 (kubernetes-debug)


### 1. kubectl debug

- Установим kubectl debug согласно инструкции из репозитария проекта, поправим agent_daemonset.yml, чтобы избавится от ошибки:

```console
$ kubectl apply -f strace/agent_daemonset.yml
error: unable to recognize "strace/agent_daemonset.yml": no matches for kind "DaemonSet" in version "extensions/v1beta1"
```
- Запустим pod

```console
kubectl apply -f strace/nginx.yaml
pod/nginx created
```

- Запустим дебаг и видим отсутсвие прав:

```console
$ kubectl-debug nginx --agentless=false --port-forward=true
bash-5.0# ps
PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;
   29 101       0:00 nginx: worker process
   39 root      0:00 bash
   45 root      0:00 ps
bash-5.0# strace -p29 -c
strace: attach: ptrace(PTRACE_SEIZE, 29): Operation not permitted
```

- Заходим на ноду и смотрим docker capabilities:

```console
$ docker inspect 0b9f3610a18e | grep CapAdd
"CapAdd": null
```
- Испраляем в файле версию образа debug-agent с 0.0.1 на latest, удаляем старый daemonset, устанавливаем новый и проверяем права:

```console
$ docker inspect 008a6212c55a | less
"CapAdd": [
    "SYS_PTRACE",    
    "SYS_ADMIN"
],
```
- Запускаем дебаг и видим нормальную работу strace

$ kubectl-debug nginx --agentless=false --port-forward=true
bash-5.0# ps
PID   USER     TIME  COMMAND
    1 root      0:00 nginx: master process nginx -g daemon off;
   29 101       0:00 nginx: worker process
   57 root      0:00 bash
   63 root      0:00 ps
bash-5.0# strace -p29 -c
strace: Process 29 attached


### 2. iptables-tailer

- Установим netperf-operator:

```console
$ kubectl apply -f ./deploy/crd.yaml
$ kubectl apply -f ./deploy/rbac.yaml
$ kubectl apply -f ./deploy/operator.yaml
```

- Запустим пример:
```console
$ kubectl apply -f ./deploy/cr.yaml
$ kubectl describe netperf.app.example.com/example

- Установим логирования iptables:
```console
$ kubectl apply -f kit/netperf-calico-policy.yaml
$ kubectl delete -f ../deploy/cr.yaml
$ kubectl apply -f ../deploy/cr.yaml
$ kubectl describe netperf.app.example.com/example
...
Status:
  Client Pod:          netperf-client-4bdd99b825f8
  Server Pod:          netperf-server-4bdd99b825f8
  Speed Bits Per Sec:  0
  Status:              Started test
...
```
- Подключаемся к ноде и смотрим логи:

```console
$ kubectl logs pod/netperf-operator-55b49546b5-fxjp6
....
time="2020-11-12T12:07:10Z" level=debug msg="New Netperf event, name: example, deleted: false, status: Done"
time="2020-11-12T12:07:10Z" level=debug msg="Nothing needed to do for update event on Netperf example in state Done"
time="2020-11-12T12:07:15Z" level=debug msg="New Netperf event, name: example, deleted: false, status: Done"
time="2020-11-12T12:07:15Z" level=debug msg="Nothing needed to do for update event on Netperf example in state Done"
time="2020-11-12T12:07:20Z" level=debug msg="New Netperf event, name: example, deleted: false, status: Done"
time="2020-11-12T12:07:20Z" level=debug msg="Nothing needed to do for update event on Netperf example in state Done"
time="2020-11-12T12:07:25Z" level=debug msg="New Netperf event, name: example, deleted: false, status: Done"
time="2020-11-12T12:07:25Z" level=debug msg="Nothing needed to do for update event on Netperf example in state Done"
time="2020-11-12T12:07:30Z" level=debug msg="New Netperf event, name: example, deleted: false, status: Done"
time="2020-11-12T12:07:30Z" level=debug msg="Nothing needed to do for update event on Netperf example in state Done"
```
```console
root@k1s:~# sudo iptables --list -nv | grep DROP - счетчики дропов 
root@k1s:~# sudo iptables --list -nv | grep LOG
root@k1s:~# journalctl -k | grep calico
```

### iptailer

- Прменяем манифест:
```console
$ kubectl apply -f kit/iptables-tailer.yaml 
$ kubectl describe daemonset kube-iptables-tailer -n kube-system
Events:
  Type     Reason        Age                From                  Message
  ----     ------        ----               ----                  -------
  Warning  FailedCreate  10m  daemonset-controller  Error creating: pods "kube-iptables-tailer-" is forbidden: error looking up service account kube-system/kube-iptables-tailer: serviceaccount "kube-iptables-tailer" not found
```
- Прменяем ServiceAccount
```console
$ kubectl apply -f kit/kit-serviceaccount.yaml
$ kubectl apply -f kit/kit-clusterrole.yaml
$ kubectl apply -f kit/kit-clusterrolebinding.yaml 
$ kubectl describe daemonset kube-iptables-tailer -n kube-system
Events:
  Type     Reason            Age                   From                  Message
  ----     ------            ----                  ----                  -------
  ...
  Normal   SuccessfulCreate  25s                   daemonset-controller  Created pod: kube-iptables-tailer-jd65k
  Normal   SuccessfulCreate  25s                   daemonset-controller  Created pod: kube-iptables-tailer-r9vs5
```
- Пересоздаём netperf

```console
$ kubectl delete -f ../deploy/cr.yaml
$ kubectl apply -f ../deploy/cr.yaml
$ kubectl describe netperf.app.example.com/example
Status:
  Client Pod:          netperf-client-4bdd99b825f8
  Server Pod:          netperf-server-4bdd99b825f8
  Speed Bits Per Sec:  4119.22
  Status:              Done
```
- Проверяем:

```console
kubectl describe pod/netperf-client-4bdd99b825f8
Name:         netperf-client-4bdd99b825f8
Namespace:    default
Priority:     0
Node:         k1s/192.168.33.110
Start Time:   Thu, 12 Nov 2020 17:00:03 +0300
Labels:       app=netperf-operator
              netperf-type=client
Annotations:  cni.projectcalico.org/podIP: 10.244.45.220/32
              cni.projectcalico.org/podIPs: 10.244.45.220/32
Status:       Running
IP:           10.244.45.220
IPs:
  IP:           10.244.45.220
Controlled By:  Netperf/example
Containers:
  netperf-client-4bdd99b825f8:
    Container ID:  docker://5a00317f4d95fe9a39875e01d0d398d4d80d0a7c48291dec967cbf0035cfa4aa
    Image:         tailoredcloud/netperf:v2.7
    Image ID:      docker-pullable://tailoredcloud/netperf@sha256:0361f1254cfea87ff17fc1bd8eda95f939f99429856f766db3340c8cdfed1cf1
    Port:          <none>
    Host Port:     <none>
    Command:
      netperf
      -H
      10.244.45.219
    State:          Running
      Started:      Thu, 12 Nov 2020 17:55:12 +0300
    Last State:     Terminated
      Reason:       Error
      Exit Code:    255
      Started:      Thu, 12 Nov 2020 17:47:53 +0300
      Finished:     Thu, 12 Nov 2020 17:50:03 +0300
    Ready:          True
    Restart Count:  11
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-bl5kq (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-bl5kq:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-bl5kq
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason      Age                    From                  Message
  ----     ------      ----                   ----                  -------
  Normal   Scheduled   57m                    default-scheduler     Successfully assigned default/netperf-client-4bdd99b825f8 to k1s
  Normal   Created     47m (x5 over 57m)      kubelet               Created container netperf-client-4bdd99b825f8
  Normal   Started     47m (x5 over 57m)      kubelet               Started container netperf-client-4bdd99b825f8
  Warning  BackOff     6m58s (x123 over 52m)  kubelet               Back-off restarting failed container
  Normal   Pulled      2m5s (x12 over 57m)    kubelet               Container image "tailoredcloud/netperf:v2.7" already present on machine
  Warning  PacketDrop  2m5s (x11 over 55m)    kube-iptables-tailer  Packet dropped when sending traffic to netperf-server-4bdd99b825f8 (10.244.45.219)
```

## Домашняя работа 14 (kubernetes-production-clusters)

### Разворачивание и обновление кластера k8s (kubeadm)

- Создаем 4 ноды с ubuntu 18.04 LTS:

```console
$ gcloud compute instances create master --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --machine-type=n1-standard-2
Created [https://www.googleapis.com/compute/v1/projects/project-otus-287811/zones/europe-west4-c/instances/master].
NAME    ZONE            MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP  STATUS
master  europe-west4-c  n1-standard-2               10.164.0.63  34.90.85.26  RUNNING
$ gcloud compute instances create node-1 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --machine-type=n1-standard-1 
Created [https://www.googleapis.com/compute/v1/projects/project-otus-287811/zones/europe-west4-c/instances/node-1].
NAME    ZONE            MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP    EXTERNAL_IP   STATUS
node-1  europe-west4-c  n1-standard-1               10.164.15.192  34.90.236.93  RUNNING
$ Createdgcloud compute instances create node-2 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --machine-type=n1-standard-1
Createdgcloud: команда не найдена
$ gcloud compute instances create node-2 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --machine-type=n1-standard-1
Created [https://www.googleapis.com/compute/v1/projects/project-otus-287811/zones/europe-west4-c/instances/node-2].
NAME    ZONE            MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP    EXTERNAL_IP   STATUS
node-2  europe-west4-c  n1-standard-1               10.164.15.193  34.90.214.66  RUNNING
$ gcloud compute instances create node-3 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --machine-type=n1-standard-1
Created [https://www.googleapis.com/compute/v1/projects/project-otus-287811/zones/europe-west4-c/instances/node-3].
NAME    ZONE            MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP    EXTERNAL_IP   STATUS
node-3  europe-west4-c  n1-standard-1               10.164.15.194  34.90.18.167  RUNNING
```

- Отключаем swap

```console
$ gcloud beta compute ssh master
$ sudo swapoff -a
$ gcloud beta compute ssh node-1
$ sudo swapoff -a
$ gcloud beta compute ssh node-2
$ sudo swapoff -a
$ gcloud beta compute ssh node-3
$ sudo swapoff -a
```

- Включаем маршрутизацию пакетов на всех нодах и применяем её.

```console
# cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
# sysctl --system
```

- Добавим ключ официального репозитария docker, сам репозитарий и установим из него необходимые пакеты

```console
# apt update && apt-get install -y \
    apt-transport-https ca-certificates curl software-properties-common gnupg2
# curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - 
# add-apt-repository \
            "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
# apt update && apt-get install -y \
          containerd.io=1.2.13-1 \
          docker-ce=5:19.03.8~3-0~ubuntu-$(lsb_release -cs) \
          docker-ce-cli=5:19.03.8~3-0~ubuntu-$(lsb_release -cs)
# cat > /etc/docker/daemon.json <<EOF
{
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m"
     },
    "storage-driver": "overlay2"
}
EOF
# mkdir -p /etc/systemd/system/docker.service.d && systemctl daemon-reload && systemctl restart docker
```

- Установим kubeadm, kubectl, kubelet на всех нодах

```console
# apt update && apt install -y apt-transport-https curl
# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
# cat <<EOF > /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
# apt update && apt install -y kubelet=1.17.4-00 kubeadm=1.17.4-00 kubectl=1.17.4-00
```

- Создаем кластер

```console
# kubeadm init --pod-network-cidr=192.168.0.0/24
...
hen you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.164.0.63:6443 --token 3zca56.pkgnfs2f5ddnu8vt \
    --discovery-token-ca-cert-hash sha256:c2658f8d4f01bf2ceed86bc067fcf7ba57e3ec819596e27abadb55d70b940aac
```

- Копируем конфиг kubectl и проверяем

```console
# mkdir -p $HOME/.kube
# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# sudo chown $(id -u):$(id -g) $HOME/.kube/config
# kubectl get nodes 
NAME     STATUS     ROLES    AGE     VERSION
master   NotReady   master   4m15s   v1.17.4
```

- Устанавливаем сетевой плагин calico

```console
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

- Подключаем worker-ноды

```console
# kubeadm token list
TOKEN                     TTL         EXPIRES                USAGES                   DESCRIPTION                                                EXTRA GROUPS
3zca56.pkgnfs2f5ddnu8vt   23h         2020-11-14T10:09:49Z   authentication,signing   The default bootstrap token generated by 'kubeadm init'.   system:bootstrappers:kubeadm:default-node-token

# openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \
  openssl dgst -sha256 -hex | sed 's/^.* //'
c2658f8d4f01bf2ceed86bc067fcf7ba57e3ec819596e27abadb55d70b940aac
# kubeadm join 10.164.0.63:6443 --token 3zca56.pkgnfs2f5ddnu8vt \
    --discovery-token-ca-cert-hash sha256:c2658f8d4f01bf2ceed86bc067fcf7ba57e3ec819596e27abadb55d70b940aac
# kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   13m   v1.17.4
node-1   Ready    <none>   79s   v1.17.4
node-2   Ready    <none>   72s   v1.17.4
node-3   Ready    <none>   69s   v1.17.4
```

- Для демонстрации работы кластера запустим nginx

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 4
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.17.2
        ports:
        - containerPort: 80
```
```console
# kubectl apply -f deployment.yaml
# kubectl get pod -o wide
NAME                               READY   STATUS    RESTARTS   AGE   IP              NODE     NOMINATED NODE   READINESS GATES
nginx-deployment-c8fd555cc-6w6fm   1/1     Running   0          17s   192.168.0.65    node-2   <none>           <none>
nginx-deployment-c8fd555cc-jfbpl   1/1     Running   0          17s   192.168.0.193   node-3   <none>           <none>
nginx-deployment-c8fd555cc-lbd2b   1/1     Running   0          17s   192.168.0.66    node-2   <none>           <none>
nginx-deployment-c8fd555cc-wb59m   1/1     Running   0          17s   192.168.0.129   node-1   <none>           <none>
```

- Обновляем master-ноду до версии 1.18, анализируем версии.

```console
# apt update && apt-get install -y kubeadm=1.18.0-00 \
kubelet=1.18.0-00 kubectl=1.18.0-00
# kubectl get pod -o wide
NAME                               READY   STATUS    RESTARTS   AGE     IP              NODE     NOMINATED NODE   READINESS GATES
nginx-deployment-c8fd555cc-6w6fm   1/1     Running   0          3m43s   192.168.0.65    node-2   <none>           <none>
nginx-deployment-c8fd555cc-jfbpl   1/1     Running   0          3m43s   192.168.0.193   node-3   <none>           <none>
nginx-deployment-c8fd555cc-lbd2b   1/1     Running   0          3m43s   192.168.0.66    node-2   <none>           <none>
nginx-deployment-c8fd555cc-wb59m   1/1     Running   0          3m43s   192.168.0.129   node-1   <none>           <none>

# kubeadm version
kubeadm version: &version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:56:30Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
# kubelet --version
Kubernetes v1.18.0
# kubectl version
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:58:59Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.14", GitCommit:"f238f5142728be4033c37aa0ad69bf806090beae", GitTreeState:"clean", BuildDate:"2020-11-11T13:03:54Z", GoVersion:"go1.13.15", Compiler:"gc", Platform:"linux/amd64"}
# kubectl describe pod kube-apiserver-master -n kube-system
Name:                 kube-apiserver-master
Namespace:            kube-system
Priority:             2000000000
Priority Class Name:  system-cluster-critical
Node:                 master/10.164.0.63
Start Time:           Fri, 13 Nov 2020 10:09:50 +0000
Labels:               component=kube-apiserver
                      tier=control-plane
Annotations:          kubernetes.io/config.hash: 9cac50f26a161064521996c064f0e92a
                      kubernetes.io/config.mirror: 9cac50f26a161064521996c064f0e92a
                      kubernetes.io/config.seen: 2020-11-13T10:30:17.743884453Z
                      kubernetes.io/config.source: file
Status:               Running
IP:                   10.164.0.63
IPs:
  IP:           10.164.0.63
Controlled By:  Node/master
Containers:
  kube-apiserver:
    Container ID:  docker://728288b5692e9a164b661bb85abdd7b902758fa260dfa33e280ddde083570a68
    Image:         k8s.gcr.io/kube-apiserver:v1.17.14
    Image ID:      docker-pullable://k8s.gcr.io/kube-apiserver@sha256:ad3a0ff8f8ef783e097566f35bf40087d8a454abd37a6c78afead04f9b0c1a17
    Port:          <none>
    Host Port:     <none>
    Command:
      kube-apiserver
      --advertise-address=10.164.0.63
      --allow-privileged=true
      --authorization-mode=Node,RBAC
      --client-ca-file=/etc/kubernetes/pki/ca.crt
      --enable-admission-plugins=NodeRestriction
      --enable-bootstrap-token-auth=true
      --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
      --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
      --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
      --etcd-servers=https://127.0.0.1:2379
      --insecure-port=0
      --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
      --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
      --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
      --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
      --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
      --requestheader-allowed-names=front-proxy-client
      --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
      --requestheader-extra-headers-prefix=X-Remote-Extra-
      --requestheader-group-headers=X-Remote-Group
      --requestheader-username-headers=X-Remote-User
      --secure-port=6443
      --service-account-key-file=/etc/kubernetes/pki/sa.pub
      --service-cluster-ip-range=10.96.0.0/12
      --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
      --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    State:          Running
      Started:      Fri, 13 Nov 2020 10:30:22 +0000
    Ready:          True
    Restart Count:  0
    Requests:
      cpu:        250m
    Liveness:     http-get https://10.164.0.63:6443/healthz delay=15s timeout=15s period=10s #success=1 #failure=8
    Environment:  <none>
    Mounts:
      /etc/ca-certificates from etc-ca-certificates (ro)
      /etc/kubernetes/pki from k8s-certs (ro)
      /etc/ssl/certs from ca-certs (ro)
      /usr/local/share/ca-certificates from usr-local-share-ca-certificates (ro)
      /usr/share/ca-certificates from usr-share-ca-certificates (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  ca-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ssl/certs
    HostPathType:  DirectoryOrCreate
  etc-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/ca-certificates
    HostPathType:  DirectoryOrCreate
  k8s-certs:
    Type:          HostPath (bare host directory volume)
    Path:          /etc/kubernetes/pki
    HostPathType:  DirectoryOrCreate
  usr-local-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/local/share/ca-certificates
    HostPathType:  DirectoryOrCreate
  usr-share-ca-certificates:
    Type:          HostPath (bare host directory volume)
    Path:          /usr/share/ca-certificates
    HostPathType:  DirectoryOrCreate
QoS Class:         Burstable
Node-Selectors:    <none>
Tolerations:       :NoExecute
Events:
  Type    Reason   Age    From             Message
  ----    ------   ----   ----             -------
  Normal  Pulled   3m25s  kubelet, master  Container image "k8s.gcr.io/kube-apiserver:v1.17.14" already present on machine
  Normal  Created  3m25s  kubelet, master  Created container kube-apiserver
  Normal  Started  3m24s  kubelet, master  Started container kube-apiserver
```
**kubeadm: v1.18.0**

**kubelet: v1.18.0**

**kubectl client v1.18.0 server v1.17.14**

**api-server: v1.17.14**

- Обновим остальные компоненты кластера

```console
# kubeadm upgrade plan
# kubeadm upgrade apply v1.18.0
# kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   32m   v1.18.0
node-1   Ready    <none>   20m   v1.17.4
node-2   Ready    <none>   19m   v1.17.4
node-3   Ready    <none>   19m   v1.17.4
```
**kubeadm: v1.18.0**

**kubelet: v1.18.0**

**kubectl client v1.18.0 server v1.18.0**

**api-server: v1.18.0**

- Выводим worker-ноду из планирования и обновляем и возвращаем в работу.

```console
root@master:~# kubectl drain node-1 --ignore-daemonsets
root@master:~# kubectl get nodes -o wide
NAME     STATUS                     ROLES    AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
master   Ready                      master   45m   v1.18.0   10.164.0.63     <none>        Ubuntu 18.04.5 LTS   5.4.0-1029-gcp   docker://19.3.8
node-1   Ready,SchedulingDisabled   <none>   32m   v1.17.4   10.164.15.192   <none>        Ubuntu 18.04.5 LTS   5.4.0-1029-gcp   docker://19.3.8
node-2   Ready                      <none>   32m   v1.17.4   10.164.15.193   <none>        Ubuntu 18.04.5 LTS   5.4.0-1029-gcp   docker://19.3.8
node-3   Ready                      <none>   32m   v1.17.4   10.164.15.194   <none>        Ubuntu 18.04.5 LTS   5.4.0-1029-gcp   docker://19.3.8
root@node-1:~# apt install -y kubelet=1.18.0-00 kubeadm=1.18.0-00
root@node-1:~# systemctl restart kubelet
root@master:~# kubectl get nodes -o wide
NAME     STATUS                     ROLES    AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
master   Ready                      master   47m   v1.18.0   10.164.0.63     <none>        Ubuntu 18.04.5 LTS   5.4.0-1029-gcp   docker://19.3.8
node-1   Ready,SchedulingDisabled   <none>   34m   v1.18.0   10.164.15.192   <none>        Ubuntu 18.04.5 LTS   5.4.0-1029-gcp   docker://19.3.8
node-2   Ready                      <none>   34m   v1.17.4   10.164.15.193   <none>        Ubuntu 18.04.5 LTS   5.4.0-1029-gcp   docker://19.3.8
node-3   Ready                      <none>   34m   v1.17.4   10.164.15.194   <none>        Ubuntu 18.04.5 LTS   5.4.0-1029-gcp   docker://19.3.8
root@master:~# kubectl uncordon node-1
```

- Повторяем действия с оставшимися нодами

```console
root@master# kubectl drain node-2 --ignore-daemonsets
root@node-2# apt install -y kubelet=1.18.0-00 kubeadm=1.18.0-00
root@node-2# systemctl restart kubelet
root@master# kubectl uncordon node-2
root@master# kubectl drain node-3 --ignore-daemonsets
root@node-3# apt install -y kubelet=1.18.0-00 kubeadm=1.18.0-00
root@node-3# systemctl restart kubelet
root@master# kubectl uncordon node-3
root@master# kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   54m   v1.18.0
node-1   Ready    <none>   41m   v1.18.0
node-2   Ready    <none>   41m   v1.18.0
node-3   Ready    <none>   41m   v1.18.0
```

### Kubespray

- Создаем ноды и производим обмен ключами ssh и узнаем внешние ip виртуалок

```console
gcloud compute instances create master --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --machine-type=n1-standard-2 && \
> gcloud compute instances create node-1 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --machine-type=n1-standard-1 && \
> gcloud compute instances create node-2 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --machine-type=n1-standard-1 && \
> gcloud compute instances create node-3 --image-family ubuntu-minimal-1804-lts --image-project ubuntu-os-cloud --machine-type=n1-standard-1
Created [https://www.googleapis.com/compute/v1/projects/project-otus-287811/zones/europe-west4-c/instances/master].
NAME    ZONE            MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP    EXTERNAL_IP   STATUS
master  europe-west4-c  n1-standard-2               10.164.15.195  34.90.236.93  RUNNING
NAME    ZONE            MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP    EXTERNAL_IP  STATUS
node-1  europe-west4-c  n1-standard-1               10.164.15.196  34.90.85.26  RUNNING
NAME    ZONE            MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP    EXTERNAL_IP   STATUS
node-2  europe-west4-c  n1-standard-1               10.164.15.197  34.90.18.167  RUNNING
NAME    ZONE            MACHINE_TYPE   PREEMPTIBLE  INTERNAL_IP    EXTERNAL_IP   STATUS
node-3  europe-west4-c  n1-standard-1               10.164.15.198  34.90.214.66  RUNNING
```
```console
$ gcloud beta compute ssh master
$ gcloud beta compute ssh node-1
$ gcloud beta compute ssh node-2
$ gcloud beta compute ssh node-3
```
```console
$ for name in master node-1 node-2 node-3; do echo $name:; gcloud compute instances describe $name | grep natIP; done
master:
    natIP: 34.90.236.93
node-1:
    natIP: 34.90.85.26
node-2:
    natIP: 34.90.18.167
node-3:
    natIP: 34.90.214.66
```

- Устанавливаем
```console
# получение kubespray
git clone https://github.com/kubernetes-sigs/kubespray.git
# установка зависимостей
sudo pip3 install -r requirements.txt
# копирование примера конфига в отдельную директорию
cp -rfp inventory/sample inventory/mycluster
```

- Правим файл ***inventory/mycluster/inventory.ini***
```ini
[all]
master1 ansible_host=34.90.236.93  ip=10.164.15.195 etcd_member_name=etcd1
node1 ansible_host=34.90.85.26    # ip=10.3.0.1 etcd_member_name=etcd1
node2 ansible_host=34.90.18.167    # ip=10.3.0.1 etcd_member_name=etcd1
node3 ansible_host=34.90.214.66    # ip=10.3.0.1 etcd_member_name=etcd1

[kube-master]
master1

[etcd]
master1

[kube-node]
node1
node2
node3

[calico-rr]

[k8s-cluster:children]
kube-master
kube-node
calico-rr
```

- Применяем
```console
$ ansible-playbook -i inventory/mycluster/inventory.ini --become --become-user=root \
    --user=dimon --key-file="~/.ssh/google_compute_engine" cluster.yml
```

- Логинимся на матер-ноду и проверяем
```console
root@master1:~# kubectl get node -o wide
NAME      STATUS   ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
master1   Ready    master   9m5s    v1.19.3   10.164.15.195   <none>        Ubuntu 18.04.5 LTS   5.4.0-1029-gcp   docker://19.3.13
node1     Ready    <none>   7m54s   v1.19.3   10.164.15.196   <none>        Ubuntu 18.04.5 LTS   5.4.0-1029-gcp   docker://19.3.13
node2     Ready    <none>   7m54s   v1.19.3   10.164.15.197   <none>        Ubuntu 18.04.5 LTS   5.4.0-1029-gcp   docker://19.3.13
node3     Ready    <none>   7m53s   v1.19.3   10.164.15.198   <none>        Ubuntu 18.04.5 LTS   5.4.0-1029-gcp   docker://19.3.13
```

