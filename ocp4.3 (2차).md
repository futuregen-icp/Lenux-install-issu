## 노드 구성 정보 
```
Bootstrap	 RHCOS	            4	 16 GB	120 GB
Control    plane	RHCOS	      4	 16 GB	120 GB
Compute	   RHCOS or RHEL 7.6	2	 8 GB	  120 GB

-이번 작업에서는 worke node 구성x
```
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




