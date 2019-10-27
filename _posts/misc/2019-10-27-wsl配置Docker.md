---
layout: post
title:  "wsl配置Docker"
date:   2019-10-27
desc: ""
keywords: ""
categories: [Misc]
tags: []
icon: icon-html
---


# wsl配置Docker

`Docker Desktop requires Windows 10 Pro or Enterprise version 15063 to run.`

告辞




##ubuntu18.04安装docker
### 卸载旧版本

```
sudo apt-get remove docker docker-engine docker.io containerd runc
```

### 设置Docker仓库

```
sudo apt-get update
```

```
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

添加密钥

```
url -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

```
sudo apt-key fingerprint 0EBFCD88
```

此时应出现

```
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

设置仓库

```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"
```

### 安装Docker Engine-Community

```
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

```
docker -v
Docker version 19.03.4, build 9013bf583a
```



若要卸载docker，卸载docker-ce时可能会报错

```
apt-get remove docker-ce
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following packages were automatically installed and are no longer required:
  aufs-tools cgroupfs-mount libcddb2 libcue1 libdiscid0 libfaad2 libmodplug1 libmpcdec6 libopusfile0 pigz
Use 'apt autoremove' to remove them.
The following packages will be REMOVED:
  docker-ce
0 upgraded, 0 newly installed, 1 to remove and 167 not upgraded.
1 not fully installed or removed.
After this operation, 109 MB disk space will be freed.
Do you want to continue? [Y/n] y
(Reading database ... 248263 files and directories currently installed.)
Removing docker-ce (5:19.03.4~3-0~ubuntu-bionic) ...
invoke-rc.d: could not determine current runlevel
 * Stopping Docker: docker
start-stop-daemon: warning: failed to kill 4733: No such process
No process in pidfile '/var/run/docker-ssd.pid' found running; none killed.
invoke-rc.d: initscript docker, action "stop" failed.
dpkg: error processing package docker-ce (--remove):
 installed docker-ce package pre-removal script subprocess returned error exit status 1
dpkg: error while cleaning up:
 installed docker-ce package post-installation script subprocess returned error exit status 1
Errors were encountered while processing:
 docker-ce
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

此时执行

```
sudo rm -rf /var/lib/dpkg/info/docker-ce.*
```

然后

```
apt-get remove docker-ce
```



## 配置Docker for Windows

WSL不支持Docker的守护进程，但可以使用Docker CLI连接到通过Docker for Windows运行的远程Docker守护进程

在Docker for Windows勾选 `Expose deamon on tcp://localhost:2345 without TLS`

然后在wsl执行export DOCKER_HOST=tcp://127.0.0.1:2375

每次打开bash都要设置，所以把他添加到.bashrc中

```
vi ~/.bashrc
把 export DOCKER_HOST=tcp://127.0.0.1:2375 添加到最后一行
source ~/.bashrc
```

