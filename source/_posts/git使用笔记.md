---
title: Git使用笔记
tags:
  - Git
---

Git使用笔记

## Git使用笔记

#### 安装和配置

下载地址 ： ```https://git-scm.com/downloads```      
查看当前git用户名： ```git config user.name```      
查看当前git邮箱： ```git config user.email```      
切换git用户名: ```git config --global user.name "user name"```       
切换git邮箱： ```git config --global user.email  "e-mail"```     
生成SSH Keys: ```ssh-keygen -t rsa -C "e-mail"```


#### Git指令模式


##### 基本操作      

- ```git init``` 初始化一个git项目
- ```git commit -am 'new project'``` 加入缓存区并提交
- ```git chceckout [-b] king```  切换(新增分支)

##### 远程仓库

- ```git remote add origin https://github.com/xx/xxx.git```  本地新建一个远程连接
- ```git remote [-v]```  查看远程连接(详情)
- ```git branch --set-upstream-to=origin/xxx xxx```  本地分支与远程分支关联
- ```git push --set-upstream origin master```  上传本地项目至GitHub
- ```git fetch```  提取远程仓库

##### 分支合并

- ```git merge xxx``` 将指定版本与当前版本合并

> 有冲突的话相关的文件会显示,需要手动解决冲突后再次push


##### 日志标签

- ```git log [--oneline] [--graph]``` 显示提交日志(简化成一行)(显示分支)
- ```git tag [-a][-d] xxx -m 'xxxx'``` 为某一个分支(创建)(删除)一个tag
- ```git show xxx``` 显示某一个tag的附属信息
- ```git push origin -tags``` 本地标签提交到远程仓库


##### 代码回滚 两种方式：回退（reset）、反做（revert）

- ```git reset``` 的作用是修改HEAD的位置，即将HEAD指向的位置改变为之前存在的某个版本。（比如需要回到到V0.3）使用 ```git reset --hard 目标版本号``` 命令将版本回退
- ```git revert``` 的作用通过反做创建一个新的版本，这个版本的内容与我们要回退到的目标版本一样，但是HEAD指针是指向这个新生成的版本，而不是目标版本。


