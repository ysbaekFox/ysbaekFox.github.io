---
published: true
layout: single
title: "[Airflow] KubernetesExecutor의 Log Persistence 및 ExtraVolume 적용"
category: airflow
tags:
comments: true
sidebar:
  nav: "mainMenu"
---
* * *

KubernetesExecutor를 적용한 후에 각 Task가 실행될 때마다 새로운 Pod가 생성되고 삭제되며 동작하는 것을 확인했습니다.
그런데 Airflow WebServer를 사용해서 DAG 실행 로그도 확인할 수가 없고, 심지어 다운로드한 데이터도 어디에 있는지 도저히 찾을 수 없었습니다.

```
# Web Server애서 로그를 보려고 하면 표시되는 메세지
*** Could not read served logs: [Errno -2] Name or service not known
```
  
고민을 좀 해본 결과, 해당 Pod들은 Empty Volume이 Default 설정이어서 Pod가 매번 생성되고 삭제될 때마다 로그를 포함한 모든 Data가 삭제되는 문제가 아닐까?하고 생각했습니다.

레퍼런스 페이지를 확인 해보니 logs.persistence.enabled / logs.persistence.existingClaim이라는 셋팅이 있었습니다. 해당 PVC를 바인딩 시킨 후, 
파라미터를 적용하여 helm chart를 업그레이드 해주니 log가 정상적으로 표시되는 것을 확인할 수 있었습니다.
- [Airflow Helm Parameters reference](https://airflow.apache.org/docs/helm-chart/stable/parameters-ref.html)

logs.persistence 성공적으로 구성한 방법은 다음과 같습니다.

**1) pvc 바인딩**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: airflow-log-pvc
  namespace: airflow
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 50G
  storageClassName: ""
```

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/952d431b-bff3-49f1-bda5-8fb23decac4c)

**2) helm chart upgrade**

```shell
helm upgrade --install airflow apache-airflow/airflow \ 
--set logs.persistence.enabled=true \ 
--set logs.persistence.existingClaim=airflow-log-pvc \
-n airflow -f values.yaml --debug
```

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/07879c4f-f564-4f1e-ab19-5869ab5e5ca7)

## Task Pod에 영구 볼륨 적용

하지만 여전히 curl 명령어를 사용하여 다운로드 한 데이터를 찾을 수 없는 문제가 있었습니다. 
Data를 다운로드하는 경로가 Pod에 "Empty Dir"로 Mount된 경로이기 때문일거라 생각했습니다. 
여러가지 파라미터를 검토해보았는데 extraVolumeMounts / extraVolumes을 사용하는게 적당해 보였습니다. 다만 해당 파라미터를 
구체적으로 어떻게 사용해야 하는지에 대해서는 레퍼런스 페이지에서도 나와있지 않아서 구글링을 통해 예제를 찾았고 의도한 대로 동작하는 것도 확인하였습니다.

extra-persistence-volume-claim의 마운트 경로는 worker-2 node의 /pv 경로인데요, Task Pod가 종료된 이후에도 해당 경로에 다운로드 했었던 이미지가 남아있는 것을 확인할 수 있었습니다.

**1) pvc 바인딩**

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: extra-persistence-volume-claim
  namespace: airflow
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 50G
  storageClassName: ""
```

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/a04c53df-a9d0-478b-9a83-913299949211)

**2) helm chart upgrade**

아래 예제는 Log.persistence를 사용법과 하나로 합친 것입니다. 

```shell
helm upgrade --install airflow apache-airflow/airflow \
--set logs.persistence.enabled=true \
--set logs.persistence.existingClaim=airflow-log-pvc \
--set workers.extraVolumeMounts[0].name=extra-persistence-volume \
--set workers.extraVolumeMounts[0].mountPath=/opt/airflow/share \
--set workers.extraVolumeMounts[0].readOnly=false \
--set workers.extraVolumes[0].name=extra-persistence-volume \
--set workers.extraVolumes[0].persistentVolumeClaim.claimName=extra-persistence-volume-claim \
-n airflow -f values.yaml --debug
```

worker-2의 pv4 경로에 정상 다운로드 된 것 확인할 수 있습니다.

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/a5ef1afd-f8b5-4500-87b4-3588ee341a60)

마지막으로 이 문제를 해결할 때 제가 설정한 셋팅이 정상적으로 반영이 되었는지 확인하기 위해 제가 사용한 방법(팁?)을 알려드리겠습니다. 
바로 스케쥴 Pod에 접속해서 Pod Template yaml 파일을 확인하는 것이 도움이 많이 되었습니다.
  
아마도 제가 설정한 parameter를 기반으로 pod template yaml 파일이 자동으로 생성되는 것 같았습니다.
  
예를 들어서 AirFlow Update를 수행하고나서 모든 파드에 문제가 없을 때, 
kubectl exec 명령어를 통해 schedule pod에 접속해서 Pod template yaml 파일을 확인하면, 제가 설정한 log.persistence나 extra volume이 설정되어있는 것을 확인할 수 있었습니다.

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/c4c8f1a1-4531-4697-80c1-cd8e92400a10)

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/d60aea6b-d158-4f87-ac15-c6c3e8431c79)