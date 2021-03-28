---
title: "创建ManicTime的服务器"
date: 2021-03-27T09:05:30+08:00
draft: false
description: 在Nas里建立MacnicTime的服务器的正确姿势
tags: [折腾,nas]

---

## 开搞前

关于ManicTime（后文称MT）是什么，在之前的一篇博客里有详细介绍过，自己用了一段时间下来，拖延症的症状得到了明显的改善，每天的时间安排也变得更加有序了，就想着看看手机端是什么亚子的，结果打开手机直接显示要登陆MT，这哪行，一定要看到APP端的界面才行。

![image-20210328091813533](https://30924398.xyz:6001/images/2021/03/28/image-20210328091813533.png)

本想着直接抄袭下现成的作业就好了，无奈这款软件实在小众，中文搜索到的都是关于怎么用客户端的水文，没有现成的配置服务器的作业可以抄，谷歌搜到的相关内容都不多。那只能发挥靠天靠地靠祖上，不如靠自己的折腾精神了。话不多说，开搞。

## 服务器的选择

官方提供了两种搭建服务器的选择，**付费选择官方提供的云服务器**以及**自行搭建服务器**。前者貌似是订阅版才可以使用，我目前还是在产品的试用期，那自然就只能选择自建服务器（这个选项也是需要Pro版的）了。

![image-20210328092521980](https://30924398.xyz:6001/images/2021/03/28/image-20210328092521980.png)

在自行搭建服务器的基础上，有三个版本可以选择：

+ Windows
+ Linux
+ Docker

刚好手上有群晖的Nas，所以选择了Docker，后面就因为对Docker的不了解，绕了一些弯路

## 误以为的安装过程

安装过程回过头来看其实不难，只是其中有一些知识盲区，所以饶了一些弯路。**原本以为**整个安装过程无非就是下面几步：

1. 在群里Dcoker中下载MT容器；
2. 在NAS中创建供MT使用的文件夹；
3. 在容器启动的配置项里填好上一步创建好的文件夹以及装载路径；
4. 通过SSH连接到NAS中创建好服务器账号密码；
5. 通过SSH连接到NAS中启动MT服务器。

需要注意的是，在SSH中进行DOCKER的相关操作需要ROOT权限，SSH连接成功后，输入`sudo -i`后重新输入一次nas密码即可切换到ROOT账号下了。

![image-20210328140527775](https://raw.githubusercontent.com/tsq456/picBed/master/image-20210328140527775.png)

这几步中自己主要是在第3和第5步卡住了，这里简单记录下。

### 容器的安装目录和装在路径

因为在[MT容器的说明](https://hub.docker.com/r/manictime/manictimeserver/)里关于容器的安装就简单的两个步骤：创建账号密码以及启动服务器（即上面提到的4、5两步），而关于创建账号密码的说明只有下面简单的一行Docker命令。

``` dockerfile
docker run -v /var/lib/manictimeserver/Data:/app/Data --rm --entrypoint dotnet  manictime/manictimeserver ManicTimeServer.dll addadmin -u <admin email> -p <admin password>
```

因为原本对Docker的了解也就局限于群晖上的一些类似迅雷下载这样的实际应用，再加上后来玩ESXi时候知道了原来Docker也是一种虚拟技术，并且基本也都是抄别人作业，在此之外就再无任何了解了。因此也就有些搞不懂在Nas下创建容器时是否需要预先设置一些参数。

后来发现上面命令中`/var/lib/manictimeserver/Data:/app/Data`这一段路径代码里提到了两个路径，就尝试着在容器高级设置里面的卷分页签下装在路径填上`/app/Data`，之后就可以了。

![image-20210328142335054](https://raw.githubusercontent.com/tsq456/picBed/master/image-20210328142335054.png)

但是这里还有个没弄清楚的地方就是在于前面这个路径`/var/lib/manictimeserver/Data`代表什么，因为在卷里面选择的文件夹可以是自己创建的文件夹，但是在创建账号密码时使用的命令中又可以包含这串路径。而且在成功创建服务器后，这两个路径下都会有相同的文件。

![111](https://raw.githubusercontent.com/tsq456/picBed/master/image-20210328143817392.png)

![image-20210328143842959](https://raw.githubusercontent.com/tsq456/picBed/master/image-20210328143842959.png)

> 写这篇的时候顺道搜索了下Docker的一些说明文档，原来`/var/lib/manictimeserver/Data:/app/Data`这行命令的意思是将宿主机（Nas）的目录`/var/lib/manictimeserver/Data`挂载到容器的`/app/Data`下。
>
> 因此如果是在群晖中已经创建好MT数据的存储路径DirA，并且在群晖Docker下高级配置填写了/app/Data的情况下，最开始那行命令应该是下面这段命令

``` dockerfile
docker run -v DirA:/app/Data --rm --entrypoint dotnet  manictime/manictimeserver ManicTimeServer.dll addadmin -u <admin email> -p <admin password>
```

所以如果不想在宿主机里出现两份MT数据的话，其实就是需要制定上面提到的DirA为自己创建的数据库路径。因此，其实就没有所谓的第3步。

### 启动容器

在[MT容器的说明](https://hub.docker.com/r/manictime/manictimeserver/)里关于启动服务器通用也是一行简单的命令。

``` dockerfile
docker run -v /var/lib/manictimeserver/Data:/app/Data -p 8080:8080 manictime/manictimeserver
```

那其实经过上面带冒号的路径的理解后，这里的带冒号的前后端口号就也相对好理解了，其实就是将容器的8080端口映射到宿主机的8080端口，以下搜到的这段话可以更好地帮助理解。

> 8080：80表示在容器中您正在使用端口80，并将该端口转发到主机的8080端口。因此，在方案2中的任何位置，您都在容器内的端口80上运行Jenkins，并且在容器内的端口8080上运行Jenkins，并将其暴露在主机的同一端口上。
> 例如，如果我在容器中运行mysql，则可能会使用8080：3306，因此mysql将在端口3306上运行，但在主机的8080上暴露，但是如果为mysql选择8080：80，则它可能无法工作，因为按照mysql的代码，它绑定在端口3306而不是端口80上。在您的詹金斯案例中，情况也是如此。

因此，如果你的命令中的端口号是`1234:6789`就是指定容器运行在Docker的6789端口，但是转发到宿主机的1234端口（这样后续如果要在nas中做外网的端口暴露，需要映射的就是1234端口）。

## 真实的安装过程

因此，重新归纳一遍安装MTServer的流程就是。

1. 在群里Dcoker中下载MTServer的容器镜像；
2. 在NAS中创建供MT使用的文件夹**（目录DirA，群辉文件夹的真实路径一般是/volumN/共享文件夹文件夹名/下级文件夹名，可通过右键-属性查看）**；
3. 创建容器时无需配置任何信息，直接启动；
4. 通过SSH连接到NAS，将容器挂载到第2部创建好的文件夹路径下并创建好服务器的账号密码，命令是`docker run -v DirA:/app/Data --rm --entrypoint dotnet  manictime/manictimeserver ManicTimeServer.dll addadmin -u <admin email> -p <admin password>`，其中-u后面就是设定的账号，-p后面就是设定的密码，自行替换。

![image-20210328153647113](https://30924398.xyz:6001/images/2021/03/28/image-20210328153647113.png)

5. 启动MT服务器，命令是`docker run -v DirA:/app/Data -p 1234:5678 manictime/manictimeserver`其中1234是MTserver在宿主机中的运行端口号，5678是其在Docker中的运行端口号，不要与其他程序或者容器发生冲突。

![image-20210328153725815](https://30924398.xyz:6001/images/2021/03/28/image-20210328153725815.png)

启动成功后，在容器里会自动创建一个随机命名的新容器。

![image-20210328154102067](https://30924398.xyz:6001/images/2021/03/28/image-20210328154102067.png)

再之后 可以在浏览器里输入你的nas内网地址+端口号访问MT Server的服务端了，输入之后创建时候设置的账号密码就可以进到服务端的管理台了。

## 总结

碰到问题多读文档，其实问题的根源就在于不懂docker挂载的命令格式，碰到路径之间加冒号就懵逼了。