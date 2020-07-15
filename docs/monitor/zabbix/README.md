# Ghi chép một số bước monitor CEPH bằng phần mềm zabbix - grafana

### Mục lục

[1. Mô hình](#mohinh)<br>
[2. IP Planning](#planning)<br>
[3. Thao tác trên node CEPH](#nodeceph)<br>
[4. Thao tác trên node zabbix](#nodezabbix)<br>
[5. Test](#test)<br>
[6. Import graph grafana](#grafana)<br>

<a name="mohinh"></a>
## 1. Mô hình triển khai

Mô hình triển khai gồm

+ 01 cụm CEPH (192.168.90.51)<br>
+ 01 zabbix server (192.168.90.110)<br>
+ 01 grafana server(192.168.90.111)<br>

![](../images/img-ceph-zabbix/topo.png)

<a name="planning"></a>
## 2. IP Planning


<a name="nodeceph"></a>
## 3. Thao tác trên node CEPH

Thực hiện trên node CEPH cài service monitor của cụm CEPH

![](../images/img-ceph-zabbix/Screenshot_365.png)

### Cài zabbix-sender trên node Ceph
```sh 
sudo ceph mgr module ls | grep zabbix 
rpm -ivh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
yum update -y 
yum install zabbix-sender -y
ceph mgr module enable zabbix
ceph zabbix config-show
# Local 
ceph zabbix config-set identifier CephAIO
# **Lưu ý**: Tên định danh `CephAIO` phải trùng với tên `Host name` khi add host trên zabbix.
# Zabbix server
ceph zabbix config-set zabbix_host 10.0.10.59
ceph zabbix config-set zabbix_port 10051
ceph zabbix config-set interval 60
```

`zabbix_host`: IP hoặc host name của Zabbix server 
`identifier`: Hostname của node khai báo trên Zabbix 
`port`: mặc định 10051
`interval`: mặc định 60s 

### Link tham khảo

https://developers.redhat.com/blog/2020/03/23/ceph-storage-monitoring-with-zabbix/

