# Nginx with Docker-Compose
## Nginx 是什么
我目前的认识是，Nginx是一个静态资源服务器，与Tomcat和Jetty不同，Nginx主要管理静态资源。  
但是实际上，Nginx是一个反向代理服务器，同时也是负载均衡器，http缓存和Web服务器。  
我目前的需求是前后端分离时前端的静态网页使用Nginx来管理，使用了它Web服务器的功能。以后如果前端使用了Vue或者React之类的话就不会用到这个功能。（Nginx其它功能目前用不上，暂时不去了解）

## 怎么使用Nginx
在看了DockerHub nginx的官方文档后，我觉得采用Dockerfile拉取镜像的方式比较可取，所以准备采用这种方式。
### 新建必要文件
首先，新建对应的文件夹，我的文件夹为ui-service，在其下新建一个static文件夹和Dockerfile文件以及nginx.conf，然后把静态网页复制到static文件夹下：  
![](assets/002/20190131-895ade8b.png)  
### 编写Dockerfile
之后打开Dockerfile，按照官方教程写对应的内容：
```shell
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
COPY static /usr/share/nginx/html
```
这里主要设置了去拉取镜像，并且把静态文件和配置文件复制到指定文件夹（其实类似于Docker-compose中使用image，volume）。这种写法主要参考了DockerHub nginx 下的文档，见相关网站的[1]。

### 编写nginx.conf
```yml
worker_processes  5;  ## Default: 1
# error_log  logs/error.log;
# pid        logs/nginx.pid;
worker_rlimit_nofile 8192;

events {
  worker_connections  4096;  ## Default: 1024
}

http {
  include    mime.types;
  #include    /etc/nginx/proxy.conf;
  #include    /etc/nginx/fastcgi.conf;
  #index    index.html index.htm index.php;

  default_type application/octet-stream;
  #log_format   main '$remote_addr - $remote_user [$time_local]  $status '
  #  '"$request" $body_bytes_sent "$http_referer" '
  #  '"$http_user_agent" "$http_x_forwarded_for"';
  #access_log   logs/access.log  main;
  sendfile     on;
  #tcp_nopush   on;
  server_names_hash_bucket_size 128; # this seems to be required for some vhosts

  #server { # php/fastcgi
  #  listen       80;
  #  server_name  domain1.com www.domain1.com;
  #  access_log   logs/domain1.access.log  main;
  #  root         html;

  #  location ~ \.php$ {
  #    fastcgi_pass   127.0.0.1:1025;
  #  }
  #}

  server { # simple reverse-proxy
    listen       8081;
    #server_name  domain2.com www.domain2.com;
    #access_log   logs/domain2.access.log  main;

    # serve static files
    # location ~ ^/(images|javascript|js|css|flash|media|static|html)/  {
    #   root    /usr/share/nginx/html;
    # }
    location = / {
        root /usr/share/nginx/html;
        index index.html;
    }

    location ~ ^\S*\.(html|css|js|png|jpg|gif)$ {
        root /usr/share/nginx/html;
    }


    # pass requests for dynamic content to rails/turbogears/zope, et al

    location ~ /v1/user {
      proxy_pass      http://user-service:8081;
    }
  }

  #upstream big_server_com {
  #  server 127.0.0.3:8000 weight=5;
  #  server 127.0.0.3:8001 weight=5;
  #  server 192.168.0.1:8000;
  #  server 192.168.0.1:8001;
  #}

  #server { # simple load balancing
  #  listen          80;
  #  server_name     big.server.com;
  #  access_log      logs/big.server.access.log main;

  #  location / {
  #    proxy_pass      http://big_server_com;
  #  }
  #}
}
}
```
nginx.cong主要参数我参考了nginx官方wiki的内容，见相关网站[2]。  
因为很多功能我目前用不上，所以就直接注释掉了，只保留了一些必须的内容，主要改动的是location的内容。再改动的过程中尝试花了不少时间，具体每个配置参数的作用以后有空了会专门写一篇博客或者wiki来研究，目前不花太多时间在这个上面。

## 修改docker-compose.yml
之后需要修改docker-compose.yml，添加对应的service描述：
```yml
# ui-service
ui-service:
  build: ui-service
  image: withme3.0/ui-service
  restart: always
  ports:
    - 80:8081
  networks:
    - your-network
```

之后按照正常程序build和up就可以了。

## 相关网站
1. DockerHub Nginx: https://hub.docker.com/_/nginx
2. Nginx Config:  https://www.nginx.com/resources/wiki/start/topics/examples/full/
