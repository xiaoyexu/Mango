# uwsgi
可配合nginx使用


```


>在/etc/uwsgi下有apps-available，apps-enabled目录

```cat caibird.ini 

/var/run/uwsgi/app/下会产生一个caibird（caibird.ini去掉扩展名）的目录内有pid和socket文件

```

``service uwsig start``和``service nginx start``后可以通过nginx访问django项目
