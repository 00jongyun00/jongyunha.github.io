---
title: How to docker install without internet
date: 2023-03-04
description: '인터넷 없는 환경에서 docker 를 설치하는 방법'
tags: [docker, docker-ce, container-d, redhat, centos, debian]
---

# Approach

인터넷이 없는 환경에서 docker 를 설치하기 위해서는 binary file 이 미리 준비되어 있어야 합니다.

### debian

```shell
sudo apt-get update -y
sudo apt-get download $(apt-cache depends --recurse --no-recommends --no-suggests \
    --no-conflicts --no-breaks --no-replaces --no-enhances \
    --no-pre-depends docker-ce docker-ce-cli containerd.io docker-buildx-plugin | grep "^\w")
sudo apt-get update -y
mkdir dependency
tar xvf docker.tar.gz -C ~/dependency
```

### redhat

```
sudo yum update -y
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

sudo yum update -y
yumdownloader --resolve docker-ce
mkdir dependency
tar xvf docker.tar.gz -C ~/dependency
```

위의 명령어를 통해 docker 에 관한 파일을 다운 받습니다.

그리고 설치 하실 서버에 미리 업로드 하신뒤 다음과 같은 명령어로 docker 를 install 합니다.

### debian

```
tar -xvf docker.tar.gz
sudo dpkg -i *.deb
```

### redhat

```
tar -xvf debian.tar
rpm -ivh --replacefiles --replacepkgs *.rpm
```

다운받은 환경과 설치할 환경의 OS 가 동일해야 문제없이 동작합니다.
