---
published: true
layout: single
title: "[Airflow] Helm을 사용하여 CeleryExecutor를 KubernetesExecutor로 재배포"
category: airflow
tags:
comments: true
sidebar:
  nav: "mainMenu"
---  
* * *

먼저 설정 파일을 아래처럼 다운로드하도록 합니다.

```
$ helm show values apache-airflow/airflow > values.yaml
```

<br>

그리고 다운로드한 values.yaml에 정의된 'executor'에 CeleryExecutor를 KubernetesExecutor로 변경해줍니다.

```
# Enable RBAC (default on most clusters these days)
rbac:
  # Specifies whether RBAC resources should be created
  create: true
  createSCCRoleBinding: false

# Airflow executor
# Options: LocalExecutor, CeleryExecutor, KubernetesExecutor, CeleryKubernetesExecutor
executor: "CeleryExecutor"

# If this is true and using LocalExecutor/KubernetesExecutor/CeleryKubernetesExecutor, the scheduler's
# service account will have access to communicate with the api-server and launch pods.
# If this is true and using CeleryExecutor/KubernetesExecutor/CeleryKubernetesExecutor, the workers
# will be able to launch pods.
allowPodLaunching: true

# Environment variables for all airflow containers
env: []
```

<br>

파일을 수정했으면 수정된 파일을 upgrade command를 사용하여 적용시켜줍니다. 이전에 네임스페이스를 airflow로 생성해줬기 때문에 배포할 네임스페이스도 airflow로 지정해주어야합니다.

```
$ helm upgrade --install airflow apache-airflow/airflow -n airflow -f values.yaml --debug
```

<br>

배포가 끝나면 pod 목록이 바뀝니다. CeleryExecutor를 사용할 때 있었던 redis, worker가 없어진걸 확인할 수 있습니다. DAG 안에 task가 실행되는 단위가 pod기 때문에 현재는 대기하는 worker가 없어서 그런 것 입니다.

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/24850b88-4ba0-4de9-af16-b81589083ffd)
