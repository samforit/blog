---
title: mysql blocked because of many connection errors
date: 2019-11-01 14:49:46
categories:
    - Mysql
tags:
    - 异常
    - sql
---


# 背景
今天启动服务的时候，发现有一个服务一直启动不起来，报错如下：
```shell
Caused by:
java.sql.SQLException: null,
message from server: "Host '192.168.0.10' is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'"
```

<!-- more -->

# 分析
这里的错误已经很明确了，就说某一个ip产生了大量的错误链接，然后这个IP就被锁了，如果要解除锁定，就用mysql自带的名命令'mysqladmin flush-hosts'解锁即可。

# 解决
1. 通过提示，用命令'mysqladmin flush-hosts'。
> 如果是远程机器，可以使用 :
mysqladmin flush-hosts -h 192.168.1.9 -P 3306 -u root -p

2. 进入mysql命令行，输入'flush hosts'也是可以的。如下图所示：
{% asset_img mysql-flush-hosts-1.png %}

3. 修改max_connection_errors
> 查看：show variables like 'max_connect_errors';
修改：set global max_connect_errors = 1000;
校验：show variables like 'max_connect_errors';

{% asset_img mysql-flush-hosts-2.png %}
