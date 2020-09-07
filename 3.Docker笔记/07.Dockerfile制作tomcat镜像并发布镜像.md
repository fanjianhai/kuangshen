## Dockerfile制作tomcat镜像

1. 准备镜像文件 tomcat压缩包，jdk的压缩包！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813164403261.png#pic_center)

2. 编写Dockerfile文件，官方命名`Dockerfile`, build会自动寻找这个文件，就不需要-f指定了！

```shell
[root@iZ2zeg4ytp0whqtmxbsqiiZ tomcat]# cat Dockerfile 
FROM centos
MAINTAINER xiaofan<594042358@qq.com>

COPY readme.txt /usr/local/readme.txt

ADD jdk-8u73-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.37.tar.gz /usr/local/

RUN yum -y install vim

ENV MYPATH /usr/local
WORKDIR $MYPATH

ENV JAVA_HOME /usr/local/jdk1.8.0_73
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.37
ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.37
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib:$CATALINA_HOME/bin

EXPOSE 8080

CMD /usr/local/apache-tomcat-9.0.37/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.37/bin/logs/catalina.out

```

3. 构建镜像

```shell
# docker build -t diytomcat .
```

4. 启动镜像

```shell
#  docker run -d -p 3344:8080 --name xiaofantomcat1 -v /home/xiaofan/build/tomcat/test:/usr/local/apache-tomcat-9.0.37/webapps/test -v /home/xiaofan/build/tomcat/tomcatlogs/:/usr/local/apache-tomcat-9.0.37/logs diytomcat
```

5. 访问测试

6. 发布项目（由于做了卷挂载， 我们直接在本地编写项目就可以发布了）

**在本地编写web.xml和index.jsp进行测试**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813180242294.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

```shell
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.4" 
    xmlns="http://java.sun.com/xml/ns/j2ee" 
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee 
        http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd">
        
</web-app>
```

```shell
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>hello. xiaofan</title>
</head>
<body>
Hello World!<br/>
<%
System.out.println("-----my test web logs------");
%>
</body>
</html>
```

发现：项目部署成功， 可以直接访问ok！

我们以后开发的步骤：需要掌握Dockerfile的编写！ 我们之后的一切都是使用docker进行来发布运行的！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813175909845.png#pic_center)

## 发布自己的镜像到Docker Hub

>  Docker Hub

1. [地址](https://hub.docker.com/) 注册自己的账号！

   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813182851607.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

2. 确定这个账号可以登录

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813183151439.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)



3. 在我们的服务器上提交自己的镜像

```shell
# push到我们的服务器上
[root@iZ2zeg4ytp0whqtmxbsqiiZ ~]# docker push diytomcat
The push refers to repository [docker.io/library/diytomcat]
2eaca873a720: Preparing 
1b38cc4085a8: Preparing 
088ebb58d264: Preparing 
c06785a2723d: Preparing 
291f6e44771a: Preparing 
denied: requested access to the resource is denied	# 拒绝

# push镜像的问题？
The push refers to repository [docker.io/1314520007/diytomcat]
An image does not exist locally with the tag: 1314520007/diytomcat

# 解决，增加一个tag
docker tag diytomcat 1314520007/tomcat:1.0
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813184137474.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)



## 发布到阿里云镜像服务上

1. 登录阿里云
2. 找到容器镜像服务
3. 创建命名空间

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813190111625.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

4. 创建容器镜像

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813190303741.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

5. 点击仓库名称，参考官方文档即可

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813191526549.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

## 总结

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200813194116511.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020081319424581.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZhbmppYW5oYWk=,size_16,color_FFFFFF,t_70#pic_center)