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
- 현재 registry 노드 에 존재하는 팻키지 = podman, openshift-client,
  httpd-tools, mirror-registry, FTP
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
- Basurl    >> 저장소 주소이다, 위에서는 register속 
  ftp:// 10.0.10.222/pub/repos/rhel-7-server-rpms 에 저장된 내용을 가져온다.
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

## haproxy 설치
```
#yum install haproxy -y
#setsebool -P haproxy_connect_any 1 /// selenux 에서 haproxy 의 접속 을 허용

##vi /etc/haproxy/haproxy.cfg 내용 변경

*** 아래의 문구를 붙여 넣었습니다. ***

balancing for OCP Kubernetes API Server
##
frontend openshift-api-server
    bind *:6443
    default_backend openshift-api-server
    mode tcp
    option tcplog

backend openshift-api-server
    balance source
    mode tcp
    server bootstrap 10.0.10.224:6443 check
    server master1 10.0.10.227:6443 check
##
# balancing for OCP Machine Config Server
##
frontend machine-config-server
    bind *:22623
    default_backend machine-config-server
    mode tcp
    option tcplog

backend machine-config-server
    balance source
    mode tcp
    server bootstrap 10.0.10.224:22623 check
    server master1 10.0.10.227:22623 check

##
# balancing for OCP Ingress Insecure Port & Admin Page
##
frontend ingress-http
    bind *:80
    default_backend ingress-http
    mode tcp
    option tcplog

backend ingress-http
    balance source
    mode tcp
    server worker1 10.0.10.228:80 check

##
# balancing for OCP Ingress Secure Port
##
frontend ingress-https
    bind *:443
    default_backend ingress-https
    mode tcp
    option tcplog

backend ingress-https
    balance leastconn
#    balance source
    mode tcp
    server worker1 10.0.10.228:443 check

#---------------------------------------------------------------------
# static backend for serving up images, stylesheets and such
#---------------------------------------------------------------------
backend static
    balance     roundrobin
    server      static 127.0.0.1:4331 check

#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend app
    balance     roundrobin
    server  app1 127.0.0.1:5001 check
    server  app2 127.0.0.1:5002 check
    server  app3 127.0.0.1:5003 check
    server  app4 127.0.0.1:5004 check

----------------------------------------------- 
bastion ip : 10.0.10.224
 bootstrap ip : 10.0.10.226
 master01 ip : 10.0.10.227
 worker1 ip : 10.0.10.228

```
## DNS 구성

```
dns 구성 정보
-----------------------
bastion ip : 10.0.10.224 
bootstrap ip : 10.0.10.226 
master01 ip : 10.0.10.227 
worker1 ip : 10.0.10.228
FQDN : opc4-1.fu.te
-----------------------
```
## bind install
```
#yum install bind

```
```
# vi /etc/named.conf <= replace
options {
        listen-on port 53 { ANY; };     => port 53 을 listen 으로 상태 변경
        listen-on-v6 port 53 { ANY; };  => port 53 에 ipv6 사용을 허용
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        recursing-file  "/var/named/data/named.recursing";
        secroots-file   "/var/named/data/named.secroots";
        allow-query     { ANY; };       => query값 localhost 를 허용으로 변경

#  vi /etc/named.rfc1912.zones  <= append
zone "ocp4-1.fu.te" IN {
        type master;
        file "ocp4-1.fu.te.zone";
        allow-update { none; };
};

zone "10.0.10.in-addr.arpa" IN {
        type master;
        file "ocp4-1.fu.te.rr";
        allow-update { none; };
};

*** zone 일 구성 정보 *** 
vi /var/named/ocp4-1.fu.te.zone

$TTL 60
@       IN SOA  dns.ocp4-1.fu.te. root.ocp4-1.fu.te. (
                                        1       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
                IN      NS      ns.ocp4-1.fu.te.;
                IN      A       10.0.10.1;
ns              IN      A       10.0.10.1;
bastion         IN      A       10.0.10.224;
;
bootstrap       IN      A       10.0.10.226;
;
master01        IN      A       10.0.10.227;
;
worker01        IN      A       10.0.10.228;
;
api             IN      A       10.0.10.224;
;
api-int         IN      A       10.0.10.224;
;
etcd-0          IN      A       10.0.10.227;
;
*.apps          IN      A       10.0.10.1;
;
_etcd-server-ssl._tcp. ocp4-1. fu.te.  86400 IN    SRV 0        10     2380 etcd-0. ocp4-1. fu.te

```

## named.rr 파일 구성
```
# vi /var/named/ocp4-1.fu.te.rr
$TTL 20
@       IN      SOA     ns.ocp4-1.fu.te.   root (
                        2019070700      ; serial
                        3H              ; refresh (3 hours)
                        30M             ; retry (30 minutes)
                        2W              ; expiry (2 weeks)
                        1W )            ; minimum (1 week)
        IN      NS      ns.ocp4-1.fu.te.
;
;
101      IN      PTR     master01.ocp4-1.fu.te.
;
10      IN      PTR     bootstrap.ocp4-1.fu.te.
;
1      IN      PTR     api.ocp4.ocp4-1.fu.te.
1      IN      PTR     api-int.ocp4-1.fu.te.
;
201      IN      PTR     worker01.ocp4-1.fu.te.

```
## named 데몬 실행 확인
```
#systemctl start named
#systemctl enable named
#systemctl status named


--------

##DSN 작업 이슈

#systemctl status named
● named.service - Berkeley Internet Name Domain (DNS)
   Loaded: loaded (/usr/lib/systemd/system/named.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since 금 2020-07-17 00:20:02 KST; 3 days ago

 7월 17 00:20:02 bastion bash[5028]: ocp4-1.fu.te.zone:27: unknown RR type 'ocp4-1.'
 7월 17 00:20:02 bastion bash[5028]: zone ocp4-1.fu.te/IN: loading from master file ocp4-1.fu.te.zone failed: unknown class/type
 7월 17 00:20:02 bastion bash[5028]: zone ocp4-1.fu.te/IN: not loaded due to errors.
 7월 17 00:20:02 bastion bash[5028]: _default/ocp4-1.fu.te/IN: unknown class/type
 7월 17 00:20:02 bastion bash[5028]: zone 10.0.10.in-addr.arpa/IN: loaded serial 2019070700
 7월 17 00:20:02 bastion systemd[1]: named.service: control process exited, code=exited status=1
 7월 17 00:20:02 bastion systemd[1]: Failed to start Berkeley Internet Name Domain (DNS).
 7월 17 00:20:02 bastion systemd[1]: Unit named.service entered failed state.
 7월 17 00:20:02 bastion systemd[1]: named.service failed.
 7월 19 03:40:02 bastion.ocp4-1.fu.te systemd[1]: Unit named.service cannot be reloaded because it is inactive.

작업 상태 확인중 "Failed to start Berkeley Internet Name Domain (DNS)."
오류 발생

```
