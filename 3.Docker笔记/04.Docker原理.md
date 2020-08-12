> 特点

Docker奖项都是只读的，当容器启动时， 一个新的可写层被加载到镜像的顶部！

这一层就是我们通常说的容器层， 容器之下的都叫做镜像层

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081215123458.png)



- commit镜像

```shell
docker commit 提交容器成为一个新的版本

# 命令和git 原理类似
docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名：[TAG]

docker commit -a="xiaofan" -m="add webapps app" d798a5946c1f tomcat007:1.0

```

实战测试

```shell
# 1. 启动一个默认的tomcat
# 2. 发现这个默认的tomcat是没有webapps应用， 镜像的原因，官方镜像默认webapps下面是没有内容的
# 3. 我自己拷贝进去了基本的文件
# 4. 将我们操作过的容器通过commit提价为一个镜镜像！我们以后就使用我们自己制作的镜像了
```



