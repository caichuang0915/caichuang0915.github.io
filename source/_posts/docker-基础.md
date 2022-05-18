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


#创建目录
mkdir -p /usr/local/mysql/conf
mkdir -p /usr/local/mysql/data

vim /usr/local/mysql/conf/my.cnf
# 写入
[mysqld]
lower_case_table_names=1

# 启动mysql服务
docker run -d -p 9401:3306 -v /usr/local/mysql/conf/:/etc/mysql/mysql.conf.d/ -v /usr/local/mysql/data/:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=828828 --name mysql mysql:5.7.38

# 查看日志
docker logs -g mysql

# 进入容器内部
docker exec -it mysql /bin/bash

# 停止服务
docker stop mysql

# 删除一个服务
docker rm mysql


```

使用mysql客户端连接，输入宿主机ip,端口9904,登录密码828828,成功登录。


docker启动命令分析

docker

- run 启动一个镜像,镜像不存在的情况回去拉取最新的镜像  
- -d  后台启动  
- -p  端口映射,将宿主机的端口和容器里面的端口进行映射,宿主机端口:容器内端口  
- -v  文件映射,将宿主机的路径和容器里面的路径进行挂载,相当于持久化  
- -e  设置环境参数  
- --name  设置容器的名称  
- -i  展示容器输入信息  
- -t  命令行交互  
 

数据库导入测试数据 [https://launchpad.net/test-db/+download](https://launchpad.net/test-db/+download)



```sql

-- 下载 employees_db-full-1.0.6.tar.bz2
tar -jxvf employees_db-full-1.0.6.tar.bz2

-- 解压后编辑 employees.sql 修改一下两点


-- set storage_engine = InnoDB;
set default_storage_engine = InnoDB;

-- select CONCAT('storage engine: ', @@storage_engine) as INFO;
select CONCAT('storage engine: ', @@default_storage_engine) as INFO;

-- 开始导入数据
mysql -hxx.xx.xx.xx -P9401 -uroot -p < employees.sql


```



#### docker安装nacos


```sh

docker run -d --name nacos -p 9402:8848 --privileged=true -e JVM_XMS=256m -e JVM_XMX=256m -e MODE=standalone -v /docker_config/nacos/logs/:/home/nacos/logs/ -v /data/dev_home/kyjgs1-caichuang/usr/local/nacos/conf/:/home/nacos/conf/ nacos/nacos-server

```





