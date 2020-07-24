# Hướng dẫn cài đặt CEPH sử dụng `ceph-ansible` từ repo 

- Phiên bản cài đặt : Ceph Nautilus
- Host `ceph01` cài đặt `ceph-ansible`, `ceph-mon`,` ceph-osd`, `ceph-mgr`
- Host `ceph02` cài đặt `ceph-osd`, `grafana-server` (Monitor)
- Host `ceph03` cài đặt `ceph-osd`
- Mô hình khá cơ bản cho việc áp dụng vào môi trường Product

Chuẩn bị và môi trường LAB (3 Node)
- CentOS7.8.2003
- 03: HDD, trong đó:
    - `sda`: sử dụng để cài OS
    - `sdb`,`sdc`: sử dụng làm OSD (nơi chứa dữ liệu)
- 03 NICs: 
    - `eth0`: dùng để ssh và tải gói cài đặt
    - `eth1`: dùng để các trao đổi thông tin giữa các node Ceph, cũng là đường Client kết nối vào
    - `eth2`: dùng để đồng bộ dữ liệu giữa các OSD

IP Planning
- Phân hoạch IP cho các máy chủ trong mô hình trên
- Ceph01
    + Management: 10.0.10.55/24
    + Ceph Public: 10.0.12.55/24
    + Ceph Replicate: 10.0.13.55/24
- Ceph02
    + Management: 10.0.10.56/24
    + Ceph Public: 10.0.12.56/24
    + Ceph Replicate: 10.0.13.56/24
- Ceph03
    + Management: 10.0.10.57/24
    + Ceph Public: 10.0.12.57/24
    + Ceph Replicate: 10.0.13.57/24

## Chuẩn bị OS  (Thực hiện trên toàn bộ cả 3 node Ceph01,02,03)

- Đặt IP cho các node
```sh 
echo "Setup IP eth0"
nmcli c modify eth0 ipv4.addresses 10.0.10.55/24
nmcli c modify eth0 ipv4.gateway 10.0.10.1
nmcli c modify eth0 ipv4.dns 8.8.8.8
nmcli c modify eth0 ipv4.method manual
nmcli con mod eth0 connection.autoconnect yes

echo "Setup IP eth1"
nmcli c modify eth1 ipv4.addresses 10.0.12.55/24
nmcli c modify eth1 ipv4.method manual
nmcli con mod eth1 connection.autoconnect yes

echo "Setup IP eth2"
nmcli c modify eth2 ipv4.addresses 10.0.13.55/24
nmcli c modify eth2 ipv4.method manual
nmcli con mod eth2 connection.autoconnect yes
```
> IP 2 node còn lại tương tự 

- Cài đặt epel-relese và update OS 
```sh
yum -y install epel-release 
yum update -y
```

- Cài đặt CMD_log 
```sh 
curl -Lso- https://raw.githubusercontent.com/nhanhoadocs/scripts/master/Utilities/cmdlog.sh | bash
```

- Cấu hình timezone 
```sh 
timedatectl set-timezone Asia/Ho_Chi_Minh
```

- Đặt hostname
```sh
hostnamectl set-hostname ceph01
```

- Bổ sung user 
```sh 
sudo useradd -d /home/cephuser -m cephuser
sudo passwd cephuser
```

- Phân quyền cho user 
```sh 
echo "cephuser ALL = (root) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/cephuser
sudo chmod 0440 /etc/sudoers.d/cephuser
firewall-cmd --add-service=ssh --permanent
firewall-cmd --reload 
```

## Ceph-ansible prepare  (Thực hiện trên node Ceph01)

-  Cài đặt ceph-ansible 
```sh 
yum -y install epel-release centos-release-ceph-nautilus
yum -y install ceph-ansible 
```

- Tạo ssh-keygen 
```sh 
ssh-keygen -t rsa -N "" -f ~/.ssh/id_rsa
```

- Tạo ssh config 
```sh 
cat <<EOF > ~/.ssh/config 
Host 10.0.10.55
   Hostname 10.0.10.55
   User cephuser
Host 10.0.10.56
   Hostname 10.0.10.56
   User cephuser
Host 10.0.10.57
   Hostname 10.0.10.57
   User cephuser
EOF
```

- Copy ssh-key 
```sh 
ssh-copy-id 10.0.10.55
ssh-copy-id 10.0.10.56
ssh-copy-id 10.0.10.57
```

Kiểm tra 
```sh 
cd /usr/share/ceph-ansible
ansible -m ping -i inventory_hosts all 
```

Kêt quả 
```sh 
[root@ceph01 ceph-ansible]#  ansible -m ping -i inventory_hosts all
[WARNING]: log file at /root/ansible/ansible.log is not writeable and we cannot create it, aborting

[WARNING]: Invalid characters were found in group names but not replaced, use -vvvv to see details
10.0.10.56 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
10.0.10.57 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
10.0.10.55 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "ping": "pong"
}
[root@ceph01 ceph-ansible]# 
```

## Install ceph (Thực hiện trên node Ceph01)

> Dưới đây thực hiện cài đặt kiểu `Bare-metal`

Copy file config mẫu 
```sh 
cd /usr/share/ceph-ansible/
cp group_vars/all.yml.sample group_vars/all.yml
cp group_vars/osds.yml.sample group_vars/osds.yml
cp site.yml.sample site.yml
```

Inventory 
```sh 
cat <<EOF > /usr/share/ceph-ansible/inventory_hosts
[mons]
10.0.10.55
10.0.10.56
10.0.10.57

[osds]
10.0.10.55
10.0.10.56
10.0.10.57

[mgrs]
10.0.10.55

[grafana-server]
10.0.10.56
EOF
```

Tạo `all.yml`
```sh
cat <<EOF > group_vars/all.yml
## General
## ------- Cluster
---
dummy:

fetch_directory: fetch/
cluster: ceph

mon_group_name: mons
osd_group_name: osds
rgw_group_name: rgws
mds_group_name: mdss
nfs_group_name: nfss
rbdmirror_group_name: rbdmirrors
client_group_name: clients
iscsi_gw_group_name: iscsigws
mgr_group_name: mgrs
rgwloadbalancer_group_name: rgwloadbalancers
grafana_server_group_name: grafana-server

## ------- Firewalld
configure_firewall: True

ceph_mon_firewall_zone: public
ceph_mgr_firewall_zone: public
ceph_osd_firewall_zone: public
ceph_rgw_firewall_zone: public
ceph_mds_firewall_zone: public
ceph_nfs_firewall_zone: public
ceph_rbdmirror_firewall_zone: public
ceph_iscsi_firewall_zone: public
ceph_dashboard_firewall_zone: public
ceph_rgwloadbalancer_firewall_zone: public

## Packages

## Install
ceph_repository_type: cdn
ceph_origin: repository
ceph_repository: community
ceph_stable_release: nautilus
monitor_interface: eth1
public_network: 10.0.12.0/24
cluster_network: 10.0.13.0/24
ip_version: ipv4
generate_fsid: true
cephx: true

## Config override
ceph_conf_overrides:
  global:
    osd_pool_default_pg_num: 8
    osd_pool_default_size: 2
    osd_pool_default_min_size: 1

## CephFS

## NFS-Ganesha

## CephRGW

## Multisite -RGW

## OS turning

## Docker

## Openstack

## Dashboard - Grafana
dashboard_enabled: True
dashboard_protocol: http
dashboard_port: 8443
dashboard_admin_user: admin
dashboard_admin_password: Cas@2020
grafana_admin_user: admin
grafana_admin_password: Cas@2020


## iSCSI
EOF
```

`group_vars/osds.yml`
```yml
cat << EOF> group_vars/osds.yml
osd_auto_discovery: true
osd_auto_discovery_exclude: "dm-*|loop*|md*|rbd*"
EOF
```

Cài đặt byobu 
```sh 
yum install byobu -y 
```

Bật session byobu 
```sh 
byobu
```

Deploy ceph
```sh 
cd /usr/share/ceph-ansible
ansible-playbook site.yml -i inventory_hosts
```

Log install OK
```sh 
...
INSTALLER STATUS ****************************************************************************************************
Install Ceph Manager           : Complete (0:00:24)
Install Ceph OSD               : Complete (0:00:41)
Install Ceph Dashboard         : Complete (0:00:33)
Install Ceph Grafana           : Complete (0:00:31)
Install Ceph Node Exporter     : Complete (0:00:26)

Monday 22 June 2020  15:00:23 +0700 (0:00:00.045)       0:03:49.725 *********** 
=============================================================================== 
ceph-grafana : wait for grafana to start ------------------------------------------------ 5.22s
ceph-dashboard : set or update dashboard admin username and password -------------------- 4.25s
ceph-common : configure red hat ceph community repository stable key -------------------- 2.86s
ceph-facts : check for a ceph mon socket ------------------------------------------------ 2.74s
gather and delegate facts --------------------------------------------------------------- 2.55s
ceph-config : look up for ceph-volume rejected devices ---------------------------------- 1.76s
ceph-osd : systemd start osd ------------------------------------------------------------ 1.67s
ceph-facts : check if the ceph mon socket is in-use ------------------------------------- 1.65s
ceph-config : look up for ceph-volume rejected devices ---------------------------------- 1.61s
ceph-osd : apply operating system tuning ------------------------------------------------ 1.60s
ceph-facts : check for a ceph mon socket ------------------------------------------------ 1.56s
ceph-config : look up for ceph-volume rejected devices ---------------------------------- 1.54s
ceph-dashboard : disable mgr dashboard module (restart) --------------------------------- 1.33s
ceph-handler : check if the ceph mon socket is in-use ----------------------------------- 1.30s
ceph-facts : get default crush rule value from ceph configuration ----------------------- 1.30s
ceph-handler : check if the ceph osd socket is in-use ----------------------------------- 1.26s
ceph-handler : check for a ceph mon socket ---------------------------------------------- 1.25s
ceph-mgr : copy ceph key(s) if needed --------------------------------------------------- 1.23s
ceph-config : generate ceph configuration file: ceph.conf ------------------------------- 1.19s
ceph-config : generate ceph configuration file: ceph.conf ------------------------------- 1.16s
(venv) [root@ceph01 ceph-ansible]# 
[B]   0:-*                                                                                                      
```

Kiểm tra ceph
```sh 
[root@ceph01 ~]# ceph -v
ceph version 14.2.9 (581f22da52345dba46ee232b73b990f06029a2a0) nautilus (stable)
[root@ceph01 ~]# ceph -s
  cluster:
    id:     9889fc17-5f1e-43c4-a631-99009e12dfe2
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum ceph01,ceph02,ceph03 (age 5h)
    mgr: ceph01(active, since 5h)
    osd: 6 osds: 6 up (since 5h), 6 in (since 5h)

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   6.0 GiB used, 294 GiB / 300 GiB avail
    pgs:

[root@ceph01 ~]# 
```

## Install ceph (Thực hiện trên node Ceph01)

> Dưới đây thực hiện cài đặt kiểu `Containner`

Copy file config mẫu 
```sh 
cd /usr/share/ceph-ansible/
cp group_vars/all.yml.sample group_vars/all.yml
cp group_vars/osds.yml.sample group_vars/osds.yml
cp site-container.yml.sample site-container.yml
```

Inventory 
```sh 
cat <<EOF > /usr/share/ceph-ansible/inventory_hosts
[mons]
10.0.10.55
10.0.10.56
10.0.10.57

[osds]
10.0.10.55
10.0.10.56
10.0.10.57

[mgrs]
10.0.10.55

[grafana-server]
10.0.10.56
EOF
```

Tạo `all.yml`
```sh
cat <<EOF > group_vars/all.yml
## General
## ------- Cluster
---
dummy:

fetch_directory: fetch/
cluster: ceph

mon_group_name: mons
osd_group_name: osds
rgw_group_name: rgws
mds_group_name: mdss
nfs_group_name: nfss
rbdmirror_group_name: rbdmirrors
client_group_name: clients
iscsi_gw_group_name: iscsigws
mgr_group_name: mgrs
rgwloadbalancer_group_name: rgwloadbalancers
grafana_server_group_name: grafana-server

## ------- Firewalld
configure_firewall: True

ceph_mon_firewall_zone: public
ceph_mgr_firewall_zone: public
ceph_osd_firewall_zone: public
ceph_rgw_firewall_zone: public
ceph_mds_firewall_zone: public
ceph_nfs_firewall_zone: public
ceph_rbdmirror_firewall_zone: public
ceph_iscsi_firewall_zone: public
ceph_dashboard_firewall_zone: public
ceph_rgwloadbalancer_firewall_zone: public

## Packages

## Install
ceph_repository_type: cdn
ceph_origin: repository
ceph_repository: community
ceph_stable_release: nautilus
monitor_interface: eth1
public_network: 10.0.12.0/24
cluster_network: 10.0.13.0/24
ip_version: ipv4
generate_fsid: true
cephx: true

## Config override
ceph_conf_overrides:
  global:
    osd_pool_default_pg_num: 8
    osd_pool_default_size: 2
    osd_pool_default_min_size: 1

## CephFS

## NFS-Ganesha

## CephRGW

## Multisite -RGW

## OS turning

## Docker
containerized_deployment: True
#ceph_docker_image: "ceph/daemon"
#ceph_docker_image_tag: latest-nautilus
#ceph_docker_registry: docker.io
#ceph_docker_registry_auth: false
#ceph_docker_registry_username:
#ceph_docker_registry_password:
#ceph_docker_enable_centos_extra_repo: false
#timeout_command: "{{ 'timeout --foreground -s KILL ' ~ docker_pull_timeout if (docker_pull_timeout != '0') and (ceph_docker_dev_image is undefined or not ceph_docker_dev_image) else '' }}"

#rolling_update: false
#docker_pull_timeout: "290s"

## Openstack

## Dashboard - Grafana
dashboard_enabled: True
dashboard_protocol: http
dashboard_port: 8443
dashboard_admin_user: admin
dashboard_admin_password: Cas@2020
grafana_admin_user: admin
grafana_admin_password: Cas@2020


## iSCSI
EOF
```

`group_vars/osds.yml`
```yml
cat << EOF> group_vars/osds.yml
osd_auto_discovery: true
osd_auto_discovery_exclude: "dm-*|loop*|md*|rbd*"
EOF
```

Cài đặt byobu 
```sh 
yum install byobu -y 
```

Bật session byobu 
```sh 
byobu
```

Deploy ceph
```sh 
cd /usr/share/ceph-ansible
ansible-playbook site-container.yml -i inventory_hosts
```

Cài đặt hoàn tất 
```sh 
...
INSTALLER STATUS ***************************************************************************
Install Ceph Monitor           : Complete (0:00:51)
Install Ceph Manager           : Complete (0:00:33)
Install Ceph OSD               : Complete (0:00:58)
Install Ceph Dashboard         : Complete (0:00:33)
Install Ceph Grafana           : Complete (0:00:37)
Install Ceph Node Exporter     : Complete (0:00:23)

Friday 24 July 2020  16:48:13 +0700 (0:00:00.044)       0:06:15.859 *********** 
=============================================================================== 
ceph-container-common : pulling docker.io/ceph/daemon:latest-nautilus image -------- 28.61s
ceph-container-engine : install container packages --------------------------------- 25.33s
ceph-infra : install firewalld python binding -------------------------------------- 20.41s
ceph-grafana : wait for grafana to start ------------------------------------------- 14.23s
ceph-mon : waiting for the monitor(s) to form the quorum... ------------------------ 13.70s
ceph-osd : wait for all osd to be up ----------------------------------------------- 11.76s
get ceph status from the first monitor --------------------------------------------- 10.82s
ceph-osd : use ceph-volume lvm batch to create bluestore osds ----------------------- 8.64s
ceph-mgr : wait for all mgr to be up ------------------------------------------------ 7.00s
ceph-dashboard : set or update dashboard admin username and password ---------------- 6.16s
ceph-mon : fetch ceph initial keys -------------------------------------------------- 5.03s
ceph-config : create ceph initial directories --------------------------------------- 3.71s
gather and delegate facts ----------------------------------------------------------- 3.69s
ceph-config : create ceph initial directories --------------------------------------- 3.67s
ceph-config : create ceph initial directories --------------------------------------- 3.54s
ceph-container-engine : start container service ------------------------------------- 2.53s
ceph-osd : apply operating system tuning -------------------------------------------- 2.26s
ceph-container-common : get ceph version -------------------------------------------- 2.23s
ceph-mgr : disable ceph mgr enabled modules ----------------------------------------- 2.18s
add modules to ceph-mgr ------------------------------------------------------------- 2.13s
[root@ceph01 ceph-ansible]#
```

Kiểm tra tình trạng service 
```sh 
[root@ceph01 ceph-ansible]# docker ps 
CONTAINER ID        IMAGE                                   COMMAND                  CREATED              STATUS              PORTS               NAMES
e2243ace58dc        prom/node-exporter:v0.17.0              "/bin/node_exporte..."   About a minute ago   Up About a minute                       node-exporter
d6b56e814259        docker.io/ceph/daemon:latest-nautilus   "/opt/ceph-contain..."   2 minutes ago        Up 2 minutes                            ceph-osd-5
d47539ae1882        docker.io/ceph/daemon:latest-nautilus   "/opt/ceph-contain..."   2 minutes ago        Up 2 minutes                            ceph-osd-2
7fd1a1a9327e        docker.io/ceph/daemon:latest-nautilus   "/opt/ceph-contain..."   3 minutes ago        Up 3 minutes                            ceph-mgr-ceph01
fc7ad11ee406        docker.io/ceph/daemon:latest-nautilus   "/opt/ceph-contain..."   4 minutes ago        Up 4 minutes                            ceph-mon-ceph01
[root@ceph01 ceph-ansible]# docker exec fc7ad11ee406 ceph -s 
  cluster:
    id:     20ac6e99-0dd4-4310-b1c6-c83f8d05869e
    health: HEALTH_OK
 
  services:
    mon: 3 daemons, quorum ceph01,ceph02,ceph03 (age 4m)
    mgr: ceph01(active, since 56s)
    osd: 6 osds: 6 up (since 2m), 6 in (since 2m)
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   6.0 GiB used, 294 GiB / 300 GiB avail
    pgs:     
 
[root@ceph01 ceph-ansible]# docker exec fc7ad11ee406 ceph osd tree 
ID CLASS WEIGHT  TYPE NAME       STATUS REWEIGHT PRI-AFF 
-1       0.29279 root default                            
-7       0.09760     host ceph01                         
 2   hdd 0.04880         osd.2       up  1.00000 1.00000 
 5   hdd 0.04880         osd.5       up  1.00000 1.00000 
-5       0.09760     host ceph02                         
 1   hdd 0.04880         osd.1       up  1.00000 1.00000 
 3   hdd 0.04880         osd.3       up  1.00000 1.00000 
-3       0.09760     host ceph03                         
 0   hdd 0.04880         osd.0       up  1.00000 1.00000 
 4   hdd 0.04880         osd.4       up  1.00000 1.00000 
[root@ceph01 ceph-ansible]# 
```

## Appendix(Pending): Network cho Grafana và CephDashboard binding vào dải CephPublic 

![](https://i.imgur.com/BgnGvx0.png)

Mong muốn: Grafana và Ceph Dashboard binding vào dải Mngt 


# Tài liệu tham khảo 

- https://docs.ceph.com/ceph-ansible/master/
- https://www.marksei.com/how-to-install-ceph-with-ceph-ansible/ (*)
- https://kruschecompany.com/ceph-ansible/
- https://access.redhat.com/documentation/en-us/red_hat_ceph_storage/4/html-single/installation_guide/index#prerequisites_3