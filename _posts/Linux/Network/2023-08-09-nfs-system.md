---
published: true
layout: single
title: "[Linux] VirtualBox를 활용한 NFS Server / Client 통신 구축"
category: linux-network
tags:
comments: true
sidebar:
  nav: "mainMenu"
---
* * *

최근에 쿠버네티스의 동적 프로비저닝 기능을 공부하면서, NFS(Network File System)를 구축하고 동적 프로비저닝에 활용해보고 싶어졌습니다. 
평소 업무에 NFS를 사용해본적이 있지만, 직접 구축해본적은 없어서 개인 Server에 구축해보았습니다.

참고로 제가 구축한 환경은 Virtual Box에서 구동되는 control-plane 노드를 포함한 모든 쿠버네티스 Node들이 Client이고  Virtual Box를 구동하는 Host가 NFS Server입니다.

## 1. 방화벽 설정

방화벽 상태를 확인하려면 ufw (Uncomplicated Firewall) 도구를 사용할 수 있습니다. 다음 명령을 통해 현재 방화벽 상태를 확인합니다:

기본적으로 NFS는 TCP/UDP 2049번 포트를 사용합니다. 다음 명령으로 현재 열려 있는 포트를 확인합니다:

```
sudo ufw status
```

필요한 NFS 관련 포트를 방화벽에서 허용하려면 ufw 명령을 사용하여 규칙을 추가합니다. NFS TCP/UDP 2049번 포트를 열어보겠습니다:

```
sudo ufw allow 2049/tcp
sudo ufw allow 2049/udp
```

방화벽 설정을 변경한 후에는 변경 내용을 적용하려면 ufw를 재시작합니다:

```
sudo ufw reload
```

## 2. NFS Server 설치

NFS Server 구축에 필요한 패키지를 설치합니다.
```
sudo apt-get update
sudo apt-get install -y nfs-common nfs-kernel-server rpcbind portmap
```

NFS Server를 설정합니다.
```
sudo mkdir /mnt/shared
sudo chmod 777 /mnt/shared
sudo echo '/mnt/shared *(rw,sync,no_subtree_check)' >> /etc/exports
```

참고로 초기 root user가 없는 경우 제대로 동작하지 않을 수 있습니다. 
해당 경우에 root user를 설정해주도록 합니다.
```
sudo passwd root
```

nfs server 서비스를 reload 해줍니다.
```
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

## 3. NFS Client 설치

NFS Client 설치 시엔 nfs-common 패키지만 있으면 됩니다.
```
sudo apt-get update
sudo apt-get install -y nfs-common
```

NFS Server 경로를 mount할 경로를 생성 해줍니다
```
sudo mkdir /mnt/shared
sudo chmod 777 /mnt/shared
```

NFS Server 경로를 mount 해줍니다. 저의 경우 Server의 IP가 192.168.0.2 였습니다.
```
sudo mount -t nfs -v 192.168.0.2:/mnt/shared /mnt/shared
```

## 4. 테스트

server에서 mount 경로에 text 파일을 하나 생성 합니다.

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/369a4c71-239d-421a-8f79-a9c6feaf4ec4)

client에서 mount 경로에 text 파일을 확인할 수 있습니다.

![image](https://github.com/ysbaekFox/ysbaekFox.github.io/assets/54944434/b236c381-cee2-4d0a-b360-ed3453d6157d)