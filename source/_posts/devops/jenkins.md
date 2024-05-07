---
title: Jenkin安装
date: 2018-09-06
categories:
- 开发部署
tags:
- devops
---



# Jenkins部署与使用

1. 去官网下载war包，注意支持的Java版本[Jenkins官网](https://www.jenkins.io/zh/)

2. 上传到服务器，下载jdk，使用命令：nohup java -Xms515m -Xmx1024m -jar -Dfile.encoding=UTF-8 jenkins.war > jenkins.log 2>&1 &

3. 进入2中生成log查看密码，或者/root/.jenkins/secrets/initialAdminPassword中看到密码。

4. host:8080登录,选择推荐安装，能跳过先跳过。

5. 修改密码

6. 安装[Maven Integration plugin](https://plugins.jenkins.io/maven-plugin)、[Build Authorization Token Root Plugin](https://plugins.jenkins.io/build-token-root)、[Publish Over SSH](https://plugins.jenkins.io/publish-over-ssh)。用于进行maven打包、自动构建匿名token，链接服务器。

7. 在服务器上安装maven，git。maven要配置好国内镜像。

8. 新建任务选择**构建一个maven项目**，这一步必须安装插件才有

9. 在**系统管理**->**系统配置**中，**JekinsLocation**指定jenkins服务器地址；配置**Publish over SSH**执行jar包的服务器地址与用户

10. 在**源码管理中**配置git仓库地址与分支，**PostSteps**中配置执行jar包的服务器(使用绝对路径)，设置启动命令。在**构建触发器**中触发远程构建，自定义令牌，利用Build Authorization Token Root Plugin，生成链接，具体参考插件官网。

    ```bash
    #启动项目
    nohup java -Xms515m -Xmx1024m -jar -Dfile.encoding=UTF-8 *.jar > app.log 2>&1 &
    ```

    

11. 在**PreSteps**中远程清除jar包，杀死旧进程。

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

    

12. 点击构建，观察控制台。

13. 可以配置gitlab的webhook触发自动部署，记得切换管理员到network中**外发请求**中勾选允许本地。

14. 可以利用定时构建和cron表达式来定期构建。判断表达式可用工具：[cron工具](https://crontab.guru/)

15. 可以配置告警邮件，要自己给出支持SMTP的邮箱，推荐网易，QQ不行。