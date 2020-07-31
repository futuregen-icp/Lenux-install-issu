## 노드 구성 정보 

|nodename|OS|core|memory|storage|
|--------|-------|------|-------|
|Bootstrap|RHCOS|4|16 GB|120 GB|
|Control|plane	RHCOS|4|16 GB|120 GB|
|Compute|RHCOS or RHEL 7.6|2|8 GB|120 GB|

-이번 작업에서는 worke node 구성x

## IP구성정보
```
bastion 10.0.XXX.XXX , 192.168.100.204
bootstrap 192.168.100.206
master01 192.168.100.207
master02  ~.208
master03  ~.209

```
## bastion config
```
-서비스 구성 정보

bastion= 
external= Masquerad
internal= DNS,FTP,Haproxy

## ens 포트 구성 
 vm ware에서 ens192 이외에 ens224 internal port 추가구성

- resolve.conf 설정
nameserver 10.0.xxx.xxx

## /etc/host FQDN 설정

10.0.xxx.xxx bastion ocp4-3.yg.te

## nmtui config

-ens192
address 10.0.xxx.xxx/24
gatway 10.0.xxx.1
dnsserver 10.0.xxx.xxx

-ens224
addr 192.168.100.204
gatway 192.168.100.1
dnnsserver 10.0.xxx.xxx

```
## yum repo 
```
>> subscription-manager 등록X

#vi /etc/yum.repos.d/ocp4-4.repo 

[rhel-7-server-ansible-2.8-rpms]
name=rhel-7-server-rpms
baseurl=ftp:// 10.0.xxx.xxx/pub/repos/rhel-7-server-rpms
enabled=1
gpgcheck=0
[rhel-7-server-extras-rpms]
name=rhel-7-server-extras-rpms
baseurl=ftp:// 10.0.xxx.xxx/pub/repos/rhel-7-server-extras-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ansible-2.8-rpms]
name=rhel-7-server-ansible-2.8-rpms
baseurl=ftp:// 10.0.xxx.xxx/pub/repos/rhel-7-server-ansible-2.8-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ansible-2.9-rpms]
name=rhel-7-server-ansible-2.9-rpms
baseurl=ftp:// 10.0.xxx.xxx/pub/repos/rhel-7-server-ansible-2.9-rpms
enabled=1
gpgcheck=0
[rhel-7-server-ose-4.4-rpms]
name=rhel-7-server-ose-4.4-rpms
baseurl=ftp:// 10.0.xxx.xxx/pub/repos/rhel-7-server-ose-4.4-rpms
enabled=1
gpgcheck=0

- 위의 ensible 2.8 과 2.9 중 하나만 사용 사용하지 않은 패키지 는 주석 처리할것
( 주석처리 안하면 충돌로 yum 실해시 오류생김)

- 로그 오타 나지 않도록 조심 할것
```

## firewall-cmd port add
```
-external zone port add

#firewall-cmd --permanent --zone=external --add-port=80/tcp
#firewall-cmd --permanent --zone=external --add-port=443/tcp

```
```
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

#firewall-cmd –reload

- 내부 Private 망에서 사용하기에 perment --zone 을 internal 로 구성
```

## firewall 상태 확인
```
[root@bastion ~]# firewall-cmd --zone=public --list-all
public
  target: default
  icmp-block-inversion: no
  interfaces:
  sources:
  services: dhcpv6-client ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

[root@bastion ~]# firewall-cmd --zone=external --list-all
external (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens192
  sources:
  services: ssh
  ports: 80/tcp 443/tcp
  protocols:
  masquerade: yes
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

[root@bastion ~]# firewall-cmd --zone=internal --list-all
internal (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens224
  sources:
  services: dhcpv6-client dns ftp mdns samba-client ssh
  ports: 80/tcp 443/tcp 6443/tcp 8443/tcp 10256/tcp 2379-2380/tcp 9000-9999/tcp 
  10249-10259/tcp 22623/tcp 4789/udp 6081/udp 9000-9999/udp 30000-32767/udp 5000/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

-internal 에 ative 확인및 서비스,포트 확인

```
## FTP config
```
#yum install vsftpd -y   

///yum 으로 vsftp 설치

#cd /var/ftp/pub/     

/// FTP 의 기본 접속 디렉터리 확인

FTP 설치 running 작업 및 상태 확인

#systemctl enable vsftpd
#systemctl start vsftpd
#systemctl status vsftpd

## ftp 설치확인

#curl 192.168.100.204/pub/

```
## DNS 구성

```
-----------------------
bastion ip : 10.0.xxx.xxx
           : 192.168.100.204
bootstrap ip : 192.168.100.206
master01 ip : 192.168.100.207
master02 ip : 192.168.100.208
master03 ip : 192.168.100.209
worker ip : 192.168.10.213~216
FQDN : opc4-1.yg.te
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

zone "100.168.192-addr.arpa" IN {
        type master;
        file "ocp4-1.fu.te.rr";
        allow-update { none; };
};

*** zone 일 구성 정보 *** 
vi /var/named/ocp4-3.yg.te.zone

$TTL 60
@       IN SOA  dns.ocp4-1.fu.te. root.ocp4-1.fu.te. (
                                        1       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
                IN      NS      ns.ocp4-1.fu.te.;
                IN      A       192.168.100.204;
ns              IN      A       192.168.100.204;
bastion         IN      A       192.168.100.204;
;
bootstrap       IN      A       192.168.100.206;
;
master01        IN      A       192.168.100.207;
;
master02        IN      A       192.168.100.208;
;
master03        IN      A       192.168.100.209;
;
worker01        IN      A       192.168.100.213;
;
worker02         IN      A       192.168.100.214;
;
api              IN      A       192.168.100.204;
;
api-int          IN      A       192.168.100.204;
;
etcd-0           IN      A       192.168.100.207;
etcd-1           IN      A       192.168.100.208;
etcd-2           IN      A       192.168.100.209;
;
*.apps           IN      A       192.168.100.204;
;
_etcd-server-ssl._tcp.ocp4-3.yg.te.  86400 IN    SRV 0        10     2380 etcd-0.ocp4-3.yg.te
_etcd-server-ssl._tcp.ocp4-3.yg.te.  86400 IN    SRV 0        10     2380 etcd-1.ocp4-3.yg.te
_etcd-server-ssl._tcp.ocp4-3.yg.te.  86400 IN    SRV 0        10     2380 etcd-2.ocp4-3.yg.te
;

```

## named.rr 파일 구성
```
# vi /var/named/ocp4-3.yg.te.rr

$TTL 20
@       IN      SOA     ns.ocp4-3.yg.te.   root (
                        2019070700      ; serial
                        3H              ; refresh (3 hours)
                        30M             ; retry (30 minutes)
                        2W              ; expiry (2 weeks)
                        1W )            ; minimum (1 week)
        IN      NS      ns.ocp4-3.yg.te.
;
; syntax is "last octet" and the host must have fqdn with trailing dot
207      IN      PTR     master01.ocp4-3.yg.te.
208      IN      PTR     master02.ocp4-3.yg.te.
209      IN      PTR     master03.ocp4-3.yg.te.
;
206      IN      PTR     bootstrap.ocp4-3.yg.te.
;
204      IN      PTR     api.ocp4.ocp4-3.yg.te.
204      IN      PTR     api-int.ocp4-3.yg.te.
;
213      IN      PTR     worker01.ocp4-3.yg.te.
214      IN      PTR     worker02.ocp4-3.yg.te.

```
## named 데몬 실행 확인
```
#systemctl start named
#systemctl enable named
#systemctl status named
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
    server bootstrap 192.168.100.204:6443 check
    server master01 192.168.100.206:6443 check
    server master02 192.168.100.207:6443 check
    server master03 192.168.100.208:6443 check
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
    server bootstrap 192.168.100.204:6443 check
    server master01 192.168.100.206:6443 check
    server master02 192.168.100.207:6443 check
    server master03 192.168.100.208:6443 check

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
    server worker1 192.168.100.213:80 check
    server worker1 192.168.100.214:80 check
    server worker1 192.168.100.215:80 check
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
    server worker1 192.168.100.213:80 check
    server worker1 192.168.100.214:80 check
    server worker1 192.168.100.215:80 check

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

```

## bootstrap-install 
```
coreos=inst
coreos.inst.install_dev=sda 
coreos.inst.image_url=ftp://192.168.100.204/pub/rhcos4-3-3.raw.gz
coreos.inst.ignition_url=ftp://192.168.100.204/pub/bootstrap.ign
ip=192.168.100.206::192.168.100.1:ocp4-3.yg.te:ens192:none nameserver=192.168.100.204
```

## master-install
```

coreos=inst
coreos.inst.install_dev=sda 
coreos.inst.image_url=ftp://192.168.100.204/pub/rhcos4-3-3.raw.gz
coreos.inst.ignition_url=ftp://192.168.100.204/pub/master.ign
ip=192.168.100.207::192.168.100.1:ocp4-3.yg.te:ens192:none nameserver=192.168.100.204
```






