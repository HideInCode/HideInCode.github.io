---
title: Docker部署
date: 2024-05-07 01:45:50
categories:
- 开发部署
tags:
- devops
---



## docker部署

## 部署操作

1. 进入官网，查看manu文档

2. 按照系统执行命令，本地虚拟机可以先yum update

3. 安装好引擎后添加阿里云镜像加速

   ```bash
   sudo mkdir -p /etc/docker
   sudo tee /etc/docker/daemon.json <<-'EOF'
   {
     "registry-mirrors": ["https://p63g7kqk.mirror.aliyuncs.com"]
   }
   EOF
   sudo systemctl daemon-reload
   sudo systemctl restart docker
   ```

​	4. 设置开机自启动：`sudo systemctl enable docker`

## 安装docker引擎时出错：

重新配置yum源

```bash
cd /etc/yum.repos.d

rm -rf  !(CentOS_Base.Repo)

yum update
```

## 打包成镜像运行

```bash
# docker build -f Dockerfile -t docker.io/shh/demo:v1.0 .
# docker run -d --name demo -p 8080:8080 shh/demo:v1.0
```

## Dockfile模板

```bash
FROM java:8
EXPOSE 8080
# VOLUME 指定了临时文件目录为/tmp。
# 其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp
VOLUME /tmp
# 将jar包添加到容器中并更名为app.jar
ADD HelloDocker-0.0.1-SNAPSHOT.jar /app.jar
# 运行jar包
RUN bash -c 'touch /app.jar'
ENTRYPOINT ["java","-jar","/app.jar"]
```

## 安装Mysql

1. 下载镜像：`docker pull mysql:5.7`

2. 指定端口映射、文件挂载、密码后启动：

   ```bash
   docker run -p 3306:3306 --name mysql \
   -v /mydata/mysql/log:/var/log/mysql \
   -v /mydata/mysql/data:/var/lib/mysql \
   -v /mydata/mysql/conf:/etc/mysql \
   -e MYSQL_ROOT_PASSWORD=root \
   -d mysql:5.7
   ```

3. 修改配置

   ```bash
   vi /mydata/mysql/conf/my.cnf
   
   [client]
   default-character-set=utf8
   [mysql]
   default-character-set=utf8
   [mysqld]
   init_connect='SET collation_connection = utf8_unicode_ci' init_connect='SET NAMES utf8' character-set-server=utf8
   collation-server=utf8_unicode_ci
   skip-character-set-client-handshake
   skip-name-resolve
   ```

   

##   安装Redis

 1. 下载镜像：`docker pull redis`

 2. 启动：要先在本机生成文件后，才能挂载。

    ```bash
    mkdir -p /mydata/redis/conf
    touch /mydata/redis/conf/redis.conf
    
    docker run -p 6379:6379 --name redis -v /mydata/redis/data:/data \
    -v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
    -d redis redis-server /etc/redis/redis.conf
    ```


## 安装ES

1. 镜像

   ```bash
   docker pull elasticsearch:7.4.2
   docker pull kibana:7.4.2 
   ```

2. 创建实例

   ```bash
   mkdir -p /mydata/elasticsearch/config
   mkdir -p /mydata/elasticsearch/data
   echo "http.host: 0.0.0.0" >> /mydata/elasticsearch/config/elasticsearch.yml
   
   chmod -R 777 /mydata/elasticsearch/
   
   docker run --name elasticsearch -p 9200:9200 -p 9300:9300 \
   -e "discovery.type=single-node" \
   -e ES_JAVA_OPTS="-Xms64m -Xmx512m" \
   -v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
   -v /mydata/elasticsearch/data:/usr/share/elasticsearch/data \
   -v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
   -d elasticsearch:7.4.2
   
   # kibibna 地址要改
   docker run --name kibana -e ELASTICSEARCH_HOSTS=http://192.168.56.10:9200 -p 5601:5601 \
   -d kibana:7.4.2
   ```


## RabbitMQ

1. 下载镜像：docker pull rabbitmq

2. 运行：

   ```bash
   docker run -d --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 rabbitmq:management
   ```

3. 端口解释：

   - 4369, 25672 (Erlang发现&集群端口)
   - 5672, 5671 (AMQP端口)
   - 15672 (web管理后台端口) 
   - 61613, 61614 (STOMP协议端口) 
   - 1883, 8883 (MQTT协议端口) https://www.rabbitmq.com/networking.html
