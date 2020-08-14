# 1. Docker网络

## 链接Docker0

> 测试

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814091723905.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

三个网络

```shell
# 问题： docker是如何处理容器网络访问的？

# [root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker run -d -P --name tomcat01 tomcat

# 查看容器内部的网络地址 ip addr
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker exec -it tomcat01 ip addr， 发现容器启动的时候得到一个eth0@if115 ip地址，docker分配的！
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
114: eth0@if115: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

# 思考： linux 能不能ping通容器？
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.077 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.069 ms
64 bytes from 172.17.0.2: icmp_seq=3 ttl=64 time=0.075 ms

# linux 可以 ping 通docker容器内部！
```

> 原理

1. 我们没启动一个docker容器， docker就会给docker容器分配一个ip， 我们只要安装了docker，就会有一个网卡 docker0桥接模式，使用的技术是veth-pair技术！

再次测试 ip addr

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814092954929.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

2. 再启动一个容器测试， 发现又多了一对网卡

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814093432884.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

```shell
# 我们发现这个容器带来网卡，都是一对对的
# veth-pair 就是一对的虚拟设备接口，他们都是成对出现的，一端连着协议，一端彼此相连
# 正因为有这个特性，veth-pair充当一个桥梁， 连接各种虚拟网络设备
# OpenStac， Docker容器之间的链接，OVS的链接， 都是使用veth-pair技术
```

3. 我们测试一下tomcat01和tomcat02之间是否可以ping通！

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081409492726.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

结论：容器与容器之间是可以相互ping通的！

绘制一个网络模型图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814101617604.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

结论：tomcat01和tomcat02是共用的一个路由器，docker0

所有容器不指定网络的情况下，都是docker0路由的，doucker会给我们的容器分配一个默认的可用IP



> 小结

Docker使用的是Linux的桥接，宿主机中是一个Docker容器的网桥docker0.

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814102155735.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

Docker中的所有的网络接口都是虚拟的，虚拟的转发效率高！（内网传递文件！）

只要容器删除，对应的网桥一对就没有了！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814103808900.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

## -- link

> 思考一个场景，我们编写了一个微服务，database url =ip； 项目不重启，数据ip换掉了，我们希望可以处理这个问题，可以按名字来进行访问容器

```shell
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker exec -it tomcat02 ping tomcat01
ping: tomcat01: Name or service not known

# 如何可以解决呢？
# 通过--link既可以解决网络连通问题
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker run -d -P  --name tomcat03 --link tomcat02 tomcat
3a2bcaba804c5980d94d168457c436fbd139820be2ee77246888f1744e6bb473
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                     NAMES
3a2bcaba804c        tomcat              "catalina.sh run"   4 seconds ago       Up 3 seconds        0.0.0.0:32772->8080/tcp   tomcat03
f22ed47ed1be        tomcat              "catalina.sh run"   57 minutes ago      Up 57 minutes       0.0.0.0:32771->8080/tcp   tomcat02
9d97f93401a0        tomcat              "catalina.sh run"   About an hour ago   Up About an hour    0.0.0.0:32770->8080/tcp   tomcat01
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker exec -it tomcat03 ping tomcat02
PING tomcat02 (172.17.0.3) 56(84) bytes of data.
64 bytes from tomcat02 (172.17.0.3): icmp_seq=1 ttl=64 time=0.129 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=2 ttl=64 time=0.100 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=3 ttl=64 time=0.110 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=4 ttl=64 time=0.107 ms

# 反向可以ping通吗？
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker exec -it tomcat02 ping tomcat03
ping: tomcat03: Name or service not known

```

探究：inspect！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814104403761.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

其实这个tomcat03就是在本地配置了tomcat02的配置？

```shell
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker exec -it tomcat03 cat /etc/hosts
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.3	tomcat02 f22ed47ed1be
172.17.0.4	3a2bcaba804c
```

本质探究：--link 就是我们在hosts配置中增加了一个172.17.0.3	tomcat02 f22ed47ed1be

我们现在玩Docker已经不建议使用--link了！

自定义网络！不使用Docker0！

Docker0的问题：它不支持容器名链接访问！



## 自定义网络

> 查看所有的docker网络

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814105640929.png#pic_center)

**网络模式**

bridge： 桥接模式，桥接 docker 默认，自己创建的也是用brdge模式

none： 不配置网络

host： 和宿主机共享网络

container：容器网络连通！（用的少， 局限很大）

**测试**

```shell
# 我们直接启动的命令默认有一个 --net bridge，而这个就是我们的docker0
docker run -d -P --name tomcat01 tomcat
docker run -d -P --name tomcat01 --net bridge tomcat

# docker0特点，默认，容器名不能访问， --link可以打通连接！
# 我们可以自定义一个网络！
# --driver bridge
# --subnet 192.168.0.0/16 可以支持255*255个网络 192.168.0.2 ~ 192.168.255.254
# --gateway 192.168.0.1
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
26a5afdf4805d7ee0a660b82244929a4226470d99a179355558dca35a2b983ec
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
30d601788862        bridge              bridge              local
226019b14d91        host                host                local
26a5afdf4805        mynet               bridge              local
7496c014f74b        none                null                local
```

我们自己创建的网络就ok了！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814112009570.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

在自己创建的网络里面启动两个容器

```shell
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker run -d -P --name tomcat-net-01 --net mynet tomcat
0e85ebe6279fd23379d39b27b5f47c1e18f23ba7838637802973bf6449e22f5c
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker run -d -P --name tomcat-net-02 --net mynet tomcat
c6e462809ccdcebb51a4078b1ac8fdec33f1112e9e416406b606d0c9fb6f21b5
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker network inspect mynet
[
    {
        "Name": "mynet",
        "Id": "26a5afdf4805d7ee0a660b82244929a4226470d99a179355558dca35a2b983ec",
        "Created": "2020-08-14T11:12:40.553433163+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "0e85ebe6279fd23379d39b27b5f47c1e18f23ba7838637802973bf6449e22f5c": {
                "Name": "tomcat-net-01",
                "EndpointID": "576ce5c0f5860a5aab5e487a805da9d72f41a409c460f983c0bd341dd75d83ac",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            },
            "c6e462809ccdcebb51a4078b1ac8fdec33f1112e9e416406b606d0c9fb6f21b5": {
                "Name": "tomcat-net-02",
                "EndpointID": "81ecbc4fe26e49855fe374f2d7c00d517b11107cc91a174d383ff6be37d25a30",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]

# 再次拼连接
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker exec -it tomcat-net-01 ping 192.168.0.3
PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
64 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=0.113 ms
64 bytes from 192.168.0.3: icmp_seq=2 ttl=64 time=0.093 ms
^C
--- 192.168.0.3 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 999ms
rtt min/avg/max/mdev = 0.093/0.103/0.113/0.010 ms
# 现在不使用 --link也可以ping名字了！
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker exec -it tomcat-net-01 ping tomcat-net-02
PING tomcat-net-02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.068 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.096 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=3 ttl=64 time=0.094 ms

```

我们自定义的网络docker都已经帮我们维护好了对应的关系，推荐我们平时这样使用网络

好处：

redis - 不同的集群使用不同的网络，保证集群时安全和健康的

mysql - 不同的集群使用不同的网络，保证集群时安全和健康的

## 网络连通

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814114621170.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

测试打通tomcat01 和mynet

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081411482318.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

```shell
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker network connect  mynet tomcat01

# 连通之后就是讲tomcat01 放到了mynet网路下
# 一个容器两个ip地址：
# 阿里云服务器，公网ip，私网ip
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814115317846.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814115404720.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

```shell
# 连通ok
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker exec -it tomcat01 ping tomcat-net-01
PING tomcat-net-01 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.100 ms
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.085 ms
^C
--- tomcat-net-01 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.085/0.092/0.100/0.012 ms
# 依旧无法连通，没有connect
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker exec -it tomcat02 ping tomcat-net-01
ping: tomcat-net-01: Name or service not known

```

结论：假设要跨网络 操作别人，就要使用docker network connect连通.....!

## 实战：部署redis

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814124440671.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

```shell
# 创建网卡
docker network create redis --subnet 172.38.0.0/16

# 通过脚本创建六个redis配置
for port in $(seq 1 6); \
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done
# 创建结点1
docker run -p 6371:6379 -p 16371:16379 --name redis-1 \
-v /mydata/redis/node-1/data:/data \
-v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

#创建结点2
docker run -p 6372:6379 -p 16372:16379 --name redis-2 \
-v /mydata/redis/node-2/data:/data \
-v /mydata/redis/node-2/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.12 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
#创建结点3
docker run -p 6373:6379 -p 16373:16379 --name redis-3 \
-v /mydata/redis/node-3/data:/data \
-v /mydata/redis/node-3/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.13 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
#创建结点4
docker run -p 6374:6379 -p 16374:16379 --name redis-4 \
-v /mydata/redis/node-4/data:/data \
-v /mydata/redis/node-4/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.14 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
#创建结点5
docker run -p 6375:6379 -p 16375:16379 --name redis-5 \
-v /mydata/redis/node-5/data:/data \
-v /mydata/redis/node-5/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.15 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf
#创建结点6
docker run -p 6376:6379 -p 16376:16379 --name redis-6 \
-v /mydata/redis/node-6/data:/data \
-v /mydata/redis/node-6/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.16 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

# 创建集群
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker exec -it redis-1 /bin/sh
/data # ls
appendonly.aof  nodes.conf
/data # redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.38.0.15:6379 to 172.38.0.11:6379
Adding replica 172.38.0.16:6379 to 172.38.0.12:6379
Adding replica 172.38.0.14:6379 to 172.38.0.13:6379
M: 541b7d237b641ac2ffc94d17c6ab96b18b26a638 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
M: a89c1f1245b264e4a402a3cf99766bcb6138dbca 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
M: 259e804d6df74e67a72e4206d7db691a300c775e 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
S: 9b19170eea3ea1b92c58ad18c0b5522633a9e271 172.38.0.14:6379
   replicates 259e804d6df74e67a72e4206d7db691a300c775e
S: 061a9d38f22910aaf0ba1dbd21bf1d8f57bcb7d5 172.38.0.15:6379
   replicates 541b7d237b641ac2ffc94d17c6ab96b18b26a638
S: 7a16b9bbb0615ec95fc978fa62fc054df60536f0 172.38.0.16:6379
   replicates a89c1f1245b264e4a402a3cf99766bcb6138dbca
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
...
>>> Performing Cluster Check (using node 172.38.0.11:6379)
M: 541b7d237b641ac2ffc94d17c6ab96b18b26a638 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
M: a89c1f1245b264e4a402a3cf99766bcb6138dbca 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
S: 7a16b9bbb0615ec95fc978fa62fc054df60536f0 172.38.0.16:6379
   slots: (0 slots) slave
   replicates a89c1f1245b264e4a402a3cf99766bcb6138dbca
S: 061a9d38f22910aaf0ba1dbd21bf1d8f57bcb7d5 172.38.0.15:6379
   slots: (0 slots) slave
   replicates 541b7d237b641ac2ffc94d17c6ab96b18b26a638
M: 259e804d6df74e67a72e4206d7db691a300c775e 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 9b19170eea3ea1b92c58ad18c0b5522633a9e271 172.38.0.14:6379
   slots: (0 slots) slave
   replicates 259e804d6df74e67a72e4206d7db691a300c775e
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.


```

docker搭建redis集群完成！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814140307648.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

## SpringBoot微服务打包Docker镜像

1. 构建springboot项目

?	[IDEA2020 Ultimate版本激活方案](https://tech.souyunku.com/?p=11599) `亲测有效`

2. 打包应用
3. 编写Dockerfile

```shell
FROM java:8

COPY *.jar /app.jar

CMD ["--server.port=8080"]

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "/app.jar"]
```



4. 构建镜像

5. 发布运行！

```shell
# 把打好的jar包和Dockerfile上传到linux
[root@iZ2zeg4ytp0whqtmxbsqiiZ idea]# ll
total 16140
-rw-r--r-- 1 root root 16519871 Aug 14 17:38 demo-0.0.1-SNAPSHOT.jar
-rw-r--r-- 1 root root      122 Aug 14 17:38 Dockerfile

# 构建镜像，不要忘了最后有一个点
[root@iZ2zeg4ytp0whqtmxbsqiiZ idea]# docker build -t xiaofan666 .
Sending build context to Docker daemon  16.52MB
Step 1/5 : FROM java:8
8: Pulling from library/java
5040bd298390: Pull complete 
fce5728aad85: Pull complete 
76610ec20bf5: Pull complete 
60170fec2151: Pull complete 
e98f73de8f0d: Pull complete 
11f7af24ed9c: Pull complete 
49e2d6393f32: Pull complete 
bb9cdec9c7f3: Pull complete 
Digest: sha256:c1ff613e8ba25833d2e1940da0940c3824f03f802c449f3d1815a66b7f8c0e9d
Status: Downloaded newer image for java:8
 ---> d23bdf5b1b1b
Step 2/5 : COPY *.jar /app.jar
 ---> d4de8837ebf9
Step 3/5 : CMD ["--server.port=8080"]
 ---> Running in e3abc66303f0
Removing intermediate container e3abc66303f0
 ---> 131bb3917fea
Step 4/5 : EXPOSE 8080
 ---> Running in fa2f25977db7
Removing intermediate container fa2f25977db7
 ---> d98147377951
Step 5/5 : ENTRYPOINT ["java", "-jar", "/app.jar"]
 ---> Running in e1885e23773b
Removing intermediate container e1885e23773b
 ---> afb6b5f28a32
Successfully built afb6b5f28a32
Successfully tagged xiaofan666:latest

# 查看镜像
[root@iZ2zeg4ytp0whqtmxbsqiiZ idea]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
xiaofan666          latest              afb6b5f28a32        14 seconds ago      660MB
tomcat              latest              2ae23eb477aa        8 days ago          647MB
redis               5.0.9-alpine3.11    3661c84ee9d0        3 months ago        29.8MB
java                8                   d23bdf5b1b1b        3 years ago         643MB

# 运行容器
[root@iZ2zeg4ytp0whqtmxbsqiiZ idea]# docker run -d -P --name xiaofan-springboot-web xiaofan666
fd9a353a80bfd61f6930c16cd92204532bfd734e003f3f9983b5128a27b0375e
# 查看运行起来的容器端口（因为我们启动的时候没有指定）
[root@iZ2zeg4ytp0whqtmxbsqiiZ idea]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
fd9a353a80bf        xiaofan666          "java -jar /app.jar …"   9 seconds ago       Up 8 seconds        0.0.0.0:32779->8080/tcp   xiaofan-springboot-web
# 本地访问1
[root@iZ2zeg4ytp0whqtmxbsqiiZ idea]# curl localhost:32779
{"timestamp":"2020-08-14T09:42:57.371+00:00","status":404,"error":"Not Found","message":"","path":"/"}
# 本地访问2
[root@iZ2zeg4ytp0whqtmxbsqiiZ idea]# [root@iZ2zeg4ytp0whqtmxbsqiiZ idea]# curl localhost:32779/hello
hello, xiaofan
# 远程访问（开启阿里云上的安全组哦）
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200814175211444.png#pic_center)

以后我们使用了Docker之后，给别人交互的就是一个镜像即可！