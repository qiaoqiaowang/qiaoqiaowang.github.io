---
title: crontab执行Node.js脚本
date: 2019-01-27 13:22:46
tags:
---

    25 13 * * * /usr/bin/nohup /root/.nvm/versions/node/v10.12.0/bin/node /search/odin/username/cmd/wenke-webpack-cmd.js > /search/odin/username/test1.log 2>&1 &

以上是一个正确的命令。
创建了crontab脚本，一直不执行。
### 1. 查找crontab任务执行失败的原因可以查看 /var/spool/mail/root 文件的日志。(linux机器)我起初是用非root账号username登陆，创建的crontab任务，但是日志却是在/var/spool/mail/root文件，而不是/var/spool/mail/username文件下。提示 username: user NOT in sudoers。 于是我给username加了sudo权限。执行命令:

    visudo
    //然后在文件内增加一行 username ALL=(ALL) ALL

最后发现白费功夫，请看下面两条。

### 2. crontab脚本中的node命令需要使用绝对路径。查看crontab的环境变量:

    cat /etc/crontab

    SHELL=/bin/bash
    PATH=/sbin:/bin:/usr/sbin:/usr/bin
    MAILTO=root

    # For details see man 4 crontabs

    # Example of job definition:
    # .---------------- minute (0 - 59)
    # |  .------------- hour (0 - 23)
    # |  |  .---------- day of month (1 - 31)
    # |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
    # |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
    # |  |  |  |  |
    # *  *  *  *  * user-name  command to be executed

看起来这里记录了PATH，不太理解这个文件的作用。总之把node路径换成绝对的，然后看第3点

### 3. 如果node脚本里面也有访问node命令或者其他的，也要换成绝对路径。这里我的node是在/root/ 目录下，所以改完crontab定时任务还是不执行，非root账户没有权限访问/root/下的文件。。 最终换成在root账户下创建crontab 任务，才顺利执行。


