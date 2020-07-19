# 7월14 일 작업 이슈
## subscription-manager 작업 이슈

```
*** 사용 commend ****

#subscription-manager register --username=hyg6565 --password=Hong7828@@

#subscription-manager refresh

#subscription-manager list --available --matches '*OpenShift*'

#subscription-manager attach --pool=8a85f99c707807c801709f913ded7153

#yum install openshift-ansible openshift-clients jq

#subscription-manager repos     \
   
   --enable="rhel-7-server-rpms"      \
   
   --enable="rhel-7-server-extras-rpms"      \
   
   --enable="rhel-7-server-ansible-2.8-rpms"      \
  
  --enable="rhel-7-server-ose-4.3-rpms"

#yum install openshift-ansible openshift-clients jq

#subscription-manager list

#subscription-manager status


*** 작업 이슈 ***

#subscription-manager list --available --matches '*OpenShift*'

"*OpenShift*" 표현식에 일치하는 사용 가능한 서브스크립션 풀을 찾을 수 없습니다.

/subscription-manager list --available --matches '*OpenShift*' 작업 중 오류

"*OpenShift*" 표현식과 일치하는 사용 가능한 구독 풀을 찾을 수 없습니다. 라는 문구 가 뜸
```

## 작업 환경 변경
```
- 작업 구성을 10.0.10.222(라처송 과장님 의 registry 노드 접속)
- 현재 registry 노드 에 존재하는 팻키지 = podman, openshift-client, httpd-tools, mirror-registry, FTP
- 접속을 위해 registry 노드 의 접속 허용 작업 실핼
```

## yum repo 작성
```
#vi /etc/yum.repos.d/ocp4-4.repo 

[rhel-7-server-ansible-2.8-rpms]
name=rhel-7-server-rpms
baseurl=ftp:// 10.0.10.222/pub/repos/rhel-7-server-rpms
enabled=1
gpgcheck=0
[rhel-7-server-extras-rpms]
name=rhel-7-server-extras-rpms
baseurl=ftp:// 10.0.10.222/pub/repos/rhel-7-server-extras-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ansible-2.8-rpms]
name=rhel-7-server-ansible-2.8-rpms
baseurl=ftp:// 10.0.10.222/pub/repos/rhel-7-server-ansible-2.8-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ansible-2.9-rpms]
name=rhel-7-server-ansible-2.9-rpms
baseurl=ftp:// 10.0.10.222/pub/repos/rhel-7-server-ansible-2.9-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ose-4.4-rpms]
name=rhel-7-server-ose-4.4-rpms
baseurl=ftp:// 10.0.10.222/pub/repos/rhel-7-server-ose-4.4-rpms
enabled=1
gpgcheck=0


- subscription-manager 를 통하여 서비스 틍록이 되지 않으면 yum 명령어 사용이 되지 
않아 수동로 yum 명령어를 사용가능 하게 하는 작업 

- 위의 내용을 구성한 ‘ocp4-4.repo” 파일 생성
- 테스트: # yum update   >> 성공 
```

## yum repo 작업 내용
```
- [base]
- Name    >> 저장소 표시이름, 위에선 저장소 이름표시를 rhel-7-server-rpm 으로 설정
- Basurl    >> 저장소 주소이다, 위에서는 register속 ftp:// 10.0.10.222/pub/repos/rhel-7-server-rpms 에 저장된 내용을 가져온다.
- Enable    >> 활성화 여부 (0 =NO 1 =YES)
- gpgcheck   >> 서명키 사용 여부 (0 =NO 1 =YES)
```
## 방화벽 설정
```

#firewall-cmd --permanent --zone=external --add-port=80/tcp
#firewall-cmd --permanent --zone=external --add-port=443/tcp
#firewall-cmd --permanent --zone=external --add-port=5000/tcp

## internal zone firewall
#firewall-cmd --permanent --zone=internal --add-service=ftp
#firewall-cmd --permanent --zone=internal --add-service=dns
#firewall-cmd --permanent --zone=internal --add-service=tftp
#firewall-cmd --permanent --zone=internal --add-service=ssh
#firewall-cmd --permanent --zone=internal --add-service=dhcp
#firewall-cmd --permanent --zone=internal --add-service=proxy-dhcp

#firewall-cmd --permanent --zone=internal --add-port=53/tcp
#firewall-cmd --permanent --zone=internal --add-port=80/tcp
#firewall-cmd --permanent --zone=internal --add-port=443/tcp
#firewall-cmd --permanent --zone=internal --add-port=6443/tcp
#firewall-cmd --permanent --zone=internal --add-port=8443/tcp
#firewall-cmd --permanent --zone=internal --add-port=10256/tcp
#firewall-cmd --permanent --zone=internal --add-port=2379-2380/tcp
#firewall-cmd --permanent --zone=internal --add-port=9000-9999/tcp
#firewall-cmd --permanent --zone=internal --add-port=10249-10259/tcp
#firewall-cmd --permanent --zone=internal --add-port=22623/tcp
#firewall-cmd --permanent --zone=internal --add-port=5000/tcp

#firewall-cmd --permanent --zone=internal --add-port=53/udp
#firewall-cmd --permanent --zone=internal --add-port=4789/udp
#firewall-cmd --permanent --zone=internal --add-port=6081/udp
#firewall-cmd --permanent --zone=internal --add-port=9000-9999/udp
#firewall-cmd --permanent --zone=internal --add-port=30000-32767/udp

/// 방화벽 대몬 재실행
#firewall-cmd –reload 
```

###방화벽 개념 정리
```
 firewalld 는 방화벽을 관리하는 데몬 ‘firewall-cmd’ 로 명령어 정의
Ex) #firewall-cmd -- reload    방화벽 설정 후 다시 로드
             --state     실행 중이면 running, 실행 중이 아니면 not running을 출력
firewall-cmd --list-al   사용가능한 서비스/포트 목록 출력
# firewall-cmd --permanent --zone=external --add-port=80/tcp
 Permanent는 시스템,방화벽 재부팅 이후에도 적용 시킨다는 것 
--zone=external 는 zone 기본 옵션이 default zone 이기에 위 명령어 에서는 internal 과 external 존을 지정
내부,외부 포트에 작업을 한 것
# firewall-cmd --permanent --zone=internal --add-service=ftp
   --add-port =80/tcp    >   는 tip 프로토콜 80번 포트를 추가 한다는 뜻
   --add-service=ftp    > 는 ftp 서비스를 추가한다는 뜻
```

## 계정추가
```
Core 계정 생성
#Useradd core passwd core

/ coreos 설치시 core 의 계정이 파일에 접근 하기에 파일 계정 추가 필요
```

## FTP 설치 진행 
```
#yum install vsftpd -y   

///yum 으로 vsftp 설치

#cd /var/ftp/pub/     

/// FTP 의 기본 접속 디렉터리 확인

FTP 설치 running 작업 및 상태 확인

#systemctl enable vsftpd
#systemctl start vsftpd
#systemctl status vsftpd
#curl ftp://10.0.10.222/pub 
>>> 과장님의 Registry 노드에 접속 확인
```

 






