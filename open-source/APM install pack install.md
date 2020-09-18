## APM 설치 정보 
```
###Mariadb info

mysql -V
mysql  Ver 15.1 Distrib 10.4.14-MariaDB, for Linux (x86_64) using readline 5.1

###apache info

httpd -v
Server version: Apache/2.4.6 (Red Hat Enterprise Linux)
```

##yum version update 
```
 yum update 
```
##필요 라이브러리 설치 
```
yum install libjpeg* libpng* freetype* gd-* gcc gcc-c++ gdbm-devel libtermcap-devel

###필요 도구 설치
 yum -y install net-tool psmisc 

APM install 

Apache - mariadb - php 순서로 진행

yum inatall httpd -y
```

##MariaDB 설치
```
10버전의 MariaDB를 설치하기 위해서는 repo 설정을 먼저 진행해야 한다.

repo 설정
먼저 MariaDB.repo 라는 파일을 만들고 아래처럼 내용을 입력

vi /etc/yum.repos.d/MariaDB.repo
# MariaDB 10.4 CentOS repository list - created 2020-06-05 23:35 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.4/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

##MariaDB 설치
```
yum install MariaDB-server MariaDB-client

mysql -v
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)

위의 에러 발생시 
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
```


##설치 파일 확인 
rpm -qa httpd mysql php

Yum으로 설치할경우 httpd 기본환경설정 경로는

 vi /etc/httpd/conf/httpd.conf

 User apache → nobody 변경
 Group apache → nobody 변경


root 권한으로 실행된 아파치의 하위 프로세스를 이곳에서 지정한 사용자로 실행한다는 의미
기본값으로 apache 혹은 daemon 으로 되어있지만 대부분 nobody 변경하여 사용합니다.



 ServerName [ 서버의 IP입력 ]
 ex) ServerName [ 192.168.119.132:80 ]
```

##firewalld 설정 
```
firewall-cmd --permanent --zone=public --add-port=80/tcp
firewall-cmd --permanent --zone=public --add-port=3306/tcp
firewall-cmd --permanent --zone=public --add-port=443/tcp     
firewall-cmd --permanent --zone=public --add-port=138/tcp
firewall-cmd --permanent --zone=public --add-port=139/tcp
firewall-cmd --reload
```

##hpptd 실행 
```
systemctl start httpd
systemctl enable httpd
```

##체크

###port LISTEN 확인
```
netstat -nap | grep LISTEN

tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1150/sshd
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1713/master
tcp6       0      0 :::80                   :::*                    LISTEN      9559/httpd
tcp6       0      0 :::22                   :::*                    LISTEN      1150/sshd
tcp6       0      0 ::1:25                  :::*                    LISTEN      1713/master
tcp6       0      0 :::3306                 :::*                    LISTEN      10661/mysqld

###방화벽 확인  
firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens192
  sources:
  services: dhcpv6-client ssh
  ports: 80/tcp 443/tcp 3306/tcp 50001/tcp 138/tcp 139/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

###web browser 접속 확인 
-apache 접속 확인 
 http://server IP:80     >>> 접속
```


 
##PHP 설정 
```
vi /etc/httpd/conf/httpd.conf
--------------
 DirectoryIndex: sets the file that Apache will serve if a directory
# is requested.
#
<IfModule dir_module>
    DirectoryIndex index.html index.htm index.php >>>>[index.htm index.php(추가)]
</IfModule>
--------------
                   :
                   :
                   :
--------------                   
# If the AddEncoding directives above are commented-out, then you
# probably should define those extensions to indicate media types:
#
  AddType application/x-compress .Z
  AddType application/x-gzip .gz .tgz
  AddType application/x-httpd-php .php .html .htm .inc    >>>추가
  AddType application/X-hrttpd-php-source .phps            >>>추가
--------------

###테스트 페이지 작성

html 업로드 기본 디렉토리는 /var/www/html/ 

 vi /var/www/html/phpinfo.php
--------------
 <?php
  phpinfo(); 
  ?> 추가 

  (PHP 의 정보를 보여주는 함수)
--------------
 ```

###apache 재시작
 ```
 -------------
 systmectl restart httpd
--------------

  만약 httpd.con 가 제대로 입력되지 않았다면
 에러메세지가 뜰것
```

###에러 발생시
```
find / -name php.ini  경로 확인

php.ini 접속

short_open_tag = Off   를
>>>>
short_open_tag = On  으로 변경

pstree 로 httpd 상태 확인

[root@localhost ~]# pstree
systemd─┬─NetworkManager───2*[{NetworkManager}]
        ├─VGAuthService
        ├─agetty
        ├─auditd───{auditd}
        ├─chronyd
        ├─crond
        ├─dbus-daemon───{dbus-daemon}
        ├─firewalld───{firewalld}
        ├─httpd───8*[httpd]
        ├─irqbalance
        ├─lvmetad
        ├─master─┬─pickup
        │        └─qmgr
        ├─polkitd───6*[{polkitd}]
        ├─rhnsd
        ├─rhsmcertd
        ├─rsyslogd───2*[{rsyslogd}]
        ├─sshd───sshd───bash───pstree
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-udevd
        ├─tuned───4*[{tuned}]
        └─vmtoolsd

web	browser 접속 
http:// server IP/phpinfo.php 

php information  화면이 나오면 성공
```

##mysql 확인 
```
mywql -p root -u

접속 확인 password 치고 접속

show databases 확인
--------------
[root@localhost ~]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 12
Server version: 10.4.14-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| test               |
+--------------------+
4 rows in set (0.033 sec)

MariaDB [(none)]> exit
--------------
```






