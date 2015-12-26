# Samba

* Ubuntu安装

```
apt-get install samba sambin-common-bin
```* 配置在/etc/samba/smb.conf加入配置，以下配置为匿名全共享

```[share]    comment = Home's Share    path = /share    read only = no    public = yes    guest ok = yes    browseable = yes    writable = yessecurity = share
```

