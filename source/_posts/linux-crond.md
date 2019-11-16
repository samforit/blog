---
title: 使用crond构建linux定时任务及日志查看
categories:
    - Linux
tags:
    - crond
    - Linux
date: 2018-03-02 00:00:00
---


# quick start
1. 新建一个定时任务配置文件
```
[root@ubuntu ~]# vim /etc/cron.d/myTask
```

<!-- more -->

2. 编辑内容如下：
```
[root@ubuntu ~]# cat /etc/cron.d/myTask
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# 将当前时间写入到log文件,一小时执行一次
0 * * * * root echo `date` >> /root/date.log
```

3. 重新载入配置文件：
```
[root@ubuntu ~]# service crond reload
```
搞定

---
# 解释
1. 定时任务配置文件可以放到`/etc/cron.d`文件夹下，reload 或 restart 的时候crond服务会扫描该文件夹下的文件。

2. 定时任务配置：
执行时间(cron表达式) + 执行用户 + 任务
> 示例： 0 * * * * root python /root/hello.py

3. 新增或修改定时任务配置后，需要reload才能生效。

4. 服务相关命令：
service crond start    //启动服务
service crond stop     //关闭服务
service crond restart  //重启服务
service crond reload   //重新载入配置
service crond status   //查看服务状态

---
# 日志
1. 查看任务有没有运行：
```
[root@ubuntu ~]# tail -2 /var/log/cron
Dec 15 06:00:01 ubuntu CROND[28783]: (root) CMD (echo `date` >> /root/date.log)
Dec 15 06:01:01 ubuntu CROND[28923]: (root) CMD (echo `date` >> /root/date.log)
```

2. 如果运行中报错等，会有邮件记录，在此处查看详情：
```
[root@ubuntu ~]# tail -2 /var/spool/mail/root
```

