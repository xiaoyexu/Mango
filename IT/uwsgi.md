# uwsgi
可配合nginx使用
* Ubuntu安装

```apt-get install uwsgi uwsgi-core uwsgi-plugin-python```* 配置uwsgi，可能根据版本不同而不同


>在/etc/uwsgi下有apps-available，apps-enabled目录在apps-available下创建配置文件，然后软链接到apps-enabled目录下内容例如

```cat caibird.ini [uwsgi]socket = /var/run/uwsgi/app/caibird/socketchmod-socket = 666limit-as = 256processes = 6max-request = 2000memory-report = trueenable-threads = truepythonpath = /project/Caibirdchdir = /project/Caibird/Caibirdwsgi-file = /project/Caibird/Caibird/wsgi.pypost-buffering = 8192```
>post-buffering = 8192 用来设置post data缓存，不设置可能会造成服务器中断、挂起**注意**
/var/run/uwsgi/app/下会产生一个caibird（caibird.ini去掉扩展名）的目录内有pid和socket文件这个目录需要配置到nginx中，比如

```server {	listen 8080 default_server;	listen [::]:8080 default_server ipv6only=on;	# Make site accessible from http://localhost/	server_name localhost;	location / {		include uwsgi_params;		uwsgi_pass unix:///var/run/uwsgi/app/caibird/socket;	}}```
启动
``service uwsig start``和``service nginx start``后可以通过nginx访问django项目

