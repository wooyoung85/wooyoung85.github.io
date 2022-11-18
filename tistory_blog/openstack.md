```bash
sudo passwd root

sudo su

apt update
apt upgrade

apt install vim iputils-ping net-tools -y
```

```bash
vim /etc/hosts
192.168.56.5 controller


service network-manager restart
```

```bash
vim /etc/netplan/01-network-manager-all.yaml
netplan apply
```

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0s3:
      dhcp4: no
      addresses: [192.168.56.5/24]
      routes:
        - to: 192.168.56.0/24
          via: 192.168.56.2
          table: 101
      routing-policy:
        - from: 192.168.56.0/24
          table: 101
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
    eth0s8:
      dhcp4: no
      addresses: [192.168.0.81/24]
      routes:
        - to: default
          via: 192.168.0.1
        - to: 192.168.0.0/24
          via: 192.168.0.1
          table: 102
      routing-policy:
        - from: 192.168.0.0/24
          table: 101
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
```

```bash
apt install chrony ntpdate -y
timedatectl
ntpdate time.bora.net
timedatectl
timedatectl set-timezone Asia/Seoul
timedatectl

vim /etc/chrony/chrony.conf
service chrony restart
chronyc sources
```

```bash
...

# About using servers from the NTP Pool Project in general see (LP: #104525).
# Approved by Ubuntu Technical Board on 2011-02-08.
# See http://www.pool.ntp.org/join.html for more information.
# pool ntp.ubuntu.com        iburst maxsources 4
# pool 0.ubuntu.pool.ntp.org iburst maxsources 1
# pool 1.ubuntu.pool.ntp.org iburst maxsources 1
# pool 2.ubuntu.pool.ntp.org iburst maxsources 2
server 2.kr.pool.ntp.org     iburst
server 1.asia.pool.ntp.org   iburst
server 3.asia.pool.ntp.org   iburst

allow 192.168.56.0/24

...
```

```bash
apt install python3-openstackclient -y
apt install mariadb-server python3-pymysql -y
vim /etc/mysql/mariadb.conf.d/99-openstack.cnf
```

```bash
[mysqld]
bind-address = 192.168.56.5
default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```

```bash
service mysql restart
mysql_secure_installation

apt install rabbitmq-server -y
rabbitmqctl add_user openstack 1
rabbitmqctl set_permissions openstack ".*" ".*" ".*"

apt install memcached python3-memcache -y
vim /etc/memcached.conf
127.0.0.1 --> 192.168.56.5
service memcached restart
```

```bash
apt install etcd
vim /etc/default/etcd

systemctl enable etcd
systemctl restart etcd
```

```
ETCD_NAME="controller"
ETCD_DATA_DIR="/var/lib/etcd"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster-01"
ETCD_INITIAL_CLUSTER="controller=http://controller:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="http://controller:2380"
ETCD_ADVERTISE_CLIENT_URLS="http://192.168.56.5:2379"
ETCD_LISTEN_PEER_URLS="http://192.168.56.5:2380"
ETCD_LISTEN_CLIENT_URLS="http://192.168.56.5:2379"
```

```bash
mysql -u root

create database keystone;
grant all privileges on keystone.* to 'keystone'@'localhost' identified by '1';
grant all privileges on keystone.* to 'keystone'@'%' identified by '1';
```

```bash
apt install keystone -y
---------&&&&&&&&&&&&&
vim /etc/keystone/keystone.conf
```

```bash
su -s /bin/sh -c "keystone-manage db_sync" keystone
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
keystone-manage credential_setup --keystone-user keystone --keystone-group keystone

keystone-manage bootstrap --bootstrap-password 1 \
--bootstrap-admin-url http://controller:5000/v3/ \
--bootstrap-internal-url http://controller:5000/v3/ \
--bootstrap-public-url http://controller:5000/v3/ \
--bootstrap-region-id RegionOne
```
