---
title: Linux的玩法
date: 2019-03-06
categories:
- 开发部署
tags:
- devops
---



# Linux命令

查看端口

- lsof -i:8080 列出系统打开的文件
- netstat -tunlp |grep 8080 列出包含8080端口的tcp、udp、拒绝别名、展示监听、进程名

查看进程

- ps -ef |grep java 查看java进程
- jps 查看java进程，需要安装jdk

杀进程

- kill -9 pid 强制干掉$pid
- kill -15 pid 优雅停机

清理旧包脚本

```bash
#!/bin/bash
rm -rf target
appname=$1
echo "appname:$appname"
pid=`ps -ef | grep $appname | grep 'java -Xms' | awk '{printf $2}'`
echo $pid
if [ -z $pid];
    then
        echo "$appname not start"
    else
        kill -9 $pid
        echo "$appname stoping..."
fi
check=`ps -ef | grep -w $pid | grep java`
if [ -z $check];
    then
        echo "$appname pid:$pid is stop"
    else
        echo "$appname stop failed"
fi
```

