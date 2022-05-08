---
title: Homebrew 安装Mysql密码忘记解决
date: 2021-12-19 15:40:17
tags: Mysql
---


我的Mac很久之前安装了mysql，当时设置了密码，最近重新想起来用，但是忘记了密码，网上找了很多文章，千篇一律几乎都是复制粘贴，个别过分的就贴一个链接完事，找了很久终于找到了解决方案，所以做个记录，免得遇到同样问题的人跟我一样找半天解决方案



### 5.8及之前，网上文章几乎都是针对此种情况

**第一步**
因为忘记密码无法启动，所以执行下面命令进入安全启动模式
```
mysql.server start --skip-grant-tables
```
**第二步**
登录
```
mysql -uroot
```
**第三步**
此时进入`mysql`下

```
use mysql
```
**第四步**
接着执行
```
FLUSH PRIVILEGES; 
```
**第五步**
设置密码
```
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('你的新密码');
```
退出mysql，杀掉进程，重新启动

### 5.8之后，我的是8.0.25

前四步操作同上
```
ALTER user 'root'@'localhost' IDENTIFIED BY '你的新密码';
```
退出mysql，杀掉进程，重新启动即可正常登陆

[文档](https://dev.mysql.com/doc/refman/8.0/en/resetting-permissions.html)
