---
published: true
layout: single
title: "[Airflow] Helm을 사용하여 k8s airflow에 git.sync sidecar 적용"
category: airflow
tags:
comments: true
sidebar:
  nav: "mainMenu"
---
* * *

KubernetesExecutor로 재배포할 때 사용한 values.yaml 파일을 사용하여 git-sync sidecar 기능을 사용할 수 있습니다. 
해당 파일을 열어보면 gitSync라는 설정 부분이 있습니다. 다만 그전에 git repo를 사용하기 위한 ssh를 생성해주도록 하겠습니다.

```
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

<br>

사용할 repo에 ssh-key를 등록하여 줍시다.

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/99048f30-a319-4723-9725-979926adb852)

그리고 생성한 ssh key를 secret 오브젝트로 생성해줍니다.

```
kubectl create secret generic airflow-ssh-secret \
--from-file=gitSshKey=/home/ysbaek/.ssh/id_rsa \
--from-file=known_hosts=/home/ysbaek/.ssh/known_hosts \
--from-file=id_rsa.pub=/home/ysbaek/.ssh/id_rsa.pub -n airflow
```

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/c99f3f59-3e3e-4ed5-9d98-ba816c2477f7)

<br>

그리고 values.yaml의 enabled, branch, rev, subPath, credentialsSecret, sshKeySecret 부분을 수정하였습니다. 수정 예시는 아래 yaml을 확인해주세요

<details>
<summary>변경한 value.yaml 예시 - 펼치기 </summary>
<div markdown="1">

```
  gitSync:
    enabled: true

    # git repo clone url
    # ssh example: git@github.com:apache/airflow.git
    # https example: https://github.com/apache/airflow.git
    repo: git@github.com:{your-id}/{repo-name}.git
    branch: main
    rev: HEAD
    depth: 1
    # the number of consecutive failures allowed before aborting
    maxFailures: 0
    # subpath within the repo where dags are located
    # should be "" if dags are at repo root
    subPath: ""
    # if your repo needs a user name password
    # you can load them to a k8s secret like the one below
    #   ---
    #   apiVersion: v1
    #   kind: Secret
    #   metadata:
    #     name: git-credentials
    #   data:
    #     GIT_SYNC_USERNAME: <base64_encoded_git_username>
    #     GIT_SYNC_PASSWORD: <base64_encoded_git_password>
    # and specify the name of the secret below
    #
    credentialsSecret: git-credentials
    #
    #
    # If you are using an ssh clone url, you can load
    # the ssh private key to a k8s secret like the one below
    #   ---
    #   apiVersion: v1
    #   kind: Secret
    #   metadata:
    #     name: airflow-ssh-secret
    #   data:
    #     # key needs to be gitSshKey
    #     gitSshKey: <base64_encoded_data>
    # and specify the name of the secret below
    sshKeySecret: airflow-ssh-secret
    #
    # If you are using an ssh private key, you can additionally
    # specify the content of your known_hosts file, example:
    #
    # knownHosts: |
    #    <host1>,<ip1> <key1>
    #    <host2>,<ip2> <key2>

    # interval between git sync attempts in seconds
    # high values are more likely to cause DAGs to become out of sync between different components
    # low values cause more traffic to the remote git repository
    wait: 60
    containerName: git-sync
    uid: 65533

    # When not set, the values defined in the global securityContext will be used
    securityContext: {}
    #  runAsUser: 65533
    #  runAsGroup: 0

    securityContexts:
      container: {}

    # Mount additional volumes into git-sync. It can be templated like in the following example:
    #   extraVolumeMounts:
    #     - name: my-templated-extra-volume
    #       mountPath: "{{ .Values.my_custom_path }}"
    #       readOnly: true
    extraVolumeMounts: []
    env: []
    # Supported env vars for gitsync can be found at https://github.com/kubernetes/git-sync
    # - name: ""
    #   value: ""

    resources: {}
    #  limits:
    #   cpu: 100m
    #   memory: 128Mi
    #  requests:
    #   cpu: 100m
    #   memory: 128Mi
```
</div>
</details>

<br>

이후 helm을 사용하여 airflow를 재배포하여 줍니다.

```
$ helm upgrade --install airflow apache-airflow/airflow -n airflow -f values.yaml --debug
```

<br>

배포 완료 후, Pod 상태를 확인해줍니다.

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/22f11585-d810-4929-b47a-54803b4d4fee)
