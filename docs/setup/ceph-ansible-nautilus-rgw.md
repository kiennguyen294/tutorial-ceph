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

[rgws]
10.0.10.55
10.0.10.56
10.0.10.57

[mgrs]
10.0.10.55

[grafana-server]
10.0.10.56
EOF
```


Update bổ sung `group_vars/all.ym`
```sh
## CephRGW
radosgw_interface: eth1
```

Kiểm tra lại toàn bộ cấu hình yml
```sh 
cat group_vars/*.yml | egrep -v '^$|^#'
```

Update ceph
```sh 
# Bare-metal
ansible-playbook site.yml -i inventory_hosts  --limit rgws
# Docker 
ansible-playbook site-container.yml -i inventory_hosts  --limit rgws
```

Kết quả 
```sh 
TASK [show ceph status for cluster ceph] ***************************************************************************
Friday 24 July 2020  17:42:50 +0700 (0:00:00.955)       0:09:05.858 *********** 
ok: [10.0.10.55 -> 10.0.10.55] => 
  msg:
  - '  cluster:'
  - '    id:     20ac6e99-0dd4-4310-b1c6-c83f8d05869e'
  - '    health: HEALTH_OK'
  - ' '
  - '  services:'
  - '    mon: 3 daemons, quorum ceph01,ceph02,ceph03 (age 6m)'
  - '    mgr: ceph01(active, since 19s)'
  - '    osd: 6 osds: 6 up (since 3m), 6 in (since 56m)'
  - '    rgw: 3 daemons active (ceph01.rgw0, ceph02.rgw0, ceph03.rgw0)'
  - ' '
  - '  task status:'
  - ' '
  - '  data:'
  - '    pools:   4 pools, 32 pgs'
  - '    objects: 192 objects, 3.5 KiB'
  - '    usage:   6.0 GiB used, 294 GiB / 300 GiB avail'
  - '    pgs:     32 active+clean'
  - ' '
  - '  io:'
  - '    client:   767 B/s rd, 170 B/s wr, 0 op/s rd, 0 op/s wr'
  - ' '

PLAY RECAP *********************************************************************************************************
10.0.10.56                 : ok=363  changed=15   unreachable=0    failed=0    skipped=424  rescued=0    ignored=0   
10.0.10.57                 : ok=305  changed=14   unreachable=0    failed=0    skipped=385  rescued=0    ignored=0   


INSTALLER STATUS ****************************************************************************************************
Install Ceph Monitor           : Complete (0:04:59)
Install Ceph Manager           : Complete (0:00:23)
Install Ceph OSD               : Complete (0:00:39)
Install Ceph RGW               : Complete (0:00:28)
Install Ceph Dashboard         : Complete (0:00:40)
Install Ceph Grafana           : Complete (0:00:25)
Install Ceph Node Exporter     : Complete (0:00:25)

Friday 24 July 2020  17:42:50 +0700 (0:00:00.050)       0:09:05.908 *********** 
=============================================================================== 
ceph-handler : restart ceph osds daemon(s) --------------------------------------------------------------------- 186.12s
ceph-handler : restart ceph mon daemon(s) ----------------------------------------------------------------------- 64.15s
ceph-handler : restart ceph mgr daemon(s) ----------------------------------------------------------------------- 10.94s
ceph-dashboard : set or update dashboard admin username and password --------------------------------------------- 6.02s
ceph-infra : install firewalld python binding -------------------------------------------------------------------- 5.23s
ceph-config : create ceph initial directories -------------------------------------------------------------------- 4.28s
ceph-config : create ceph initial directories -------------------------------------------------------------------- 4.10s
ceph-config : create ceph initial directories -------------------------------------------------------------------- 4.09s
ceph-config : create ceph initial directories -------------------------------------------------------------------- 3.91s
ceph-container-common : pulling docker.io/ceph/daemon:latest-nautilus image -------------------------------------- 3.41s
ceph-grafana : wait for grafana to start ------------------------------------------------------------------------- 3.22s
gather and delegate facts ---------------------------------------------------------------------------------------- 2.66s
ceph-config : generate ceph.conf configuration file -------------------------------------------------------------- 1.93s
ceph-handler : unset noup flag ----------------------------------------------------------------------------------- 1.82s
ceph-dashboard : disable mgr dashboard module (restart) ---------------------------------------------------------- 1.78s
ceph-config : generate ceph.conf configuration file -------------------------------------------------------------- 1.73s
ceph-osd : set noup flag ----------------------------------------------------------------------------------------- 1.72s
ceph-osd : systemd start osd ------------------------------------------------------------------------------------- 1.70s
ceph-config : generate ceph.conf configuration file -------------------------------------------------------------- 1.69s
ceph-osd : apply operating system tuning ------------------------------------------------------------------------- 1.65s
[root@ceph01 ceph-ansible]# 
```

## 