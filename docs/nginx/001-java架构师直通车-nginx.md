# java 架构师直通车-nginx

* 反向代理
* 静态服务

<!-- ![image-20211020192847551](./001-nginx-images/image-20211020192847551.png) -->
<img src="./001-nginx-images/image-20211020192847551.png" alt="image-20211020192847551" style="zoom:100%;" />
## 官网文档

http://nginx.org/en/docs/

## 安装

> https://www.cnblogs.com/pxstar/p/14808244.html

114.132.221.117 机器上

~~~shell
运行命令目录:
/home/soft/nginx/objs/nginx
配置文件在:
/usr/local/nginx/conf
~~~

mac 电脑安装地址为

~~~shell
/usr/local/nginx/
~~~

## 命令

> ~~~shell
> #查看版本号 
> ./nginx -v
> #查看版本号 ,当前ng 的详细信息(log,pid..)
> ./nginx -V
> #检查配置
> ./nginx -t
> #热加载配置
> ./nginx -s reload
> # 关闭
> ./nginx -s quit
> #暴力立即关闭
> ./nginx -s stop
> # 帮助
> ./nginx -h
> ./nginx -?
> # 指定特定配置文件
> ./nginx -c 指定nginx.conf文件
> 
> ~~~
> > 打包成makefile 命令
>
> ~~~shell
> ./configure \
> --prefix=/usr/local/nginx \
> --pid-path=/var/run/nginx/nginx.pid \
> --lock-path=/var/lock/nginx.lock \
> --error-log-path=/var/log/nginx/error.log \
> --http-log-path=/var/log/nginx/access.log \
> --with-http_gzip_static_module \
> --http-client-body-temp-path=/var/temp/ngnix/client \
> --http-proxy-temp-path=/var/temp/ngnix/proxy \
> --http-fastcgi-body-temp-path=/var/temp/ngnix/fastcgi \
> --http-uwsgi-body-temp-path=/var/temp/ngnix/uwsgi \
> --http-scgi-body-temp-path=/var/temp/ngnix/scgi \
> ~~~
>
<!-- > ![image-20211022172558813](./001-nginx-images/image-20211022172558813.png) -->
> <img src="./001-nginx-images/image-20211022172558813.png" alt="image-20211022172558813" style="zoom:50%;" />
> 
>
> 

## nginx.conf

> <img src="./001-nginx-images/image-20211022170145013.png" alt="image-20211022170145013" style="zoom:50%;" />
>
> 默认server
>
>
> ~~~shell
> #服务的定义
> server {
> 			  #访问进来的时候,监听80端口,
>   listen       80;
>   #请求本机80 端口 反向代理的ip地址
>   server_name  localhost;
>   #映射,  / 表示 root 
>   location / {
>  		 	# root 为ng 安装目录   html 为ng 安装目录下的html文件夹 
>       root   html;
>       # index 为主页 访问${ng_home}/html/index.html 页面
>       index  index.html index.htm;
>   }
>   #浏览器发生500 等 访问50x.html
>   error_page   500 502 503 504  /50x.html;
>   location = /50x.html {
>       root   html;
>   }
> }
> ~~~

## 进程模型-异步非阻塞

> * 以下标注为==client== 表示浏览器的请求
>
> ~~~shell
> [root@VM-8-14-centos sbin]# ps -ef |grep nginx
> root      7214     1  0 15:25 ?        00:00:00 nginx: master process ./nginx                                                                  
> nobody    7215  7214  0 15:25 ?        00:00:00 nginx: worker process                                                                          
> root     13933 22097  0 15:59 pts/0    00:00:00 grep --color=auto nginx                                                                        
> ~~~
>
> 反应到nginx.conf 上的配置为:
>
> ~~~shell
> #一个worker 进程
> worker_processes  1;
> ~~~
>
> 
>
> <img src="./001-nginx-images/image-20211022160702085.png" alt="image-20211022160702085" style="zoom:50%;" />
>
> <img src="./001-nginx-images/image-20211022160817905.png" alt="image-20211022160817905" style="zoom:50%;" />
>
> ### worker的抢占机制
>
> <img src="./001-nginx-images/image-20211022162625121.png" style="zoom:50%;" />
>
> ## 传统事件处理机制和ng的对比
>
> 
>
> <img src="./001-nginx-images/image-20211022164356136.png" alt="image-20211022164356136" style="zoom:50%;" />
>
> 
>
> <img src="./001-nginx-images/image-20211022163248742.png" style="zoom:50%;" />
>
> 
>
> * 传统处理请求是为bio ,而n g 为异步非阻塞io `epoll`  (epoll 在linux 下较为常见的使用,在其他系统中,由ng判断选择)
>
> 反应到nginx.conf 上的配置为:
>
> ~~~shell
> events {
>     #默认使用epoll
>     user epoll
>     #每个worker 最大请求连接数
>     worker_connections  1024;
> }
> ~~~

## nginx.conf-详细配置说明

> ~~~shell
> # 设置启动的工作线程 worker 的用户一般为 nobody  可以提升用户权限使用root 添加访问linux 的读写权限
> #user  nobody;
> # 工作线程 worker 的工作数量
> worker_processes  1;
> 
> # 使用错误的日志保存地址 
> #日志等级 由低到高: debug info notice warn error crit
> #error_log  logs/error.log;
> #error_log  logs/error.log  notice;
> #error_log  logs/error.log  info;
> 
> # 启动ng 的进程文件
> #pid        logs/nginx.pid;
> 
> #ng 事件处理配置
> events {
>  #默认使用epoll
>  user epoll
>  #每个worker 最大请求连接数  可以修改成10240
>  worker_connections  1024;
> }
> 
> #http 配置
> http {
>  # vi ${ng_home}/conf/mime.types 其文件为定义的http 请求类型
>  include       mime.types;
>  #默认的请求类型
>  default_type  application/octet-stream;
> 
>  #日志格式化  remote_addr:远程客户端地址 remote_user:远程客户端名称  time_local 时间
>  #           request  请求类型  status 状态   body_bytes_sent 客户端发送的body 
>  #           http_referer  记录用户从哪个网站跳转过来  http_user_agent 用户的代理   http_x_forwarded_for 通过代理的服务器记录的客户端的ip 
>  #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
>  #                  '$status $body_bytes_sent "$http_referer" '
>  #                  '"$http_user_agent" "$http_x_forwarded_for"';
> 
>  #请求的日志存放地址
>  #access_log  logs/access.log  main;
> 
>  # 文件传输
>  sendfile        on;
>  # 文件传输 时 数据包累计到一定数量的数据在进行发送 一般和  sendfile 配合使用
>  #tcp_nopush     on;
> 
>  #客户端连接服务器的心跳超时时间,不使用可以设置为0 ,下面配置为 65 单位为s ,
>  #当一个请求在65s 内完成了 此时没有关闭tcp连接 ,当一个新的请求过来时,就可以使用这个tcp 连接,提高效率
>  #keepalive_timeout  0;
>  keepalive_timeout  65;
> 
>  #压缩  传输的内容(js css..) 使传输的效率提高  但是会消耗服务器cpu性能
>  #gzip  on;
> 
> #服务的定义
>  server {
>      #访问进来的时候,监听80端口,
>      listen       80;
>      #请求本机80 端口 反向代理的ip地址
>      server_name  localhost;
>      #映射,  / 表示 root 
>      location / {
>          # root 为ng 安装目录   html 为ng 安装目录下的html文件夹 
>          root   html;
>          # index 为主页 访问${ng_home}/html/index.html 页面
>          index  index.html index.htm;
>      }
>      #浏览器发生500 等 访问50x.html
>      error_page   500 502 503 504  /50x.html;
>      location = /50x.html {
>          root   html;
>      }
>  }
> 
> }
> ~~~
>
> * upstream 负载均衡
>
> ~~~shell
> upstream nginx_servers{
> 		server eddy01:80 max_fails=2 fail_timeout=2 weight=1;
> 		server eddy02:80 max_fails=2 fail_timeout=2 weight=4;
> 		server eddy03:80 max_fails=2 fail_timeout=2 weight=4;
> 	}
> 
> 	upstream tomcat_servers{
> 		server 192.168.145.109:9090 weight=1;
> 		server 192.168.145.110:9090 weight=4;
> 		server 192.168.145.111:9090 weight=4;
> 	}
> 
> #服务的定义
> 	server {
> 		listen       80;
> 		server_name  localhost;
> 		access_log off ;
> 		# ~* 不区分大小写  \.(png|html|js|css)$  以.png|html.. 结尾的请求转发到  nginx_servers 服务
> 		location ~* \.(png|html|js|css)$ {
> 			proxy_pass http://nginx_servers;
> 		}
> 		location / {
> 			proxy_pass http://tomcat_servers;
> 		}
> 	}
> ~~~
>
> 练习
>
> * 新家一个server 监听 81端口 访问自定义的页面
>
> *  把新加的server 提取到一个新的文件取名 xxx.conf   使用 `include` 关键字导入到这个ngnix.conf 文件上
>
>   通过   `./nginx -t` 和 `./nginx -s reload` 来重启ng  访问自定义页面

## nginx 常见问题

### nginx.pid 文件不存在

> <img src="./001-nginx-images/image-20211022182459825.png" alt="image-20211022182459825" style="zoom:50%;" />
>
> ~~~shell
> mkdier /var/run/nginx
> ./nginx -t
> ./nginx -s reload
> ~~~
>
> 如果还是不行的话
>
> ~~~shell
> ./nginx -h
> ./nginx -c 指定nginx.conf文件
> ./nginx -s reload
> ~~~
>
> 另一种方案,使用ng 默认的 pid 存放路径 nginx.conf打开注释
>
> ~~~shell
> pid        logs/nginx.pid;
> ~~~

## 日志切割

### 手动

 针对 access.log 文件过大的问题进行日志切割

1,创建一个shell可执行文件:nginx-cut-log.sh ，内容为:

~~~shell
#!/bin/bash
LOG_PATH="/usr/local/nginx/logs"
#每 1分种进行日志切割
RECORD_TIME=$(date -d "yesterday" +%Y-%m-%d+%H:%M)
PID=/usr/local/nginx/logs/nginx.pid
mv ${LOG_PATH}/access.log ${LOG_PATH}/access-${RECORD_TIME}.log
mv ${LOG_PATH}/error.log ${LOG_PATH}/error-${RECORD_TIME}.log
#向nginx 主进程发送信号,用于重新打开日志文件
kill -USR1 `cat $PID`
~~~

2,添加可执行权限

~~~shell
chmod a+x nginx-cut-log.sh
~~~

3,测试日志切割后的结果

~~~shell
./nginx-cut-log.sh
~~~

### 自动

1,安装定时任务:

~~~shell
yum install crontabs
~~~

2,crontab -e 添加一行新的任务，为了测试方便每分钟运行这个任务

~~~shell
*/1 * * * * nginx-cut-log.sh
~~~

<!-- ![image-20211029142447059](./001-nginx-images/image-20211029142447059.png) -->
<img src="./001-nginx-images/image-20211029142447059.png" alt="image-20211029142447059" style="zoom:50%;" />

修改后重启

~~~shell
service crond restart
~~~

主要命令

~~~shell
service crond start //启动服务
service crond stop //关闭服务
service crond restart //重启服务
service crond reload //重新载入配置
~~~

## 处理静态资源

~~~shell
# 访问接口90 ng 反向代理本地路径为 /usr/local/nginx/foodie-shop 的index.html 
server {
        listen       90;
        server_name  localhost;
        location / {
            root   /usr/local/nginx/foodie-shop;
            index  index.html;
        }
        
        location /imooc {
            root   /usr/local/nginx/foodie-shop;
        }
    }
~~~

### 别名

适用于 不想做路径拼接的情况下 ,暴露地址

~~~shell
# 访问接口90 ng 反向代理本地路径为 /usr/local/nginx/foodie-shop 的index.html 
server {
        listen       90;
        server_name  localhost;
        location / {
            root   /usr/local/nginx/foodie-shop;
            index  index.html;
        }
        
        #访问 /imooc 路径的 统一去  /home/imooc 下去找
        location /imooc {
            root   /home;
        }
        
        #访问 /static 路径的 统一去  /home/imooc 下去找  
        #等效于上面的 location
        location /static {
            alias   /home/imooc;
        }
    }
~~~

## Gzip

 使用Gzip压缩提升请求效率!

~~~shell
#开启 gzip 压缩功能,目的:提高传输效率,节约带宽
gzip  on;
#限制最小压缩,小于1字节的文件不回被压缩
gzip_min_length 1;
#定义压缩级别(压缩比,取值[1-9],一般不设置太大,文件越大,压缩越多,cpu 使用就会越多 )
gzip_comp_level 3
#定义压缩文件的类型
gzip_types text/plain application/javascript application/x-javascript text/html text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png application/json
~~~

## location-正则表达式

location,路径匹配规则

~~~shell
server {
        listen       90;
        server_name  localhost;
        # 所有请求
        location / {
					...
        }
        
        # = 精确匹配
        location = / {
					...
        }
        # = 精确匹配 到  /home/imooc/img/face1.png
        location = /imooc/img/face1.png {
					root /home
        }
        
				# ~* 不区分大小写  \.(png|html|js|css)$  以.png|html.. 结尾的请求转发到  nginx_servers 服务
        location ~* \.(png|html|js|css)$ {
          proxy_pass http://nginx_servers;
        }
        
        # ^~ 以某一个字符开头 以/imooc/img 开头  去访问 /home/imooc/img
        location ^~* /imooc/img {
          root /home
        }
        
}
~~~

## nginx的跨域问题的解决

<img src="./001-nginx-images/image-20211101160525343.png" alt="image-20211101160525343" style="zoom:50%;" />

在server 里面添加

~~~shell
    server {
        listen       80;
        server_name  localhost;

        #允许跨域请求的域,*代表所有
        add_header ’Access-Control-Allow-Origin ’ ‘*’;
        #允许带上cookie 请求
        add_header ‘Access-Control-Allow-Credentials’ ’true’;
        #允许请求的方法,比如 GET/POST/PUT/DELETE
        add_header ‘Access-Control-Allow-Methods’ ‘*’;
        #允许请求的header
        add_header ‘Access-Control-Allow-Headers’ ‘*’;

				...
				}
~~~

## 防盗链

防止非指定站点对资源的盗用(非法站点的非法请求)

~~~shell
server {
        listen       80;
        server_name  localhost;
				
				#对站点的验证
       	valid_referers *.imooc.com;
       	#非指定站点的非法请求 返回404
       	if($valid_referers)
					return 404;
				...
				}
~~~

## 集群和负载均衡

### upstream 

见`nginx.conf-详细配置说明`

### 项目结构

<img src="./001-nginx-images/image-20211108111030851.png" alt="image-20211108111030851" style="zoom:50%;" />

### nginx.conf:

~~~shell
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #开启 gzip 压缩功能,目的:提高传输效率,节约带宽
    gzip  on;

    upstream tomcat_servers{
        server 114.132.221.117:8080 weight=1;
        server 114.132.221.117:8081 weight=1;
    }

    server {
        listen       80;
        server_name  localhost;

        #允许跨域请求的域,*代表所有
        add_header ’Access-Control-Allow-Origin’ ‘*’;
        #允许带上cookie 请求
        add_header ‘Access-Control-Allow-Credentials’ ’true’;
        #允许请求的方法,比如 GET/POST/PUT/DELETE
        add_header ‘Access-Control-Allow-Methods’ ‘*’;
        #允许请求的header
        add_header ‘Access-Control-Allow-Headers’ ‘*’;

        #access_log  logs/host.access.log  main;

        location / {
            root   /home/soft/foodie-shop;
            index  index.html;
        }

         location /foodie-api {
            proxy_pass http://tomcat_servers;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
~~~

### 负载均衡的方式

#### 轮询: 

nginx 默认的负载均衡方式,

验证:在每个被代理的3个tomcat 下的ROOT/index.jsp 中修改添加 tomcat1~3 的`<h1>` 标识,访问展示.依次显示是 tomcat1,tomcat2,tomcat3

#### 权重轮询:

~~~shell
upstream tomcat_servers{
				#八个请求会有一个被处理
        server 114.132.221.117:8080 weight=1;
				#八个请求会有2个被处理
        server 114.132.221.117:8081 weight=2;
				#八个请求会有5个被处理
        server 114.132.221.117:8081 weight=5;
    }
    
    
    server eddy01:80 max_fails=2 fail_timeout=2 weight=1;
		server eddy02:80 max_fails=2 fail_timeout=2 weight=4;
		server eddy03:80 max_fails=2 fail_timeout=2 weight=4;
~~~

### upstream 参数

~~~shell
max_conns:
最大连接数,当连接数大于最大连接数时,报错,一般不设置.
eg:
		server eddy01:80 max_conns=60

slow_start:
缓慢启动,配置开启时,启动期间 开启监控程序   
eg:
	server eddy01:80 slow_start=60s
报错,ng 需要是商业版

down:
	server 下线,不可用
eg:
	server eddy01:80 down

backup:
	备份,备用机,只有其他的服务器(被代理的服务器,如tomcat或者nginx)挂了  他才可以被访问
eg:
	server eddy01:80 backup

max_fails:
访问错误次数,如果ng错误次数超过这个数值,会被ng 剔除.被视为服务器挂了  与 fail_timeout 一起使用
fail_timeout:
在指定时间中  max_fails 超过指定数值,.被视为服务器挂了.   直到第二个单位时间 失败数值在规定数值内 ,才不被视为服务器挂了
eg:
	server eddy01:80 max_fails=2 fail_timeout=2s;
~~~



## keepalived

提高吞吐量!

吞吐量是指对网络、设备、端口、虚电路或其他设施，单位时间内成功地传送数据的数量（以比特、字节、分组等测量）

使用:

~~~shell
 upstream tomcat_servers{
        server 114.132.221.117:8080 weight=1;
        #server 114.132.221.117:8081 weight=1;

        #数值为32,为32个长连接  keepalive 提高吞吐量的原理是 #使一些连接变成
        #长连接(连接开启的时间长,当连接处理完当前请求时,这个连接还没关不,又一个请求过来时,继续使用之前未关闭的连接),
        #避免连接关闭在开启,造成资源的开销
        keepalive 32
    }
    server{
				location /foodie-api {
            proxy_pass http://tomcat_servers;
            #http 默认版本为1.0 这里设置成1.1
            proxy_http_version 1.1;
            #清除 连接 Connection 的信息
            proxy_set_header Connection "";
        }
    }
~~~

使用 JMeter 测试吞吐量!

## 负载均衡原理

### ip_hash

<img src="./001-nginx-images/image-20211108145720708.png" alt="image-20211108145720708" style="zoom:50%;" />

hash(ip) % node_counts = index

使用

~~~shell
upstream backend {
    ip_hash;

    server 114.132.221.117:8080 weight=1;
    server 114.132.221.117:8081 weight=1;
}
~~~

由于ng 中 只对访问ip[114.132.221.117] 的前 3个网段,即[114.132.221]做hash 运算. 所以  114.132.221 网段访问的都是同一个服务器(tomcat)  

验证:在每个被代理的3个tomcat 下的ROOT/index.jsp 中修改添加 tomcat1~3 的<h1> 标识,访问展示.访问的都是 tomcat1

ng 源码:  nginx/src/http/modules/ngx_http_upstream_ip_hash_module.c 中

~~~c
#取  114.132.221  3个网段
iphp->addrlen = 3;

for (i = 0; i < (ngx_uint_t) iphp->addrlen; i++) {
  hash = (hash * 113 + iphp->addr[i]) % 6271;
}
~~~

<img src="./001-nginx-images/image-20211108154228743.png" alt="image-20211108154228743" style="zoom:50%;" />

需要注意的是 

* 由于ip_hash 是恒定请求到固定tomcat 下,如果有人使用恶意攻击,会对固定tomcat 造成一定的性能损耗.

* 当一台机器宕了,我们需要临时去移除的话,我们是不能修改conf, 删除机器ip 而是标记这台机器为`down`  ,否则,ng 会重新算hash,在运行中的那些用户的会话就会拿不到

```shell
upstream backend {
    ip_hash;

    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com down;
    server backend4.example.com;
}
```

### 一致性hash

 <img src="./001-nginx-images/image-20211108160426221.png" alt="image-20211108160426221" style="zoom:50%;" />



~~~shell
原理:
1,定义了 0~2的32次方 圆环
2,先对服务器进行hash 取值(ip,主机名),对应圆环的几个点(几个数值)
3,当用户请求过来时 对应hash值 是处于 哪个阶段((0,节点1值],..(3,节点4值]) 上 找到节点服务器进行访问.
4,新增和已有服务器宕机的情况依旧是就近原则
~~~

### url_hash

<img src="./001-nginx-images/image-20211108165728654.png" alt="image-20211108165728654" style="zoom:50%;" />

~~~shell
    upstream tomcat_servers{
        #对url 进行hash  $request_uri为nginx 提供的
        hash $request_uri;
        server 114.132.221.117:8080 weight=1;
        server 114.132.221.117:8081 weight=1;
    }
~~~



### least_conn

<img src="./001-nginx-images/image-20211108171028694.png" style="zoom:50%;" />

* 选择 tomcat 连接数最小的的 tomcat3 进行负载

~~~shell
upstream tomcat_servers{
        #对url 进行hash  $request_uri为nginx 提供的
				least_conn;
        server 114.132.221.117:8080 weight=1;
        server 114.132.221.117:8081 weight=1;
    }
~~~



## 缓存

~~~shell
缓存分为两类:
1,静态资源缓存
2,缓存上游的服务器 (即处理上游服务器的返回值)
~~~

<img src="./001-nginx-images/image-20211110104758739.png" alt="image-20211110104758739" style="zoom:50%;" />

### 1,静态页面缓存

>* ng对静态页面默认是支持缓存的,浏览器返回 `304` 表示为缓存中获取的页面.
>
>* 浏览器里可以设置禁用缓存  返回 `200` 状态码
>
>### expires 指令
>
>* expires [time] 
>
>~~~shell
>location /static {
>	alias   /home/soft/imooc;
>	expires 10s;
>}
>~~~
>
>访问:http://114.132.221.117/static/cache.html
>
><img src="./001-nginx-images/image-20211110101452327.png" alt="image-20211110101452327" style="zoom:50%;" />
>
>* expires @[time] 
>
>~~~shell
>location /static {
>	alias   /home/soft/imooc;
>	# 设置在 22点30分缓存失效
>	expires @22h30m;
>}
>~~~
>
>* expires -[time] 
>
>~~~shell
>location /static {
>	alias   /home/soft/imooc;
>	# 设置在当前时间的前一个小时,缓存已经失效, Cache-Control: no-cache
>	expires -1h;
>}
>~~~
>
>* expires epoch
>
>~~~shell
>不用缓存
>~~~
>
>* expires off
>
>~~~shell
>默认设置,不启用缓存
>~~~
>
>* expires max
>
>~~~shell
>设置缓存过期时间为最大
>~~~
>
><img src="./001-nginx-images/image-20211110103426868.png" alt="image-20211110103426868" style="zoom:50%;" />
>
>

### 2,缓存反向代理服务器(tomcat)

> 使用upstream 来做反向代理负载均衡
>
> ~~~shell
> 中间遇到问题:
> The character [_] is never valid in a domain name.
> 原因:
> upstream tomcat_servers{
> 改成:
> upstream tomcatServers{
> ~~~
>
> 改成:
>
> ~~~shell
> http {
> 	
>   upstream tomcatServers{
>     server 114.132.221.117:8080 weight=1;
>     server 114.132.221.117:8081 weight=1;
>   }
> 
> 	server {
>         listen       80;
>         server_name  localhost;
>         location / {
>                 proxy_pass http://tomcatServers;
>         }
> }
> ~~~
>
> 在http{} 模块中添加
>
> ~~~shell
> #proxy_cache_path 设置缓存路径
> #keys_zone 设置共享内存和占用空间大小
> #max_size 设置缓存大小
> #inavtive 超过此时间则被清理
> #use_temp_path 设置临时目录,使用后会影响性能,这里设置为off
> proxy_cache_path /usr/local/nginx/upstream_cache keys_zone=mycache:5m max_size=200m inactive=1m use_temp_path=off;
> ~~~
>
> 可以在server 中添加 也可以在localhost 中添加
>
> ~~~shell
> #开启缓存,和keys_zone 命名一致
> proxy_cache mycache;
> #针对 200 和304 状态码缓存时间为8h;
> proxy_cache_valid 200 304 8h;
> ~~~
>
> 验证:
>
> 在tomcat容器中的webapps 中添加静态文件夹 imooc,其中有个图片资源:
>
> ~~~shell
> /imooc/img/face1.png
> ~~~
>
> 访问:
>
> ~~~shell
> http://114.132.221.117/imooc/img/face1.png
> ~~~
>
> 查看: 
>
> /usr/local/nginx/upstream_cache 文件夹,由于我们访问的是方向代理 tomcat 的资源 这个资源已经被nginx 缓存到了这个文件下
>
> ~~~shell
> [root@VM-8-14-centos upstream_cache]# pwd
> /usr/local/nginx/upstream_cache
> [root@VM-8-14-centos upstream_cache]# ll                                                                                                       
> total 52
> -rw------- 1 nobody nobody 50501 Nov 10 14:32 23e2c51965006e04667d43e444954917
> ~~~

## 配置https

> 重新配置nginx
>
> ~~~shell
> ./configure \
> --prefix=/usr/local/nginx \
> --pid-path=/var/run/nginx/nginx.pid \
> --lock-path=/var/lock/nginx.lock \
> --error-log-path=/var/log/nginx/error.log \
> --http-log-path=/var/log/nginx/access.log \
> --with-http_gzip_static_module \
> --http-client-body-temp-path=/var/temp/ngnix/client \
> --http-proxy-temp-path=/var/temp/ngnix/proxy \
> --http-fastcgi-body-temp-path=/var/temp/ngnix/fastcgi \
> --http-uwsgi-body-temp-path=/var/temp/ngnix/uwsgi \
> --http-scgi-body-temp-path=/var/temp/ngnix/scgi \
> #安装ssl 模块
> --with-http_ssl_module
> ~~~
>
> 编译和安装
>
> ~~~shell
> make
> make install
> ~~~
>
> 下载证书,以腾讯云为例:
>
> ![image-20211110152622308](./001-nginx-images/image-20211110152622308.png)
>
> 解压获取nginx 下的ssl 证书 `.crt`和私钥 `.key` 拷贝到`/usr/local/nginx/conf` 下
>
> 新增 server 监听 443 端口(腾讯云文档)
>
> <img src="./001-nginx-images/image-20211110152956814.png" alt="image-20211110152956814" style="zoom:50%;" />

## 动静分离