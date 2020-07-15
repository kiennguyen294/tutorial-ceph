# Sync Data from Vultr S3 to Local S3 using s3sync
https://www.digitalocean.com/community/tutorials/how-to-migrate-from-amazon-s3-to-digitalocean-spaces-with-rclone

## Tạo user và tương ứng 
Source 
```sh 
Hostname:
https://ewr1.vultrobjects.com
Secret Key:
7aQO91d7vmqKLhWjIWopQfLitFRJxlQ1EUckKN83
Access Key:
Q1M9T1VNH5WT9JZ37CVJ 
Bucket:
s3://canhdx  
```

Target 
```sh
Hostname:
https://s3.azunce.xyz
Secret Key:
uQ7R74jPt1rEQJdUSYlQIMmVAkCUTPKO7vaE74v4
Access Key:
J5CVYE51K5WXC035HAZE 
Bucket:
s3://canhdx  
```


## Cài đặt RCLONE 

```sh 
yum install epel-release -y 
yum update -y 
yum install rclone -y 
```

Cấu hình rclone `~/.config/rclone/rclone.conf`
```sh 
[vultr]
type = s3
provider = Vultr
env_auth = false
access_key_id = Q1M9T1VNH5WT9JZ37CVJ
secret_access_key = 7aQO91d7vmqKLhWjIWopQfLitFRJxlQ1EUckKN83
region =
endpoint = https://ewr1.vultrobjects.com
location_constraint =
acl =
server_side_encryption =
storage_class =

[ceph]
type = s3
provider = Ceph
env_auth = false
access_key_id = J5CVYE51K5WXC035HAZE
secret_access_key = uQ7R74jPt1rEQJdUSYlQIMmVAkCUTPKO7vaE74v4
region =
endpoint = https://s3.azunce.xyz
location_constraint =
acl =
server_side_encryption =
storage_class =
~                                                                                           
~                    
```

![](https://i.imgur.com/hppp889.png)