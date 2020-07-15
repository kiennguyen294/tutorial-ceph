# Sync Data from Vultr S3 to Local S3 using s3sync
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


## Cài đặt Docker 
Install Docker 
```sh 
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

sudo yum install -y yum-utils

sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install docker-ce docker-ce-cli containerd.io

sudo systemctl start docker

sudo systemctl enable docker
```

## Cách sử dụng 
```
  --sk SK                Source AWS key
  --ss SS                Source AWS session secret
  --st ST                Source AWS token
  --sr SR                Source AWS Region
  --se SE                Source AWS Endpoint
  --tk TK                Target AWS key
  --ts TS                Target AWS secret
  --tt TT                Target AWS session token
  --tr TR                Target AWS Region
  --te TE                Target AWS Endpoint
  --s3-retry S3-RETRY    Max numbers of retries
```

Sync Amazon S3 bucket to FS:
```
s3sync --sk KEY --ss SECRET -w 128 s3://shared fs:///opt/backups/s3/
```

Sync S3 bucket with custom endpoint to FS:
```
s3sync --sk KEY --ss SECRET --se "http://127.0.0.1:7484" -w 128 s3://shared fs:///opt/backups/s3/
```

Sync directory (/test) from Amazon S3 bucket to FS:
```
s3sync --sk KEY --ss SECRET -w 128 s3://shared/test fs:///opt/backups/s3/test/
```

Sync directory from local FS to Amazon S3:
```
s3sync --tk KEY --ts SECRET -w 128 fs:///opt/backups/s3/ s3://shared
```

Sync directory from local FS to Amazon S3 bucket directory:
```
s3sync --tk KEY --ts SECRET -w 128 fs:///opt/backups/s3/test/ s3://shared/test_new/
```

Sync one Amazon bucket to another Amazon bucket:
```
s3sync --tk KEY2 --ts SECRET2 --sk KEY1 --ss SECRET1 -w 128 s3://shared s3://shared_new
```

Sync S3 bucket with custom endpoint to another bucket with custom endpoint:
```
s3sync --tk KEY2 --ts SECRET2 --sk KEY1 --ss SECRET1 --se "http://127.0.0.1:7484" --te "http://127.0.0.1:7484" -w 128 s3://shared s3://shared_new
```

Sync one Amazon bucket directory to another Amazon bucket:
```
s3sync --tk KEY2 --ts SECRET2 --sk KEY1 --ss SECRET1 -w 128 s3://shared/test/ s3://shared_new
```



Using 
```sh 
docker run --rm -ti larrabee/s3sync --tk KEY2 --ts SECRET2 --sk KEY1 --ss SECRET1 -w 128 s3://shared/test/ s3://shared_new
```

## Sync
docker run --rm -ti larrabee/s3sync --debug --tk "J5CVYE51K5WXC035HAZE" --ts "uQ7R74jPt1rEQJdUSYlQIMmVAkCUTPKO7vaE74v4" --sk "Q1M9T1VNH5WT9JZ37CVJ" --ss "7aQO91d7vmqKLhWjIWopQfLitFRJxlQ1EUckKN83" --se "https://ewr1.vultrobjects.com" --te "https://s3.azunce.xyz" -w 4 s3://canhdx s3://canhdx

Docker kết quả 
```
[root@cephaio ~]# docker run --rm -ti larrabee/s3sync --debug --tk "J5CVYE51K5WXC035HAZE" --ts "uQ7R74jPt1rEQJdUSYlQIMmVAkCUTPKO7vaE74v4" --sk "Q1M9T1VNH5WT9JZ37CVJ" --ss "7aQO91d7vmqKLhWjIWopQfLitFRJxlQ1EUckKN83" --se "https://ewr1.vultrobjects.com" --te "https://s3.azunce.xyz" -w 4 s3://canhdx s3://canhdx
INFO[0000] Starting sync                                
DEBU[0002] S3 listing failed with error: RequestError: send request failedcaused by: Get https://ewr1.vultrobjects.com/canhdx?encoding-type=url&max-keys=1000&prefix=: x509: certificate signed by unknown authority 
DEBU[0002] Pipeline step: ListSource finished           
DEBU[0002] Pipeline step: LoadObjData finished          
DEBU[0002] Recv pipeline err: RequestError: send request failed
caused by: Get https://ewr1.vultrobjects.com/canhdx?encoding-type=url&max-keys=1000&prefix=: x509: certificate signed by unknown authority DEBU[0002] Pipeline step: UploadObj finished            
DEBU[0002] Pipeline step: Terminator finished           
DEBU[0002] All pipeline steps finished                  
ERRO[0002] Sync error: pipeline step: 0 (ListSource) failed with error: RequestError: send request failed
caused by: Get https://ewr1.vultrobjects.com/canhdx?encoding-type=url&max-keys=1000&prefix=: x509: certificate signed by unknown authority, terminating 
INFO[0002] 0 ListSource: Input: 0; Output: 0 (0 obj/sec); Errors: 1 
INFO[0002] 1 LoadObjData: Input: 0; Output: 0 (0 obj/sec); Errors: 0 
INFO[0002] 2 UploadObj: Input: 0; Output: 0 (0 obj/sec); Errors: 0 
INFO[0002] 3 Terminator: Input: 0; Output: 0 (0 obj/sec); Errors: 0 
INFO[0002] Duration: 2.337719232s                       
ERRO[0002] Sync Failed                                  
DEBU[0002] Pipeline terminated                          
[root@cephaio ~]# 
```


https://tecadmin.net/install-go-on-centos/

Cài đặt GO trên CentOS7 
```sh 
yum update -y 
yum install wget -y 
wget https://dl.google.com/go/go1.13.3.linux-amd64.tar.gz
tar -xzf go1.13.3.linux-amd64.tar.gz
mv go /usr/local
reboot -f
git clone https://github.com/uncelvel/s3sync.git
cd s3sync 
go mod vendor
go build -o s3sync ./cli 
./s3sync
```


Kiểm ra env 
```sh 
[root@cephaio s3sync]# go env
GO111MODULE=""
GOARCH="amd64"
GOBIN=""
GOCACHE="/root/.cache/go-build"
GOENV="/root/.config/go/env"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GONOPROXY=""
GONOSUMDB=""
GOOS="linux"
GOPATH="/root/go"
GOPRIVATE=""
GOPROXY="https://proxy.golang.org,direct"
GOROOT="/usr/local/go"
GOSUMDB="sum.golang.org"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/linux_amd64"
GCCGO="gccgo"
AR="ar"
CC="gcc"
CXX="g++"
CGO_ENABLED="1"
GOMOD="/root/s3sync/go.mod"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build684300700=/tmp/go-build -gno-record-gcc-switches"
[root@cephaio s3sync]# 
```





```sh 
[root@cephaio s3sync]# ./s3sync --debug --tk "J5CVYE51K5WXC035HAZE" --ts "uQ7R74jPt1rEQJdUSYlQIMmVAkCUTPKO7vaE74v4" --sk "Q1M9T1VNH5WT9JZ37CVJ" --ss "7aQO91d7vmqKLhWjIWopQfLitFRJxlQ1EUckKN83" --se "https://ewr1.vultrobjects.com" --te "https://s3.azunce.xyz" -w 1 s3://canhdx s3://canhdx --sync-log
INFO[0000] Starting sync                                
DEBU[0001] Listing bucket finished                      
DEBU[0001] Pipeline step: ListSource finished      
```

RAM CPU
https://i.imgur.com/tTfnicX.png

Bandwidth 
https://i.imgur.com/zZwSOXW.png


ACL AWS
https://i.imgur.com/N3AnPnP.png

ACL Client 
https://i.imgur.com/Qs8D23j.png

[{'Grantee': {'DisplayName': 'CanhDX', 'ID': 'canhdx', 'Type': 'CanonicalUser'}, 'Permission': 'FULL_CONTROL'}]
[{'Grantee': {'DisplayName': 'CanhDX', 'ID': 'canhdx', 'Type': 'CanonicalUser'}, 'Permission': 'READ'}]
[{'Grantee': {'DisplayName': 'CanhDX', 'ID': 'canhdx', 'Type': 'CanonicalUser'}, 'Permission': 'WRITE'}]
[{'Grantee': {'DisplayName': 'CanhDX', 'ID': 'canhdx', 'Type': 'CanonicalUser'}, 'Permission': 'READ_ACP'}]
[{'Grantee': {'DisplayName': 'CanhDX', 'ID': 'canhdx', 'Type': 'CanonicalUser'}, 'Permission': 'WRITE_ACP'}]