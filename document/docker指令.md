# docker指令

docker ps ：列出正在运行的容器

docker ps -a：显示所有的容器，包括未运行的

docker pull：拉取镜像

docker image：查看镜像

###### 建立目录挂在实现文件同步

docker container run -it -v /Users/xxxx/yourFilePath:/csapp --name=csapp_env centos /bin/bash ：

```
/Users/xxxx/yourFilePath 请替换成你自己想要进行同步的目录
:/csapp 也请替换成你自己想要命名的目录
```

###### 启动容器

docker start [CONTAINER ID] 

###### 进入一个已经在运行的容器

docker ps 

docker exec -it [CONTAINER ID]  /bin/bash