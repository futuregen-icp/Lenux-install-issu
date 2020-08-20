## ocp3.11 install

|nodename|OS|core|memory|storage|
|:--:|:--:|:--:|:--:|:--:|
|bastion|RHEL 7.6|8|16 GB|150 GB|
|infra|RHEL 7.6|8|16 GB|150 GB|
|Controlplane|RHEL 7.6|8|16 GB|120 GB|
|Compute|RHEL 7.6|8|16 GB|120 GB|



## FQDN setting
```
FQDN
ocp11.yg.te

```
## networksetting
```
#nmtui
ipaddr:x.x.x.224/24
gatway:x.x.x.1

```
## subscription-manager 
```
subscription-manager register --username=***** --password=*****
subscription-manager refresh
subscription-manager list --available --matches '*OpenShift*'
subscription-manager attach --pool=8a85f9997385090b0173b342847c50b5
subscription-manager repos     --enable="rhel-7-server-rpms"     \
                               --enable="rhel-7-server-extras-rpms"     \
                               --enable="rhel-7-server-ose-3.11-rpms"     \
                               --enable="rhel-7-server-ansible-2.9-rpms"
```


## 필수 패키지 설치
```
#yum -y install yum-utils createrepo docker git vsftpd
```

## firewall setting
```

firewall-cmd --permanent --zone=external --add-service=ftp
firewall-cmd --permanent --zone=external --add-service=dns
firewall-cmd --permanent --zone=external --add-service=tftp
firewall-cmd --permanent --zone=external --add-service=ssh
firewall-cmd --permanent --zone=external --add-service=dhcp
firewall-cmd --permanent --zone=external --add-service=proxy-dhcp
firewall-cmd --permanent --zone=external --add-service=http
firewall-cmd --permanent --zone=external --add-service=https
firewall-cmd --permanent --zone=external --add-port=5000/tcp

firewall-cmd --reload

firewall default change
---------------------------
firewall-cmd  --set-default-zone=external
firewall-cmd --zone=external --change-interface=eth0
```

## DSN install and Setting
```
yum  install bind
DNS 설정
# vi /etc/named/named.conf
options {
        listen-on port 53 { ANY; };
        listen-on-v6 port 53 { ANY; };
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        recursing-file  "/var/named/data/named.recursing";
        secroots-file   "/var/named/data/named.secroots";
        allow-query     { ANY; };

# vi /etc/named.rfc1912.zones
zone "ocp11.yg.te" IN {
        type master;
        file "ocp11.yg.te.zone";
        allow-update { none; };
};

zone "10.0.10.in-addr.arpa" IN {
        type master;
        file "ocp11.yg.te.rr";
        allow-update { none; };
};

# vi /var/named/ocp11.yg.te.zone
$TTL 60
@       IN SOA  dns.ocp11.yg.te. root.ocp11.yg.te. (
                                        1       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
                IN      NS      ns.ocp11.yg.te.;
                IN      A       10.0.10.224;
ns              IN      A       10.0.10.224;
bastion         IN      A       10.0.10.224;
;
master01        IN      A       10.0.10.227;
;
infra01         IN      A       10.0.10.226;
;
worker01        IN      A       10.0.10.228;
;
*.apps          IN      A       10.0.10.224;
;

# vi /var/named/ocp11.yg.te.rr
$TTL 20
@       IN      SOA     ns.ocp11.yg.te.   root (
                        2019070700      ; serial
                        3H              ; refresh (3 hours)
                        30M             ; retry (30 minutes)
                        2W              ; expiry (2 weeks)
                        1W )            ; minimum (1 week)
        IN      NS      ns.ocp11.yg.te.
;
; syntax is "last octet" and the host must have fqdn with trailing dot
227      IN      PTR     master01.ocp11.yg.te.
;
226      IN      PTR     infra01.ocp11.yg.te.
228      IN      PTR     worker01.ocp11.yg.te.


```

## install all on nodes
```
# yum -y install wget git net-tools bind-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct
# yum -y update
# reboot
-
# yum -y install openshift-ansible


#yum -y install cri-o

#yum -y install docker
```


## ssh key 생성 
```
#(root) ssh-keygen

 노드 ssh key 정보 저장

ssh-copy-id -i '/root/.ssh/id_rsa.pub' master.ocp11.yg.te
ssh-copy-id -i '/root/.ssh/id_rsa.pub' infra.ocp11.yg.te
ssh-copy-id -i '/root/.ssh/id_rsa.pub' worker.ocp11.yg.te
ssh-copy-id -i '/root/.ssh/id_rsa.pub' bastion.ocp11.yg.te

```

## ans file config
```
 cat /etc/ans
[master]
x.x.x.227
[infra]
x.x.x.226
[node]
x.x.x.228
[bastion]
x.x.x.224
[OCP]
x.x.x.224
x.x.x.226
x.x.x.227
x.x.x.228
```
## host config
```
###########################################################################
### Global cluster
###########################################################################
[OSEv3:vars]
##-------------------------------------------------------------------------
## Ansible
##-------------------------------------------------------------------------
ansible_user=root
ansible_become=yes
debug_level=2

###########################################################################
### OpenShift Basic Vars
###########################################################################
openshift_deployment_type=openshift-enterprise
openshift_release=v3.11.188
openshift_image_tag=v3.11.188
openshift_pkg_version=-3.11.188

# Configuring Certificate Validity
openshift_hosted_registry_cert_expire_days=3650
openshift_ca_cert_expire_days=3650
openshift_node_cert_expire_days=3650
openshift_master_cert_expire_days=3650
etcd_ca_default_days=3650
openshift_certificate_expiry_warning_days=2920
openshift_certificate_expiry_fail_on_warn=2920

# Configuring Cluster Pre-install Checks
openshift_disable_check=memory_availability,disk_availability,docker_storage,docker_storage_driver,docker_image_availability,package_version,package_availability,package_update

# Node Groups
openshift_node_groups=[{'name': 'node-config-master', 'labels': ['node-role.kubernetes.io/master=true']}, {'name': 'node-config-infra', 'labels': ['node-role.kubernetes.io/infra=true']}, {'name': 'node-config-monitoring', 'labels': ['node-role.kubernetes.io/monitoring=true']}, {'name': 'node-config-compute', 'labels': ['node-role.kubernetes.io/compute=true'], 'edits': [{ 'key': 'kubeletArguments.pods-per-core','value': ['20']}]}]

# Configure logrotate scripts
logrotate_scripts=[{"name": "syslog", "path": "/var/log/cron\n/var/log/maillog\n/var/log/messages\n/var/log/secure\n/var/log/spooler\n", "options": ["daily", "rotate 7","size 500M", "compress", "sharedscripts", "missingok"], "scripts": {"postrotate": "/bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true"}}]


###########################################################################
### OpenShift Master Vars
###########################################################################
openshift_master_api_port=8443
openshift_master_console_port=8443
openshift_master_cluster_hostname=master01.ocp11.yg.te
openshift_master_cluster_public_hostname=master01.ocp11.yg.te
# openshift_master_named_certificates=[{"certfile": "/home/ebaycloud/named_certificates/STAR.ebaykorea.com/STAR_ebaykorea_com.crt", "keyfile": "/home/ebaycloud/named_certificates/STAR.ebaykorea.com/STAR_ebaykorea_com.key", "names": ["fusiona1.ebaykorea.com"], "cafile": "/home/ebaycloud/named_certificates/STAR.ebaykorea.com/STAR_ebaykorea_com.ca-bundle"}]
openshift_master_default_subdomain=apps.ocp11.yg.te
# openshift_master_overwrite_named_certificates=true
openshift_master_cluster_method=native

###########################################################################
### OpenShift Network Vars
###########################################################################
openshift_portal_net=172.40.0.0/16
osm_cluster_network_cidr=10.140.0.0/14
os_sdn_network_plugin_name='redhat/openshift-ovs-subnet'
#os_sdn_network_plugin_name='redhat/openshift-ovs-multitenant'

###########################################################################
### OpenShift Authentication Vars
###########################################################################
# LDAP AND HTPASSWD Authentication (download ipa-ca.crt first)
#openshift_master_identity_providers=[{'name': 'ldap', 'challenge': 'true', 'login': 'true', 'kind': 'LDAPPasswordIdentityProvider','attributes': {'id': ['dn'], 'email': ['mail'], 'name': ['cn'], 'preferredUsername': ['uid']}, 'bindDN': 'uid=admin,cn=users,cn=accounts,dc=shared,dc=example,dc=opentlc,dc=com', 'bindPassword': 'r3dh4t1!', 'ca': '/etc/origin/master/ipa-ca.crt','insecure': 'false', 'url': 'ldaps://ipa.shared.example.opentlc.com:636/cn=users,cn=accounts,dc=shared,dc=example,dc=opentlc,dc=com?uid?sub?(memberOf=cn=ocp-users,cn=groups,cn=accounts,dc=shared,dc=example,dc=opentlc,dc=com)'},{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]

# Just LDAP
#openshift_master_identity_providers=[{'name': 'ldap_provider', 'challenge': 'true', 'login': 'true', 'kind': 'LDAPPasswordIdentityProvider','attributes': {'id': ['sAMAccountName'], 'email': ['mail'], 'name': ['name'], 'preferredUsername': ['sAMAccountName']}, 'bindDN': 'fusion@ebaykorea.corp', 'bindPassword': '', 'insecure': 'true', 'url': 'ldap://ebaykorea.corp/dc=ebaykorea,dc=corp?sAMAccountName?sub?(&(objectClass=user)(objectCategory=person))'}]

# Just HTPASSWD
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]

# LDAP and HTPASSWD dependencies
#Command => htpasswd -c /root/htpasswd.openshift ocpadmin
#openshift_master_htpasswd_file=/home/ebaycloud/htpasswd.openshift
#openshift_master_ldap_ca_file=/root/ipa-ca.crt

###########################################################################
### OpenShift Service Catalog Vars
###########################################################################
#openshift_enable_service_catalog=true
#template_service_broker_install=true
#openshift_template_service_broker_namespaces=['openshift']
#ansible_service_broker_install=true
#ansible_service_broker_local_registry_whitelist=['.*-apb$']

openshift_enable_service_catalog=false
template_service_broker_install=false
ansible_service_broker_install=false

#template_service_broker_remove=true
#ansible_service_broker_install=true
#openshift_service_catalog_remove=true

###########################################################################
### OpenShift Hosts
###########################################################################
[OSEv3:children]
masters
nodes

[masters]
master.ocp11.yg.te openshift_ip=x.x.x.227 openshift_public_ip=x.x.x.227 openshift_public_hostname=master.ocp11.yg.te

[nodes]
## Master
master.ocp11.yg.te openshift_ip=x.x.x.227 openshift_public_ip=x.x.x.227 openshift_public_hostname=master.ocp11.yg.te openshift_node_group_name='node-config-master'

### Infra
infra.ocp11.yg.te openshift_ip=x.x.x.226 openshift_public_ip=x.x.x.226 openshift_public_hostname=infra.ocp11.yg.te openshift_node_group_name='node-config-infra'

### Node
worker.ocp11.yg.te openshift_ip=x.x.x.228 openshift_public_ip=x.x.x.228 openshift_public_hostname=worker.ocp11.yg.te openshift_node_group_name='node-config-compute'

```

## 사전점검
```
# cd /usr/share/ansible/openshift-ansible
# ansible-playbook -i /opt/ocp11/hosts playbooks/prerequisites.yml
```

## playbook 실행 
```
# cd /usr/share/ansible/openshift-ansible
# ansible-playbook -i /opt/ocp11/hosts playbooks/pre-install.yml
```

## copy the master node's KUBE/config file to bastion
```
@master# scp /root/.kube/config1 root@x.x.x.224 /opt/ocp11

