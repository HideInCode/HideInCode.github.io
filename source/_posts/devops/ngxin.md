# 安装部署

## docker下安装

1. docker pull nginx 

2. docker run --name nginx -p 9000:80 -d nginx 先启动一个容器用来复制配置

3. 生成挂载文件地址

   ```bash
   mkdir -p /opt/docker/nginx/conf
   mkdir -p /opt/docker/nginx/html
   mkdir -p /opt/docker/nginx/logs
   
   docker cp nginx:/etc/nginx/nginx.conf /opt/docker/nginx/conf/nginx.conf
   docker cp nginx:/etc/nginx/conf.d /opt/docker/nginx/conf/conf.d
   docker cp nginx:/usr/share/nginx/html /opt/docker/nginx
   ```

   

4. 干掉这个nginx

   ``` bash
   docker stop nginx 
   docker rm nginx
   ```

5. 以80端口启动，一行命令。

   ```bash
   docker run -p 80:80 --name nginx --restart=always -v /opt/docker/nginx/conf/nginx.conf:/etc/nginx/nginx.conf -v /opt/docker/nginx/conf/conf.d:/etc/nginx/conf.d -v /opt/docker/nginx/html:/usr/share/nginx/html -v /opt/docker/nginx/logs:/var/log/nginx -d nginx
   
   ```

   

## 常见问题

### 配置遗漏

1. 每句配置结尾有分号！！！

### 转发不到

1. 查看error.log，请求路径是否符合预期。

## 斜杠问题

> 配置为：
>
> location hello/ {
>
>  **proxy_pass** 192.168.56.10/
>
> }
>
> 请求：nginx主机/hello/all

1. location：不带/的话会自动补上，带的话能正确拼接。
2. proxy_pass：
   1. 带/：;最终请求：nginx主机/all，即忽略location.
   2. 不带/：最终请求：nginx主机/hello/all
3. location不带，proxy_pass带：会生成双斜杠，nginx主机//all，这是因为补齐机制导致。
4. 总结：proxy_pass看项目的controller有没有带，location最好带上。