## Yêu cầu 

[Đã cài đặt Ceph-Ansible RPMsource](ceph-ansible-nautilus.md)

hoặc 

[Đã cài đặt Ceph-Ansible Gitsource](ceph-ansible-nautilus-2.md)

## Cài đặt RGW 

Di chuyển đến thư mục `ceph-ansible`
```sh 
cd /usr/share/ceph-ansible
```

Cấu hình RGW cho cả 3 trong trong section `[rgws]`
```sh 
cat <<EOF > inventory_hosts
[mons]
10.0.10.55
10.0.10.56
10.0.10.57

[osds]
10.0.10.55
10.0.10.56
10.0.10.57

[mdss]
10.0.10.55
10.0.10.56
10.0.10.57

[mgrs]
10.0.10.55

[grafana-server]
10.0.10.56
EOF
```

Copy file cấu hình cho `mds`
```sh 
cp group_vars/mdss.yml.sample group_vars/mdss.yml
```
> Config của MDS khá ít 

Kiểm tra lại toàn bộ cấu hình yml
```sh 
cat group_vars/*.yml | egrep -v '^$|^#'
```

Update ceph
```sh 
# Bare-metal
ansible-playbook site.yml -i inventory_hosts  --limit mdss
# Docker 
ansible-playbook site-container.yml -i inventory_hosts  --limit mdss
```

Kết quả 
```sh 
PLAY RECAP ********************************************************************************************************************************
10.0.10.55                 : ok=411  changed=11   unreachable=0    failed=0    skipped=575  rescued=0    ignored=0
10.0.10.56                 : ok=293  changed=9    unreachable=0    failed=0    skipped=446  rescued=0    ignored=0
10.0.10.57                 : ok=235  changed=6    unreachable=0    failed=0    skipped=402  rescued=0    ignored=0


INSTALLER STATUS **************************************************************************************************************************
Install Ceph Monitor           : Complete (0:00:23)
Install Ceph Manager           : Complete (0:00:20)
Install Ceph OSD               : Complete (0:00:33)
Install Ceph MDS               : Complete (0:01:11)
Install Ceph Dashboard         : Complete (0:00:31)
Install Ceph Grafana           : Complete (0:00:26)
Install Ceph Node Exporter     : Complete (0:00:23)

Saturday 25 July 2020  11:36:45 +0700 (0:00:00.053)       0:05:15.596 *********
===============================================================================
ceph-infra : install firewalld python binding ------------------------------------------------------------------------------------- 35.76s
install ceph-mds package on redhat or suse ---------------------------------------------------------------------------------------- 23.75s
ceph-mds : create bootstrap-mds and mds directories -------------------------------------------------------------------------------- 5.53s
ceph-grafana : wait for grafana to start ------------------------------------------------------------------------------------------- 4.23s
gather and delegate facts ---------------------------------------------------------------------------------------------------------- 4.15s
ceph-dashboard : set or update dashboard admin username and password --------------------------------------------------------------- 4.04s
ceph-mds : create filesystem pools ------------------------------------------------------------------------------------------------- 2.63s
check for python ------------------------------------------------------------------------------------------------------------------- 2.18s
ceph-mds : customize pool size ----------------------------------------------------------------------------------------------------- 2.11s
ceph-mds : customize pool min_size ------------------------------------------------------------------------------------------------- 2.10s
ceph-common : configure red hat ceph community repository stable key --------------------------------------------------------------- 2.06s
ceph-mds : assign application to cephfs pools -------------------------------------------------------------------------------------- 2.06s
ceph-mds : set pg_autoscale_mode value on pool(s) ---------------------------------------------------------------------------------- 1.95s
ceph-facts : check if the ceph mon socket is in-use -------------------------------------------------------------------------------- 1.87s
ceph-osd : systemd start osd ------------------------------------------------------------------------------------------------------- 1.79s
ceph-facts : check if the ceph mon socket is in-use -------------------------------------------------------------------------------- 1.79s
ceph-facts : check if the ceph mon socket is in-use -------------------------------------------------------------------------------- 1.77s
ceph-osd : unset noup flag --------------------------------------------------------------------------------------------------------- 1.63s
ceph-osd : apply operating system tuning ------------------------------------------------------------------------------------------- 1.62s
ceph-mds : copy ceph key(s) if needed ---------------------------------------------------------------------------------------------- 1.46s
[root@ceph01 ceph-ansible]# ceph df
RAW STORAGE:
    CLASS     SIZE        AVAIL       USED       RAW USED     %RAW USED
    hdd       300 GiB     294 GiB     25 MiB      6.0 GiB          2.01
    TOTAL     300 GiB     294 GiB     25 MiB      6.0 GiB          2.01

POOLS:
    POOL                ID     STORED      OBJECTS     USED      %USED     MAX AVAIL
    cephfs_data          1         0 B           0       0 B         0       139 GiB
    cephfs_metadata      2     2.9 KiB          22     1 MiB         0       139 GiB
[root@ceph01 ceph-ansible]#
## 