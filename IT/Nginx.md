# Nginx

* Ubuntu安装```
sudo apt-get install nginx
```* centos

```
yum instal nginx
```
或从nginx.org下载tar包

```tar -zxvf nginx.xxx.tar.gz```>./configure如遇到c compile cc is not found等错误，安装相应缺失包
>
```yum -y install gcc gcc-c++ autoconf automakeyum -y install pcre pcre-develyum -y install zlib zlib-devel
```./configure没有错误后，make, make install成功安装后会有/usr/local/nginx目录
* conf 配置文件* html 网页* logs 日志* sbin 执行程序启动命令，以某个配置文件启动，取决于具体配置/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf命令有可能在/usr/sbin下
配置文件在/etc/nginx下以下为1.6版本内容，通过yum install nginx安装配置文件nginx.conf

``` # For more information on configuration, see: #   * Official English Documentation: http://nginx.org/en/docs/ #   * Official Russian Documentation: http://nginx.org/ru/docs/user nginx;   #nginx用户，安装时系统会创建一个nginxworker_processes auto;   #进程数，可为cpu核数或核数两倍#日志文件路径，1.8版有 notice和info#比如error_log /var/log/nginx/error.log notice; 或error_log /var/log/nginx/error.log info;error_log /var/log/nginx/error.log;    pid /run/nginx.pid;  #pid是系统控制文件，以.pid结尾events {    worker_connections 1024;  #最大连接数}http {    # 1.8版有gzip on功能，将response压缩后传给用户    # 日志格式    # main 或者combined，格式的key    # $remote_addr 客户ip地址     # $request 请求url    # $status 请求状态    # $http_referer 原网页    # $http_user_agent 用户浏览器信息    # $http_x_forwarded_for X-Forwarded-For:简称XFF头，它代表客户端，也就是HTTP的请求端真实的IP，只有在通过了HTTP 代理或者负载均衡服务器时才会添加该项    # 比如 192.168.1.103 - - [03/Sep/2015:21:52:55 +0800] "GET /favicon.ico HTTP/1.1" 404 570 "http://192.168.1.109:8080/" "Mozilla/5.0 (Windows NT 6.1; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/44.0.2403.157 Safari/537.36"    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '                      '$status $body_bytes_sent "$http_referer" '                      '"$http_user_agent" "$http_x_forwarded_for"';    access_log  /var/log/nginx/access.log  main;  #main对应于log_format的main    sendfile            on;    tcp_nopush          on;    tcp_nodelay         on;    keepalive_timeout   65;    types_hash_max_size 2048;        include             /etc/nginx/mime.types;    default_type        application/octet-stream;    # Load modular configuration files from the /etc/nginx/conf.d directory.    # See http://nginx.org/en/docs/ngx_core_module.html#include    # for more information.    include /etc/nginx/conf.d/*.conf;    server {        listen       80 default_server;        listen       [::]:80 default_server;        server_name  _;        root         /usr/share/nginx/html;        # Load configuration files for the default server block.        include /etc/nginx/default.d/*.conf;        location / {        }        error_page 404 /404.html;            location = /40x.html {        }        error_page 500 502 503 504 /50x.html;            location = /50x.html {        }    }}```* 虚拟机配置
**配置IP**ifconfig获得主机ip地址，或使用命令配置``ifconfig eth0 192.168.1.9 netmask 255.255.255.0``给eth0设备指定ip地址配置子设备，eth0:后跟任意数字，192.168.1.255为eth0的broadcast地址
以下配置两个设备

```ifconfig eth0:1 192.168.1.7 broadcast 192.168.1.255 netmask 255.255.255.0ifconfig eth0:2 192.168.1.17 broadcast 192.168.1.255 netmask 255.255.255.0
```**新建配置文件**
在主配置文件nginx中引入比如``vi new.conf``

```user nobody;worker_processes 4;events{  worker_connections 1024;}http{  server{    listen 192.168.1.7:80; #监听地址    server_name 192.168.1.7; #其他名字亦可    access_log logs/server1.access.log combined; #combined 默认格式，关闭用access_log off;    location / {      index index.html index.htm;  #默认首页      root html/server1;  #实际文件系统目录，是nginx安装目录下的html，在此目录下有server1目录，其下有index.html，或者直接指定 /var/www/html/server1    }  }  server{    listen 192.168.1.17:80; #监听地址    server_name 192.168.1.17;    access_log logs/server2.access.log combined;    location / {      index index.html index.htm;      root html/server2    }  }}```
**加载new.conf**```/usr/sbin/nginx -c /etc/nginx/new.conf
```
>通过访问ip地址访问网页，192.168.1.7或192.168.1.17**缓存配置**在server段中

```location / {  index index.html index.htm;  root html/server2}location ~ .*\.(jpg|png|swf|gif)${   #所有后缀为jpg或png或swf或gif的文件缓存两天  expires 2d;}location ~ .*\.(css|js)${  expires 1h;}```
gzip压缩，去掉或者gzip off为关闭在http段下```http {  gzip on;  gzip_min_length 1k; #最小压缩大小  gzip_buffers 4 16k; #内存缓存大小，表示4个16k的数据空间  gzip_http_version 1.1; #压缩http版本  gzip_vary on; #判断客户浏览器是否支持gzip压缩  ...}```
自动列目录在server段中```server{  ...  autoindex on;}
```##反向代理，映射>比如将80端口/api映射到8080端口的/cbws上去，这样可以在80，8080端口上使用不同的web服务

```location /api {proxy_pass    http://121.40.113.6:8080/cbws;proxy_redirect default ;}```
完整的server段配置在http段中，如是单独文件，可以在http段中使用``include /etc/nginx/sites-enabled/*;``引入
以下是个默认80为静态页面服务器例子

```server {listen 80 default_server;listen [::]:80 default_server ipv6only=on;root /project/www;index index.html index.htm; # Make site accessible from http://localhost/server_name localhost;location / {	# First attempt to serve request as file, then	# as directory, then fall back to displaying a 404.	try_files $uri $uri/ =404;	# Uncomment to enable naxsi on this location	# include /etc/nginx/naxsi.rules}location /api {proxy_pass    http://121.40.113.6:8080/cbws;proxy_redirect default ;}}
```


