---
title: git无密码push
date: 2018-01-15 10:39:10
tags: 团队协作
---

# git ssh 免密提交

1. 查看配置

> git config --lis 
> 用户名和邮箱如果填写过则pass
> git config --global user.name "username"
> git config --global user.email  "mail@qq.com"

2. 生成SSH秘钥(group-hms-key是别名，随意取)

> ssh-keygen -t rsa -C "group-hms-key"

```
执行后，
第一个提示输入保存文件名，默认为空，回车。
第二个提示输入密钥，默认为空，回车
第三个确认刚输入密钥，默认为空，回车
完成后，默认保存位置当前   用户名下/.ssh/id_rsa 和id_rsa.pub（windows8和10下位置是：C:\Users\用户名）
```

3. 设置密钥

```
上面创建的git server的用户，这里创建的Gitblit内部用于管理权限的用户，两者要同名，当通过https链接git服务器时，需要输入用户名和密码，密码就是GitBlit中创建用户时填写的密码。

创建好以后，用新账号登陆（不是admin账号），然后在当前用户的用户中心把 id_rsa.pub 中的内容复制到SSH Keys，保存确定。
```

4. clone 克隆设置

> * ssh://test@ip:port/test.git  *
>   注意：**ssh请求才能无密码访问，ssh key对https请求无效**。

