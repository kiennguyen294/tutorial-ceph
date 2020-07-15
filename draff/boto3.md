# CentOS7


# Ubuntu18 
Cài đặt python3
```sh 
sudo apt-get install -y software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get install python3-dev build-essential python3.6 python3.6-dev python3-distutils libmysqlclient-dev -y
wget https://bootstrap.pypa.io/get-pip.py
sudo python3.6 get-pip.py
sudo pip3.6 install virtualenv
sudo apt-get upgrade python3
```

Cài đặt venv
```sh 
sudo apt-get install git -y
mkdir -p /home/canhdx/MEGA/CAS-Note/boto/{src,bin,etc,docs}
cd /home/canhdx/MEGA/CAS-Note/boto/src
virtualenv venv -p python3.6
cd ..
source src/venv/bin/activate
```

Sau khi source `venv` chúng ta install boto3 
```sh 
pip install boto3
```