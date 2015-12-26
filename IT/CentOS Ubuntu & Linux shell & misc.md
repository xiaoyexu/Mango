# CentOS Ubuntu & Linux shell & misc
查看Linux版本

```lsb_releaselsb_release –a```或者

```cat /etc/redhat-release```或者```rpm -q redhat-release```
>CentOS7 yum使用python2.7版本，升级python3可能会有问题在虚拟机上(CentOS7最小版)**如果没有ifconfig用``yum search ifconfig``再install如果没有lsb_release 运行``yum install redhat-lsb -y``**如果远程ssh有问题，用``ssh -v user@server``查看错误
如果是``rsa key not found``，查看``/var/log/messages`` ，可能提到sshd error could not load host key检查``/etc/ssh/ssh_host_rsa_key`` 这类文件是否为空，如果为空，用命令``ssh-keygen -t rsa  -f /etc/ssh/ssh_host_rsa_key``生成即可在虚拟机上virtualbox，使用bridge模式，用来在主机同一个网中虚拟此机器
>好比虚拟机在内网里，用nat模式，设置端口转发表，比如主机的127.0.0.1 2222 -> 10.0.2.15 22，这样用ssh root@127.0.0.1 来访问虚拟机CentOS7使用firewalld防火墙服务，如果外部（或虚拟机主机）无法访问其服务（如httpd,django），需要关闭或配置firewalld* 对已登录用户发消息``wall ‘message’``* 对特定用户发消息``write <user> pts/1````pts/1 ttyname``可以用命令w查看``echo‘test’ | write somebody pts/1````shutdown now````shutdown–r now``* yum 命令
```yum install <package>yum–y install <package>    -- answer yes for allyum provides <command>yum whatprovides <command>   -- 查找含有某个命令或文件的包yum clean all 清除yum仓库信息
```* scp 命令
```scp txt maven@192.168.56.102:~/ 把当前txt文件传到远程服务器上```
* 设置时间
```date -s "2012-08-24 15:03:30"```* 取网页```wget http://someurl 
```输出到文件

```wget -O <file> <url>```

* find 命令```find -name ‘abc’ -o -name ‘xyz’     -o orfind -name ‘abc’ -exec ls -l {} \;find -name ‘abc’ -ok ls -l {} \;      -ok与-exec 相同但是需要用户输入回应find / -type f -print | xargs grep "device"find /yourpath -mtime +366 -exec rm {} \;在当前目录下所有文件中查找内容包含 string 的文件:find ./ -name "*" -exec grep "string" {} \;注意:在最后不能加 print ,否则会出错. 在当前目录下所有文件中查找内容包含 string 的文件并列出字符所在的文件:find ./ -name "*" -exec grep -l "string" {} \; 在当前目录下 *.c 中查找内容包含 string 的文件并列出字符所在的文件的所在行(不显示文件名):find ./ -name "*.c" -exec grep -n "string" {} \; 在当前目录下所有文件中查找内容包含 string 的文件并列出字符所在的文件,所在行及所在行的内容:find ./ -name "*" -exec grep -n "string" ./ {} \; 使用 find 查找时希望忽略某个目录(-prune)：如果希望在 /app 目录下查找文件，但不希望在 /app/bin 目录下查找：find /app -name "/app/bin" -prune -o -print 使用 type 选项：如果要在 /etc 目录下查找所有的目录：find /etc -type d -print 如果要在 /etc 目录下查找 .svn 的目录：find /etc -name .svn -type d -print为了在当前目录下查找除目录以外的所有类型的文件：find . ! -type d -print 为了在当前目录下查找所有的符号链接文件，可以用：find . -type l -print 为了用 ls -l 命令列出所匹配到的文件，可以把 ls -l 命令放在find命令的 -exec 选项中：find . -type f -exec ls -l {} \; 注： f 表示普通文件    exec 选项后面跟随着所要执行的命令，然后是一对 {}，一个空格和一个 \，最后是一个分号。```* VBOX 复制vdi磁盘

```C:\Program Files\Oracle\VirtualBox>VBoxManage.exe clonehd "c:\dev\CentOS.vdi" "c:\dev\CentOS2.vdi"```
* 修改vdi磁盘uuid

>路径有空格时需加引号

```D:\Program Files\Oracle\VirtualBox>VBoxManage internalcommands sethduuid E:\VirtualBox\Win7_Ultimate_SP1_1\Win7_Ultimate_SP1.vdi  ```

* netstat 查看

```netstat -nlp | grep 8080netstat -nlp | grep LISTEN```

* vi 搜索替换

```：s/vivian/sky/ 替换当前行第一个 vivian 为 sky ：s/vivian/sky/g 替换当前行所有 vivian 为 sky ：n，$s/vivian/sky/ 替换第 n 行开始到最后一行中每一行的第一个 vivian 为 sky  ：n，$s/vivian/sky/g 替换第 n 行开始到最后一行中每一行所有 vivian 为 sky 　n 为数字，若 n 为 .，表示从当前行开始到最后一行 ：%s/vivian/sky/（等同于：g/vivian/s//sky/）替换每一行的第一个 vivian 为 sky ：%s/vivian/sky/g（等同于：g/vivian/s//sky/g）替换每一行中所有 vivian 为 sky 　　可以使用 # 作为分隔符，此时中间出现的 / 不会作为分隔符 ：s#vivian/#sky/# 替换当前行第一个 vivian/ 为 sky/ ：%s+/oradata/apras/+/user01/apras1+ （使用+ 来替换 / ）： /oradata/apras/替换成/user01/apras1/ 　 1.：s/vivian/sky/ 替换当前行第一个 vivian 为 sky 　：s/vivian/sky/g 替换当前行所有 vivian 为 sky 2. ：n，$s/vivian/sky/ 替换第 n 行开始到最后一行中每一行的第一个 vivian 为 sky 　：n，$s/vivian/sky/g 替换第 n 行开始到最后一行中每一行所有 vivian 为 sky 　　（n 为数字，若 n 为 .，表示从当前行开始到最后一行） 3. ：%s/vivian/sky/（等同于：g/vivian/s//sky/）替换每一行的第一个 vivian 为 sky 　：%s/vivian/sky/g（等同于：g/vivian/s//sky/g）替换每一行中所有 vivian 为 sky 4. 可以使用 # 作为分隔符，此时中间出现的 / 不会作为分隔符 　：s#vivian/#sky/# 替换当前行第一个 vivian/ 为 sky/ 5. 删除文本中的^M 　　问题描述：对于换行，window下用回车换行（0A0D）来表示，linux下是回车（0A）来表示。这样，将window上的文件拷到unix上用时，总会有个^M.请写个用在unix下的过滤windows文件的换行符（0D）的shell或c程序。 　　。使用命令：cat filename1 | tr -d “^V^M” > newfile； 　　。使用命令：sed -e “s/^V^M//” filename > outputfilename.需要注意的是在1、2两种方法中，^V和^M指的是Ctrl+V和Ctrl+M.你必须要手工进行输入，而不是粘贴。 　　。在vi中处理：首先使用vi打开文件，然后按ESC键，接着输入命令：%s/^V^M//. 　　。：%s/^M$//g 　　如果上述方法无用，则正确的解决办法是： [Page]　　。 tr -d \"\\r\" < src >dest 　　。 tr -d \"\\015\" dest 　　。 strings A>B 6. 替换确认         我们有很多时候会需要某个字符(串)在文章中某些位置出现时被替换，而其它位置不被替换的有选择的操作，这就需要用户来进行确认，vi的查找替换同样支持       例如      ：s/vivian/sky/g 替换当前行所有 vivian 为 sky       在命令后面加上一个字母c就可以实现，即：s/vivian/sky/gc      顾名思意，c是confirm的缩写7. 其它 　利用：s 命令可以实现字符串的替换。具体的用法包括： 　：s/str1/str2/ 用字符串 str2 替换行中首次出现的字符串 str1 　：s/str1/str2/g 用字符串 str2 替换行中所有出现的字符串 str1 　：。，$ s/str1/str2/g 用字符串 str2 替换正文当前行到末尾所有出现的字符串 str1 　：1，$ s/str1/str2/g 用字符串 str2 替换正文中所有出现的字符串 str1 　：g/str1/s//str2/g 功能同上 　　从上述替换命令可以看到：g 放在命令末尾，表示对搜索字符串的每次出现进行替换；不加 g，表示只对搜索 　　字符串的首次出现进行替换；g 放在命令开头，表示对正文中所有包含搜索字符串的行进行替换操作%s/[ ]\{8\}/\t/g  把8个空格换成\t5,10s%^%//%g     把5到10行开头换成//，用了分隔符%5,10s%^%#%g     换成#```* Mac隐藏文件
显示：

```
defaults write com.apple.finder AppleShowAllFiles -bool true
```
隐藏：

```
defaults write com.apple.finder AppleShowAllFiles -bool false
```

* watch

```watch –d –n1 command
```>刷新command命令结果，-d高亮变化，-n刷新秒数* top CPU使用率

```使用权限：所有使用者使用方式：top [-] [d delay] [q] [c] [S] [s] [i] [n] [b]说明：即时显示process的动态d :改变显示的更新速度，或是在交谈式指令列( interactive command)按sq :没有任何延迟的显示速度，如果使用者是有superuser的权限，则top将会以最高的优先序执行c :切换显示模式，共有两种模式，一是只显示执行档的名称，另一种是显示完整的路径与名称S :累积模式，会将己完成或消失的子行程( dead child process )的CPU time累积起来s :安全模式，将交谈式指令取消,避免潜在的危机i :不显示任何闲置(idle)或无用(zombie)的行程n :更新的次数，完成后将会退出topb :批次档模式，搭配"n"参数一起使用，可以用来将top的结果输出到档案内```
**EPEL的全称叫 Extra Packages for Enterprise Linux**

```yum install epel-release
```或者

```cd /tmpwget https://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpmls *.rpmsudo yum install epel-release-7-5.noarch.rpm```

* 用户相关命令

```1、建用户：adduser phpq                            //新建phpq用户passwd phpq                            //给phpq用户设置密码2、建工作组groupadd test                          //新建test工作组3、新建用户同时增加工作组useradd -g test phpq               //新建phpq用户并增加到test工作组注：：-g 所属组 -d 家目录 -s 所用的SHELL4、给已有的用户增加工作组usermod -G groupname username或者：gpasswd -a user group5、临时关闭：在/etc/shadow文件中属于该用户的行的第二个字段（密码）前面加上*就可以了。想恢复该用户，去掉*即可。或者使用如下命令关闭用户账号：passwd peter –l重新释放：passwd peter –u6、永久性删除用户账号userdel petergroupdel peterusermod –G peter peter   （强制删除该用户的主目录和主目录下的所有文件和子目录）7、从组中删除用户编辑/etc/group 找到GROUP1那一行，删除 A 或者用命令 gpasswd -d A GROUP8、显示用户信息id usercat /etc/passwd补充:查看用户和用户组的方法用户列表文件：/etc/passwd用户组列表文件：/etc/group查看系统中有哪些用户：cut -d : -f 1 /etc/passwd查看可以登录系统的用户：cat /etc/passwd | grep -v /sbin/nologin | cut -d : -f 1查看用户操作：w命令(需要root权限)查看某一用户：w 用户名查看登录用户：who查看用户登录历史记录：last```

>添加用户到组

```
sudo addgroup xiaoyegroup sudo adduser --ingroup xiaoyegroup xiaohua #添加一个xiaohua用户到xiaoyegroup组里
```

* cron job```service crond startcrontab –u <user> -elr-e 编辑cron job table-l 显示-r 删除Cron job table就是一个文本文件* * * * * command to be executed| | | | || | | | ----- Day of week (0 - 7) (Sunday=0 or 7)| | | ------- Month (1 - 12)| | --------- Day of month (1 - 31)| ----------- Hour (0 - 23)------------- Minute (0 - 59)```>比如* * * * * a.sh   每分钟执行a.sh10 * * * * a.sh  每小时，每个第10分钟*/5 * * * * a.sh  每5分钟30 7 * * * a.sh   每天7:3030 7 * * 0 a.sh   每周日7:3030 7 1 * * a.sh   每月1日7:30* sudoer
 >让普通用户使用sudo命令，可以在/etc/sudoers添加，比如

```# Members of the admin group may gain root privileges%admin ALL=(ALL) ALLxiaoye ALL=(ALL) ALL```

* tee 从标准输入中获得输入输出到文件
``tee abc.txt`` 所有输入转到文件abc.txt中，如果文件存在会覆盖，ctrol+D结束``tee –a abc.txt`` 追加到文件abc.txt，类似 >> abc.txt* curl 发送http请求

比如``curl 'http://127.0.0.1/test'``如果是https，可设置insecure flag，比如```curl -k -insecure -H "Authorization:<encrypted stirng>" 'https://someurl'```

