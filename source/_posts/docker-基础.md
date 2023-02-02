---
title: docker基础
tags:
  - docker
---

docker

### docker安装



```sh

yum update

# 安装依赖
yum install -y yum-utils device-mapper-persistent-data lvm2

# 添加源
yum-config-manager --add-repo https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo

# 查看可安装的docker版本
yum list docker-ce --showduplicates

# 安装最新版本
yum install docker-ce -y

# 查看版本
docker -v

# 启动docker
systemctl start docker

# 查看docker 状态
systemctl status docker

# 设置开机启动
systemctl enable docker

# 设置docker镜像加速 阿里云地址 https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors  通过修改daemon配置文件/etc/docker/daemon.json来使用加速器

sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://8cnlnwwc.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```


<!-- more -->


### docker基本概念


- 镜像：包含容器运行环境的静态文件
- 容器：容器的实质就是进程
- 仓库
- xx

### docker简单操作


#### docker安装mysql


查询镜像的版本： [https://hub.docker.com/search?q=mysql](https://hub.docker.com/search?q=mysql) , 同时一个镜像的配置、环境变量之类的也可以在这个网址进行查询。


查询到合适版本拉取mysql镜像

```sh

# 拉取镜像
docker pull mysql:5.7.38

# 查看镜像
docker images

# 宿主机创建mysql配置文件




```