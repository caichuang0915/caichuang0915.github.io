---
title: Nginx - location流程 和 rewirte
tags:
  - Nginx
---

Location路径传递

#### rewrite使用

语法:rewrite regex replacement [flag];  flag=【break/last/redirect/permanent】 

- regex 是正则表达式
- replacement 是替换值，新值
- flag -- 是处理标志

break/last 内部重定向，换path值     
break标记，停止执行后续命令。for（）{break}     
last标记，会引发location重新匹配。for（）{continue;} 不要轻易使用last标签    
当有flag值的时候，rewrite层面的命令会中断。last会引发location重匹配   
当没有flag值的时候，rewrite还会往下走，最后一个rewrite覆盖前面的。再引发location重匹配    

<!-- more -->

#### proxy_pass 路径传递

- 通过url中【ip/域名+port】匹配server之后，携带后面path?params进入location逻辑
- location匹配后，path = 匹配中的path1 + 剩余path2

![路径传递](http://image.tupelo.top/root-alias.png)








