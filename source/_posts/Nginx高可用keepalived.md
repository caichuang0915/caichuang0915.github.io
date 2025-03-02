---
title: Nginx - keepalived
tags:
  - Nginx
---

Nginx高可用keepalived

#### keepalived

思路：

由 2台服务器软件虚拟出来一台 虚拟网关vip，此vip由两台机器共同协商生成。当有一台机器宕机时，另一台机器一样能维持vip。这保证了，只要两台机器不同时宕机，vip就存在(不可以用docker配置)		

keepalived不常用：域名绑定ip是以实现IP路由功能 更方便

功能：

- 管理LVS负载均衡软件
- 实现LVS集群节点的健康检查
- 作为系统网络服务的高可用性（failover）

<!-- more -->

keepalived配置步骤：

1. 下载地址：https://pan.baidu.com/s/1G7sLL-YkZGSMu8G76yz1Rw 密码：adbw。对应centos6系统
2. keepalived安装步骤：		
  A〉./configure --prefix=/data/program/keepalived --sysconf=/etc 		  ##因为keepalive启动时候会默认读取/etc/keepalived/keepalived.conf 		
  B〉make && make install		
3. 修改/etc/keepalived/keepalived.conf配置文件信息
4. 启动keepalived	/data/program/keepalived/sbin/keepalived
5. KEEPLIVED主从故障测试,起停主从keepalived，查看对应的vip=192.168.244.200的漫游情况
6. 加nginx监控脚本,停止nginx，观察keepalived自动拉起nginx服务
7. 查看keepalived日志： tail -f  /var/log/messages


配置：

```sh
global_defs {             #全局配置    
    router_id nginx_backup       #表示运行Keepalived服务器的一个标识，唯一的 两台机器一致
}
vrrp_script chk_http_port {
    script "/usr/local/src/chk_nginx_pid.sh" #心跳执行的脚本
    interval 2                          #（检测脚本执行的间隔，单位是秒）
    weight 2
}
vrrp_instance VI_1 {        #vrrp 实例定义部分
    state BACKUP            # 指定keepalived的角色，MASTER为主，BACKUP为备
    interface eth0         # 当前进行vrrp通讯的网络接口卡(当前centos的网卡)
    virtual_router_id 66    # 虚拟路由编号，主从要一致
    priority 100            # 优先级，数值越大，获取处理请求的优先级越高
    advert_int 1            # 检查间隔，默认为1s(vrrp组播周期秒数)
    authentication {
        auth_type PASS #设置验证类型和密码，MASTER和BACKUP必须使用相同的密码才能正常通信
        auth_pass 1111
    }
    track_script {
        chk_http_port            #（调用检测脚本）
    }
    virtual_ipaddress {
        192.168.244.200            # 定义虚拟ip(VIP)，可多设，每行一个
    }
}
```

监控脚本：

```sh
#!/bin/bash
A=`ps -C nginx --no-header |wc -l`        
if [ $A -eq 0 ];then                            
      /usr/local/nginx/sbin/nginx                #重启nginx
      if [ `ps -C nginx --no-header |wc -l` -eq 0 ];
      then    #nginx重启失败，则停掉keepalived服务，进行VIP转移
              killall keepalived    #杀掉，vip就漫游到另一台机器                
      fi
fi
```













