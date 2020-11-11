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
