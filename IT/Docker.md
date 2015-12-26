# Docker

镜像
>即只读的os原盘，比如类定义容器
>由镜像产生的实例，比如new出来的类

实例>仓库，实例的集合，比如数组列表* Ubuntu安装
```
sudo apt-get install –y docker.io
```
* 启动
```
sudo service docker start```

* 帮助```docker –h```
* 搜索镜像

```sudo docker search ubuntu```
* 下载镜像

```sudo docker pull ubuntu```
* 用images查看镜像

```xiaoye@xiaoye-VirtualBox:~$ sudo docker images[sudo] password for xiaoye: REPOSITORY          TAG                 IMAGE ID            CREATED             VIRTUAL SIZEubuntu              latest              ca4d7b1b9a51        3 weeks ago         187.9 MB```
* 使用镜像，这时候就进入了虚拟机

```xiaoye@xiaoye-VirtualBox:~$ sudo docker run -t -i ubuntu /bin/bashroot@df56f1b36981:/#```exit退出，一旦退出虚拟机就停了在对虚拟机进行任何形式的改动后，可以想git那样保存镜像，比如

```sudo docker commit -m 'installed python' df56f1b36981sudo docker commit -m 'installed python' df56f1b36981 ubuntu_pythonsudo docker commit -m 'installed python' df56f1b36981 ubuntu_python:v1```上述命令分别产生备份（不带名字，带名字，带名字和标签）

```xiaoye@xiaoye-VirtualBox:~$ sudo docker imagesREPOSITORY          TAG                 IMAGE ID            CREATED              VIRTUAL SIZEubuntu_python       v1                  a9bf63a0f895        2 seconds ago        211.4 MBubuntu_python       latest              5ce3d714c9f9        About a minute ago   211.4 MB<none>              <none>              514ba865aecc        About a minute ago   211.4 MBubuntu              latest              ca4d7b1b9a51        3 weeks ago          187.9 MB```
* 用DockerFile创建镜像，类似于打补丁，装机，因为你总是在一个**干净**的**源盘**上操作

```# This is a comment FROM UbuntuMAINTAINER Docker Newbee newbee@docker.comRUN apt-get –qq updateRUN ….```
    FROM 用来指明那个镜像    RUN 用来跑命令，相当于在这台虚拟机上执行命令
    ADD 命令来复制本地文件到虚拟机中    ADD myApp /var/www    EXPOSE 80  向外部开放端口    CMD 指定容器启动后运行的程序在DockerFile文件所在目录下跑

```sudo docker build –t=”ubuntu:v2” . 
```或指定DockerFile路径

```sudo docker build –t=”ubuntu:v2” /path/to/dockerfile```产生一个执行过指定命令的镜像，名字叫ubuntu 标签是v2docker tag 用来修改镜像标签

```sudo docker tag <xxxxx> <name:tag>
```如

```
sudo docker tag 5db5f84733 newOs:V1```
* 从本地导入镜像

```sudo cat ubuntu-xxx.tar.gz | docker import – Ubuntu```>Ubuntu-xxx.tar.gz 是用openvz技术制作的包* 上传镜像，类似git push```sudo docker push Ubuntu```
* 把镜像输出到本地

```sudo docker save –o ubuntu.tar ubuntu
```* 载入

```sudo docker load –input ubuntu.tar 
```或

```sudo docker load < Ubuntu.tar```* 删除镜像```docker rmi ubuntu
```
>在删之前要用docker rm删除掉容器实例

