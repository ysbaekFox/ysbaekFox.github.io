---
published: true
layout: single
title: "[Airflow] Helm을 사용하여 k8s cluster에 Airflow 설치"
category: airflow
tags:
comments: true
sidebar:
  nav: "mainMenu"
---
* * *

## Reference page

아래 페이지를 참고하여 설치를 진행하였습니다.
- [Helm Chart for Apache Airflow](https://airflow.apache.org/docs/helm-chart/1.9.0/)

## helm 설치

먼저 helm을 설치하여 줍니다.

```
$ curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/f0b2e2aa-78f8-4304-9152-28daf4c9760b)

## Persistent Volume 배포

Airflow Chart 설치 시 배포 되는 PVC에 연결될 PV를 미리 배포 해줍니다. 10GB 2개, 110GB 2개로 설정해주었습니다. 
최소 용량이 얼마나 필요한지 모르겠으나 적어도 2개 PVC가 110GB 이상을 요구하는 것 같았습니다. 그렇지 않은 경우 Pending 상태로 무한 대기상태에 빠지더군요.
  
참고로 아래 yaml을 사용하여 PV를 배포하기 전에 node local에 **yaml에 정의한 PV 경로를 꼭 미리 생성해 주세요**. 
그렇지 않으면 배포된 airflow chart에서 권한 문제로 인해 Pod가 일정시간마다 재시작 되는 문제가 발생합니다. 

<details>
<summary>pvc.yaml</summary>
<div markdown="1">

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: persistent-volume-1
spec:
  capacity:
    storage: 10G
  accessModes:
  - ReadWriteOnce
  local:
    path: /home/ysbaek/pv1
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [worker-1]}
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: persistent-volume-2
spec:
  capacity:
    storage: 10G
  accessModes:
  - ReadWriteOnce
  local:
   path: /home/ysbaek/pv2
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [worker-2]}
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: persistent-volume-3
spec:
  capacity:
    storage: 110G
  accessModes:
  - ReadWriteOnce
  local:
    path: /home/ysbaek/pv3
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [worker-1]}
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: persistent-volume-4
spec:
  capacity:
    storage: 110G
  accessModes:
  - ReadWriteOnce
  local:
    path: /home/ysbaek/pv4
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - {key: kubernetes.io/hostname, operator: In, values: [worker-2]}
```

</div>
</details>

<br>
```
kubectl apply -f pvc.yaml
```

<br>
이후에 바인딩에 성공할 경우 아래와 같이 표시되어야합니다. 참고로 바인딩은 helm airflow chart 설치후에 진행될 것이고, PV의 최초 Status는 Available이어야 합니다.
![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/f7edbd46-3d12-4840-90e8-23e98cff24d2)


## airflow helm chart 설치

아래 명령어를 입력하면, "airflow" does not exist. Installing it now. 가 표시되고 설치 및 배포를 시작합니다.

```
helm repo add apache-airflow https://airflow.apache.org
helm upgrade --install airflow apache-airflow/airflow --namespace airflow --create-namespace
```

설치가 완료되면 아래와 같이 airflow namespace에 오브젝트들이 배포되어야 합니다.

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/c5190356-0993-4a55-bebd-244e5c06612b)


## airflow webserver 포트포워딩

저는 외부에서 접속할 수 있도록 하기 위해 접속 가능한 ip를 모든 ip로 설정해주었고, iptime 공유기의 포트포워딩 설정도 변경하여주었습니다.

```
kubectl port-forward --address=0.0.0.0 svc/airflow-webserver 8080:8080 -n airflow
```

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/0707fb28-a913-46ea-80f7-23ceb1519b52)

## 설치 완료

외부에서 서버 환경의 webserver로 정상적으로 접속하는 것 확인하였습니다.

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/2c91c8e9-2404-4523-bf01-64c781456fd1)
