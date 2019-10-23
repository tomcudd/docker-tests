# docker-tests
For workshops and sessions, the copy-pastable commands for running tests

## Get docker running locally

### VMware and CentOS
I installed VMware player and CentOS on an image and put my instructions up on my site here: [tomcudd.com centos 7 setup](https://tomcudd.com/how-i-set-up-a-centos-7-virtual-machine/)

### Install docker
Here are the commands I used that I found at [Docker install instructions](https://docs.docker.com/install/linux/docker-ce/centos/)
```
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io
systemctl start docker
docker run hello-world
```

## Docker tests

### Set up and permissions
Set up the folders needed for test output, a configuration file, and set the permissions. I still plan to run everything as root or sudo for now
```
mkdir docker-tests
chmod o+w docker-tests/
chmod o+w docker-tests/*
mkdir docker-tests/pa11y
vim docker-tests/pa11y/config.json
```
The config file contents are below:
```
[root@c7v2 ~]# cat pa11y/config.json
		{
		    "defaults": {
		      "chromeLaunchConfig": {
		         "args": ["--no-sandbox"]
		       },
		        "timeout": 60000
		    },
		    "urls": [
		        "https://URLOFWEBSITETESTING"
		    ]
		}
```

### Run the commands
Here is the sauce of the work here:
#### Google Lighthouse
```
docker run --rm --name lighthouse -it -v /root/docker-tests/lighthouse:/home/chrome/reports --cap-add=SYS_ADMIN femtopixel/google-lighthouse https://URLOFWEBSITETESTING
```
#### Sitespeed.io
```
docker run --rm -v /root/docker-tests/sitespeed.io:/sitespeed.io sitespeedio/sitespeed.io:8.15.2 https://URLOFWEBSITETESTING
```
#### OWASP ZAP
```
docker run -v /root/docker-tests/zap:/zap/wrk/:rw -t owasp/zap2docker-stable zap-baseline.py -t https://URLOFWEBSITETESTING -g gen.conf -r tomcudd.html
```
#### Link Checker
```
docker run -it --rm -v /root/docker-tests/checklink:/home/checklink stupchiy/checklink -H https://URLOFWEBSITETESTING > /root/docker-tests/checklink-report.html
```
#### Pa11y
```
docker run -it -v /root/docker-tests/pa11y/config.json:/tmp/config.json digitalist/pa11y-ci:latest pa11y-ci -c /tmp/config.json > /root/docker-tests/pa11y/pa11y-output.txt
```