---
title: 앤서블(Ansible) 시작하기
date: 2021-04-26
tags: [ansible, automation]
categories: [Server]
featuredImage: "thumnail.png"
featuredImagePreview: "thumnail.png"
---

**Ansible**은 서버의 설정 및 관리, 소프트웨어 배포 등 다수의 서버에 대해 자동화를 구성하여 관리할 수 있는 도구입니다.

<!--more-->

![br](br.png)

{{< admonition note "테스트 시나리오">}}
Ansible Controller 에서 Ansible Client 으로 신규 계정 생성 및 패스워드 설정을 하는 작업을 수행한다.
{{< /admonition >}}

![br](br.png)

## 버추얼박스로 테스트 환경 구성

- CentOS 7 환경 2개
    - Ansible Controller (192.168.56.20)
    - Ansible Client (192.168.56.30)
-  [호스트 네트워크를 통한 구성](https://defact0.github.io/virtualbox/#%ED%98%B8%EC%8A%A4%ED%8A%B8-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC%EB%A5%BC-%ED%86%B5%ED%95%9C-%EA%B5%AC%EC%84%B1-19216856x)

![br](br.png)

## Ansible Client 관리자 계정 생성

- root 계정으로 접속 및 su 권한을 막을 예정
- root 를 대신할 계정을 만든다.
  - 계정은 `rocketman` 으로 하고 `wheel` 그룹설정을 한다.

![br](br.png)

```shell
# root 계정에서 계정 생성 및 비밀번호 변경
useradd rocketman
passwd rocketman

# wheel 그룹에 계정 추가
usermod -G wheel rocketman
vi /etc/pam.d/su

#auth required pam_wheel.so use_uid 주석을 지운다.
```

![br](br.png)

## Ansible Controller

ssh-keygen 생성, `root` 계정에서 작업한다.

```shell
# key 생성
ssh-keygen

# key 전송
ssh-copy-id root@192.168.56.30
```

- key 패스워드를 입력한다.

![br](br.png)

yum을 통해 `Ansible ` 설치 진행

```shell
 yum install -y epel-release
 yum install -y ansible
```

![br](br.png)

hosts 파일은 앞서 설명한 Ansible의 Inventory 이다.

```shell
vi /etc/ansible/hosts

[real]
rocketman@192.168.56.30
```

![br](br.png)

`ping` 명령어로 client 연결 체크

```shell
ansible all -m ping
```

![br](br.png)

ansible-playbook 만들기

```shell
vi /etc/ansible/test.yml
```
![br](br.png)

apple계정 생성 및 패스워드 설정(test.yml)

```
---
- hosts: real
  remote_user: rocketman
  vars:
    password: $1$SomeSalt$bV8bhWtZerRl2ACgzd7gC0
  tasks:
    - name: Test connection
      ping:

    - name: Create User
      become: yes
      user: name=apple password={{password}}
```

![br](br.png)

- 패스워드는 파이썬을 이용하여 생성한다.

```shell
python -c 'import crypt; print crypt.crypt("P@ssw0rd", "$1$SomeSalt$")'
# 아래 문자를 test.yml에 password 부분에 입력
$1$SomeSalt$bV8bhWtZerRl2ACgzd7gC0
```

![br](br.png)

ansible-playbook 실행

```shell
ansible-playbook test.yml -k -K
```
