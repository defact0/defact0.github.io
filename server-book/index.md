# Server 관련 핸드북


회사에서 자주 사용하고 있는 명령어들을 정리한 핸드북이다.  
CentOS 7 을 메인으로 사용 중이다.

<!--more-->

![br](br.png)

## CentOS 7 방화벽 설정

{{< admonition >}}
CentOS6까지는 방화벽 설정을 위해 iptables 서비스 사용 했지만,  
CentOS7부터는 firewalld를 사용한다.
{{< /admonition >}}

**포트 등록(전체)**
```shell
firewall-cmd --permanent --zone=public --add-port=8080/tcp
```

**포트 해제(전체)**
```shell
firewall-cmd --permanent --zone=public --remove-port=8080/tcp
```

**rule family 등록**
```shell
firewall-cmd --permanent --zone=public --add-rich-rule='rule family="ipv4" source address="10.1.1.10" port protocol="tcp" port="8080" accept'
```

**rule family 해제**
```shell
firewall-cmd --permanent --remove-rich-rule='rule family="ipv4" source address="10.1.1.10" port port="8080" protocol="tcp" accept'
```

**정책 반영**
```shell
firewall-cmd --reload
```

**정책 확인**
```shell
firewall-cmd --list-all
```
![br](br.png)

## Dokcer 관련

**전체 container 리스트 확인**
```shell
docker ps -a
```

**images 리스트 확인**

```shell
docker images
```

**실행 중인 컨테이너 저장**
```shell
docker commit 컨테이너이름 저장소이름:태그명
```

**저장된 컨테이너 tar 파일로 저장**
```shell
docker save -o 저장파일명.tar 저장소이름:태그명
```

**tar 파일 읽기**
```shell
docker load -i 저장파일명.tar
```
