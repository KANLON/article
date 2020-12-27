### 概述

由于自己的之前学习 nginx 只会简单使用，然后每次配置 nginx 都要找文档去了解怎么配置，有点麻烦，所以这里记录下一些常用的nginx 配置和配置的例子，到时候直接 copy 修改即可。基本上这些都是工作之后都会需要用到的配置，我在工作中用到挺多次的🤣，但是基本我每次都会忘记的，Nginx 对应的配置应该怎么写。


nginx 的主要功能为 静态文件的服务器、负载均衡、重写或重定向url、正向代理、反向代理、隐藏文件的实际后缀 等。


这里使用的 nginx 版本为 1.16.0


配置文件的主要结构为：

nginx.conf
```
http{

  # 这个用于负载均衡的配置
  upstream backend {
    server backend1.example.com       weight=5;
    server backend2.example.com:8080;
    server backup2.example.com:8080   backup;
  }
  # 服务配置1，我们一般只要修改这里的信息即可
  server{
    listen 8080;
    location / {
      proxy_pass http://localhost:8080;
    }
  }
  
  # 这个 server 是服务配置2
  server{
    listen 80;
    location ~ \.(gif|jpg|png)$ {
      proxy_pass http://backend;
    }
  }
  # includ 用来引入 nginx 在其他目录的配置文件，一般正式的公司项目会这样使用，并以每个域名或 server 块分成每个配置文件。
  include cust_conf/mainconf; # 加多一个确定的文件，这样找不到文件的时候会报错，以免配置错目录 
  includ /data/host/config/*.conf;
}

```


### 启动相关

#### window 启动 

cmd命令进入安装文件；

1、启动：`C:\server\nginx-1.0.2>start nginx` 或`C:\server\nginx-1.0.2>nginx.exe`

注：建议使用第一种，第二种会使你的cmd窗口一直处于执行中，不能进行其他命令操作。

参考：[https://www.jianshu.com/p/01f3626cf25d](https://www.jianshu.com/p/01f3626cf25d)


#### nginx 命令行参数 

不像许多其他软件系统，Nginx 仅有几个命令行参数，完全通过配置文件来配置

`-c </path/to/config>` 为 Nginx 指定一个配置文件，来代替缺省的。

`-t` 不运行，而仅仅测试配置文件。nginx 将检查配置文件的语法的正确性，并尝试打开配置文件中所引用到的文件。用法：`nginx -t`， 这个命令也可以查看nginx文件的所在位置。

`-v` 显示 nginx 的版本。

`-V` 显示 nginx 的版本，编译器版本和配置参数。

#### 启动、停止 和重新加载配置

通过执行 nginx 的可执行文件可以直接启动 nginx。只要 nginx 启动了，则可以通过执行 nginx 加 `-s` 参数来控制 nginx 的一些行为。用法如下：
```
nginx -s ${signal}
```
${signal} 可以为以下四个值：
* stop — fast shutdown 快速停止
* quit — graceful shutdown  优雅停止
* reload — reloading the configuration file 重新加载配置文件
* reopen — reopening the log files 重新打开日志文件

例如：可以通过执行 `nginx -s reload` 来重新加载配置文件，使更改过的配置文件生效。



### 静态文件的服务器

&emsp;&emsp;前端文件或者图片服务的部署，一般会使用到这个功能，通过 nginx 服务器来分发（sering out）文件，然后用户从网络上能够通过 ip 或 域名直接访问到


#### 1. 实现访问 stats-server.kanlon.com 实现将 `/data/fr` 下的目录作为静态文件访问

```conf
server {
    # 访问端口81，并且访问的域名为 stats-server.kanlon.com 使用该配置
    listen 81;
    server_name stats-server.kanlon.com;
    charset utf8;
    # 设置自动查找index文件为false，避免安全性问题
    autoindex off;
    # 设置展示首页的文件
    index  index.html index.htm index.php;
    # 如果是/img/ 路径，则转发到专门存放图片的路径/data/img/中 
    location /img/ {
       alias /data/img/;
    }
    location / {
        root /data/fr;
    }
}

```

如果访问出现错误，可以看一下 logs/error.log 的错误日志，这里会打印出实际访问的文件路径。还有测试 nginx 的时候一定要注意是否启动了多个 nginx 否则可能更改了配置会以为自己设置错误不生效的（沉重教训）

### 负载均衡
通过 nginx 可以实现将请求自动转发都自己的指定的服务域名上，减少单台服务请求量和实现服务的高可用。

nginx 上默认是通过 [ngx_http_upstream_module](http://nginx.org/en/docs/http/ngx_http_upstream_module.html) 模块实现

#### 1. 配置访问 load-balancing-test.kanlon.com:83 则负载均衡到指定三个域名上

负载的三个域名地址：load-balancing-test-1.kanlon.com:84，load-balancing-test-2.kanlon.com:85，load-balancing-test-3.kanlon.com:86 

其中 要求 `load-balancing-test-3.kanlon.com:86` 作为备份服务，load-balancing-test.kanlon.com-1:84 和load-balancing-test-2.kanlon.com:85 的请求数分布比例为 1:2

```conf
# 配置健康检查，当为502，503，504，404的时候表示服务不可用，60秒内有两个这样的失败请求则负载到另一个服务上
proxy_next_upstream http_502 http_503 http_504 http_404 error timeout invalid_header;
upstream balancing {
    server load-balancing-test-1.kanlon.com:84 max_fails=1 fail_timeout=60s  weight=1;
    server load-balancing-test-2.kanlon.com:85 max_fails=2 fail_timeout=60s  weight=2;
    server load-balancing-test-3.kanlon.com:86 backup;
}
server {
    # 访问端口83，并且访问的域名为 load-balancing-test.kanlon.com 使用该配置
    listen 83;
    server_name load-balancing-test.kanlon.com;
    # 设置自动查找index文件为false，避免安全性问题
    autoindex off;
    # 设置展示首页的文件
    index  index.html index.htm index.php;
    location / {
        proxy_pass http://balancing;
    }
}
    
server {
    # 访问端口84，并且访问的域名为 load-balancing-test-1.kanlon.com 使用该配置
    listen 84;
    server_name load-balancing-test-1.kanlon.com;
    charset utf8;
    # 设置自动查找index文件为false，避免安全性问题
    autoindex off;
    # 设置展示首页的文件
    index  index.html index.htm index.php;
    location / {
        # 通过在目录后面加上没有文件的目录可以模拟服务不可用
        root /data/nginx/load-balancing/load-balancing-test-1;
    }
}
server {
    # 访问端口85，并且访问的域名为 load-balancing-test-2.kanlon.com 使用该配置
    listen 85;
    server_name load-balancing-test-2.kanlon.com;
    charset utf8;
    # 设置自动查找index文件为false，避免安全性问题
    autoindex off;
    # 设置展示首页的文件
    index  index.html index.htm index.php;
    location / {
        # 通过在目录后面加上没有文件的目录可以模拟服务不可用
        root /data/nginx/load-balancing/load-balancing-test-2;
    }
}
server {
    # 访问端口86，并且访问的域名为 load-balancing-test-3.kanlon.com 使用该配置
    listen 86;
    server_name load-balancing-test-3.kanlon.com;
    charset utf8;
    # 设置自动查找index文件为false，避免安全性问题
    autoindex off;
    # 设置展示首页的文件
    index  index.html index.htm index.php;
    location / {
        root /data/nginx/load-balancing/load-balancing-test-3;
    }
}

```

其中的包含了一些 nginx 的健康检查的指令

`max_fails=number` 设定Nginx与服务器通信的尝试失败的次数。在fail_timeout参数定义的时间段内，如果失败的次数达到此值，Nginx就认为服务器不可用。在下一个fail_timeout时间段，服务器不会再被尝试。 失败的尝试次数默认是1。设为0就会停止统计尝试次数，认为服务器是一直可用的。 你可以通过指令proxy_next_upstream、fastcgi_next_upstream和 memcached_next_upstream来配置什么是失败的尝试。 默认配置时，http_404状态不被认为是失败的尝试。

`fail_timeout=time`  设定服务器被认为不可用的时间段以及统计失败尝试次数的时间段。在这段时间中，服务器失败次数达到指定的尝试次数，服务器就被认为不可用。

### 重写或重定向 url

nginx 可以将匹配的 url 重定向到另外的 url 去，包含改变地址

#### 指定地址重定向
将 `rewrite-local.kanlon.com/rewrite/same/site/**` 重定向到 `rewrite-local.kanlon.com/rewrite2/same/site/**`  和 将 `rewrite-local.kanlon.com/rewrite/other/site` 重定向到 `rewrite-other.kanlon.com/rewrite2/same/site`


```conf
server {
    # 访问端口88，并且访问的域名为 rewrite-local.kanlon.com 使用该配置
    listen 88;
    server_name rewrite-local.kanlon.com;
    charset utf8;
    # 设置自动查找index文件为false，避免安全性问题
    autoindex off;
    # 设置展示首页的文件
    index  index.html index.htm index.php;

    # 转发到本域名的其它路径
    location /rewrite/same/site {
        rewrite ^/rewrite/same/site(.*)$ /rewrite2/same/site$1 last;
    }
    
    # 转发到另外的域名，如果要 重定向，; 前面加上 permanent （永久重定向 301 即可），默认为 302 临时重定向
    location /rewrite/other/site {
        rewrite ^/rewrite/other/site(.*)$ http://rewrite-other.kanlon.com:89/rewrite2/same/site$1 permanent;
    }

    location /rewrite2/same/site {
       default_type text/html;
       return 200 "$request_uri";
    }

    location / {
        root /data/nginx/rewrite-test/rewriter-local;
    } 
}

server {
    # 访问端口89，并且访问的域名为 rewrite-other.kanlon.com 使用该配置
    listen 89;
    server_name rewrite-other.kanlon.com;
    charset utf8;
    # 设置自动查找index文件为false，避免安全性问题
    autoindex off;
    # 设置展示首页的文件
    index  index.html index.htm index.php;

    location /rewrite2/same/site {
       default_type text/html;
       return 200 "$request_uri";
    }
    location / {
        root /data/nginx/rewrite-test/rewriter-other;
    } 
}
```


### 正向代理

正向代理，就是好像我们平常使用的 vpn 那样，我们的访问的域名不会变，只不过通过 nginx 来帮我们发送请求和接收请求，并返回自己客户端。nginx 默认支持 http 协议的正向代理，如果要支持 https 需要安装组件。


nginx 配置如下：
```conf
server {
    listen       90;
    server_name  forward-agent.kanlon.com;
    # 这里配置 DNS 服务器的IP地址
    resolver 8.8.8.8;
    location / {
        proxy_pass http://$http_host$request_uri;
    }
}
```
然后，再在自己电脑上配置代理服务器和端口，即可完成正向代理。

### 反向代理

反向代理，则直接访问 nginx 的域名来替代访问自己原来的要访问资源的域名，然后得到原资源的域名返回的结果


```conf
server {
    listen       91;
    server_name  reverse-proxy.kanlon.com;
  
    #设置代理
    location / {
        proxy_pass http://127.0.0.1:8080;
    }
}
```

### 隐藏文件的实际后缀
有时候我们想让访问静态html文件的时候，需要隐藏掉后缀`.html`，可以像下面这样配置

```conf
server {
    listen 81;
    charset utf8;
    autoindex off;
    index  index.html index.htm index.php;
    try_files $uri $uri/ /index.html?$query_string;
    # 如果是以html或者 htm 结尾，则直接返回对应文件，以免引起nginx循环查找
    location ~ \.(htm|html)$ {
      root  /data/nginx_static;
    }
    location / {
      # 如果访问的文件不存在，则通过访问时添加后缀来隐藏URL中的后缀
      if (!-e $request_filename){
         rewrite ^(.*)$ /$1.html last; 
         break;
      } 
      root  /data/nginx_static;
    }
    
}
```

这样的话，假设项目根目录下有`login.html` 文件，原只能通过访问 `127.0.0.1/login.html` 访问到，设置之后，通过 `127.0.0.1/login` 或者  `127.0.0.1/login.html` 都能访问到


### 参考文档
1. [nginx 官方文档](http://nginx.org/en/docs/)
2. [nginx 中文文档](https://www.nginx.cn/doc/)
3. [Nginx官网翻译](https://segmentfault.com/a/1190000003708211)
4. [Windows下Nginx的启动、停止、重启等命令](https://www.jianshu.com/p/01f3626cf25d)
5. [Nginx正向代理与反向代理](https://www.jianshu.com/p/ae76c223c6ef)
6. [nginx 常见正则匹配符号表示](https://www.cnblogs.com/netsa/p/6383094.html)
7. [Nginx实战系列之功能篇----后端节点健康检查](https://blog.51cto.com/nolinux/1594029)
8. [ nginx 可视化界面配置](https://github.com/digitalocean/nginxconfig.io)
9. [让nginx上的静态网页在访问的时候没有html后缀](https://my.oschina.net/AnnaWu/blog/3120272)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201114143851114.png)
