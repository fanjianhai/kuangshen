# 1. 容器数据卷

## 1.1. docker的理解回顾

将应用和环境打包成一个镜像！

数据？如果数据都在容器中，那么我们容器删除，数据就会丢失！`需求：数据可以持久化`

MySQL，容器删了，删库跑路！`需求：MySQL数据可以存储在本地！`

容器之间可以有一个数据共享技术！Docker容器中产生的数据，同步到本地！

这就是卷技术，目录的挂载，将我们容器内的目录挂载到linux目录上面！



**总结： **容器的持久化和同步操作！容器间数据也是可以共享的！



## 1.2. 使用数据卷

> 方式一：直接使用命令来挂载 -v	

```shell
docker run -it -v 主机目录：容器目录

[root@iZ2zeg4ytp0whqtmxbsqiiZ home]# docker run -it -v /home/ceshi:/home centos /bin/bash
```



![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812165141715.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

**测试文件的同步**（在主机上改动，观察容器变化）

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081216585980.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

**再来测试**（测试通过）

1. 停止容器

2. 主机上修改文件

3. 启动容器

4. 容器内的数据依旧是同步的！

   

   
## 1.3. 实战：安装MySQL

思考：MySQL的数据持久化的问题！

```shell
# 获取镜像
[root@iZ2zeg4ytp0whqtmxbsqiiZ home]# docker pull mysql:5.7

# 运行容器， 需要做数据挂载！ # 安装启动mysql，需要配置密码（注意）
# 官方测试， docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag

# 启动我们的
-d		# 后台运行
-p		# 端口隐射
-v		# 卷挂载
-e		# 环境配置
--name	# 容器的名字
[root@iZ2zeg4ytp0whqtmxbsqiiZ home]# docker run -d -p 3344:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
9552bf4eb2b69a2ccd344b5ba5965da4d97b19f2e1a78626ac1f2f8d276fc2ba

# 启动成功之后，我们在本地使用navicat链接测试一下
# navicat链接到服务器的3344 --- 3344 和 容器的3306映射，这个时候我们就可以连接上mysql喽！

# 在本地测试创建一个数据库，查看下我们的路径是否ok！
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200812174632321.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

   

   

   

 ## 1.4. 匿名和具名挂载

  ```shell
# 匿名挂载
-v 容器内路径
docker run -d -P --name nginx01 -v /etc/nginx nginx		# -P 随机指定端口

# 查看所有volume的情况
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker volume ls
DRIVER              VOLUME NAME
local               561b81a03506f31d45ada3f9fb7bd8d7c9b5e0f826c877221a17e45d4c80e096
local               36083fb6ca083005094cbd49572a0bffeec6daadfbc5ce772909bb00be760882

# 这里发现，这种情况就是匿名挂载，我们在-v 后面只写了容器内的路径，没有写容器外的路径！

# 具名挂载
 [root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx
26da1ec7d4994c76e80134d24d82403a254a4e1d84ec65d5f286000105c3da17
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                   NAMES
26da1ec7d499        nginx               "/docker-entrypoint.…"   3 seconds ago       Up 2 seconds        0.0.0.0:32769->80/tcp   nginx02
486de1da03cb        nginx               "/docker-entrypoint.…"   3 minutes ago       Up 3 minutes        0.0.0.0:32768->80/tcp   nginx01
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker volume ls
DRIVER              VOLUME NAME
local               561b81a03506f31d45ada3f9fb7bd8d7c9b5e0f826c877221a17e45d4c80e096
local               36083fb6ca083005094cbd49572a0bffeec6daadfbc5ce772909bb00be760882
local               juming-nginx

# 通过-v 卷名：容器内的路径
# 查看一下这个卷
# docker volume inspect juming-nginx

[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker volume inspect juming-nginx
[
    {
        "CreatedAt": "2020-08-12T18:15:21+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/juming-nginx/_data",
        "Name": "juming-nginx",
        "Options": null,
        "Scope": "local"
    }
]
  ```

所有docker容器内的卷，没有指定目录的情况下都是在`/var/lib/docker/volumes/xxxxx/_data`

我们通过具名挂载可以方便的找到我们的一个卷，大多数情况下使用的是`具名挂载`

```shell
# 如何确定是具名挂载还是匿名挂载，还是指定路径挂载！
-v	容器内路径					# 匿名挂载
-v	卷名:容器内路径			   # 具名挂载
-v /主机路径:容器内路径			  # 指定路径挂载
```

拓展

```shell
# 通过 -v 容器内容路径 ro rw 改变读写权限
ro	readonly	# 只读
rw	readwrite	# 可读可写

docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:rw nginx

# ro 只要看到ro就说明这个路径只能通过宿主机来操作，容器内容无法操作
```



   

   

