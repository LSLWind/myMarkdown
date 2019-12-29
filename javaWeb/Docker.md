Docker，一种虚拟容器技术，简单说，假如有多个PC与多个应用，将应用部署到PC上，因为PC可能经常更换，所以每部署一次应用到PC上，都需要重新配置环境，非常麻烦，Docker将硬件资源抽象出来，通过在Docker内部配置环境，将应用放到Docker上来运行，一次性全部打包好，将Docker直接放到PC上即可。即屏蔽硬件差异，降低重复劳动，引入Docker这一新的中间层次进行统一环境部署，“开箱即用”。

#### 在Linux上安装Docker

centOs等版本：

```
yum install docker -y
```

ubuntu等版本：

```
sudo apt-get install docker
//进一步使用docker命令
sudo apt-get install docker.io
```

#### 基础概念

仓库： 别人做好的现成的镜像，都放在仓库里，一般都使用docker官网中的仓库中的镜像
镜像： 自己要用哪个镜像，就先拉到本地来。镜像就相当于还没激活的容器，一个镜像可以创建多个容器。
容器： 容器就是跑起来的镜像，就是一个完整的工作环境 	

 **Docker本身并不是容器**，它是创建容器的工具，是应用容器引擎。 Docker有一句口号，即**Build once，Run anywhere（搭建一次，到处能用）** 			 

#### 拉取镜像并在Docker中运行

 所谓的镜像，就是持久化后的，安装了各种工具，软件和服务的一个操作系统（如Linux）。 

使用docker可以直接从远程拉一个镜像过来，语法为：docker pull xxx

```
docker pull how2j/tmall
```

拉完之后，需要激活镜像，所谓的激活，就是把他 run 起来 

```
docker run -dit --privileged -p21:21 -p80:80 -p8080:8080 -p30000-30010:30000-30010 --name how2jtmall how2j/tmall:latest /usr/sbin/init
```

1.  docker run 表示运行一个镜像
2. -dit 是 -d -i -t 的缩写。 

* -d ，表示 detach，即在后台运行
*  -i 表示提供交互接口，这样才可以通过 docker 和 跑起来的操作系统交互
*  -t 表示提供一个 tty (伪终端)，与 -i 配合就可以通过 ssh 工具连接到这个容器里面去了

3. --privileged 启动容器的时候，把权限带进去。 这样才可以在容器里进行完整的操作

4. -p21:21 第一个21，表示在CentOS 上开放21端口。 第二个21 表示在容器里开放21端口。 这样当访问CentOS 的21端口的时候，就会间接地访问到容器里了

5. -p80:80 和 21一个道理
6. -p8080:8080 和21 一个道理
7. -p30000-30010 和21也是一个道理，这个是ftp用来传输数据的
8. --name how2jtmall 给容器取了个名字，叫做 how2jtmall，方便后续管理
9. how2j/tmall:latest  how2j/tmall就是镜像的名称， latest是版本号，即最新版本
10. /usr/sbin/init: 表示启动后运行的程序，即通过这个命令做初始化 	

**例**

先拉一个tomcat过来

```
sudo docker pull tomcat:8.0
```

在docker中运行该tomcat，即创建一个容器并在docker中运行

```
sudo docker run -it --rm -p 8888:8080 tomcat:8.0
```

然后就可以直接通过访问PC来访问应用服务

 http://<ip\>:8888  

即拉取镜像，在docker中跑镜像即变为容器，指定PC开放端口与docker对应的虚拟端口，指定运行容器时需要执行的程序，因此根本不需要在PC本地上配环境，直接跑docker中的容器即可（镜像已经把环境配置好了)

#### 进入容器

进入容器即进入已经配置好并且正在运行的虚拟运行环境（镜像中配置），容器id使用docker ps查看

```
docker exec -it 容器id /bin/bash
```

* docker exec  进入容器
*  -i 表示提供交互接口，这样才可以通过 docker 和 跑起来的操作系统交互
*  -t 表示提供一个 tty (伪终端)，与 -i 配合就可以通过 ssh 工具连接到这个容器里面去了

参数后面跟着名字，与进入后运行的程序

#### 镜像管理命令

*  docker search ：查找仓库中的镜像，如 docker search tomcat，或者直接去官网上搜索，然后pull下来即可
*  docker images ： 查看所有的本地镜像 
*  docker tag a b：用于标记镜像，即给镜像a起一个别名为b，如

```
docker tag docker.io/tomcat:8.0 docker.io/mytomcat:8.0
```

*  docker rmi $(docker images -q) ：删除全部镜像

#### commit：将容器转换为镜像

容器为运行时的镜像，是一个完整的工作环境，commit用于将一个容器装换为镜像，容器一定在运行，容器id使用docker ps查看，id时开始执行时才生成的

语法为：docker commit 容器id 镜像名:镜像标签

```
docker commit 容器id myTomcat:now
```

#### 容器管理命令

容器id使用docker ps查看

* docker pause 容器id： 暂停容器
* docker unpause 容器id： 回复暂停的容器
* docker stop 容器id： 停止容器运行
* docker start 容器id： 开始容器运行
* docker rm 容器id： 删除容器，运行中的容器不能删除，需要先stop
* docker ps：查询所有的容器
*  docker inspect 容器id：检查容器里的各项信息
*  docker rm \`docker ps -a -q` -f ：删除所有容器

#### 提交镜像

就如同git与github一样，docker的镜像都放在docker官网https://hub.docker.com/上，先要去官网上注册账号，账号就是镜像前缀。之后就是可以直接后台登录，输入账号密码后进入自己的镜像管理页面

```
docker login
```

提交镜像就可以使用

```
docker push 账户名/镜像名:标签
```

### 常见问题

#### Got permission denied 

```
Manage Docker as a non-root user

The docker daemon binds to a Unix socket instead of a TCP port. By default that Unix socket is owned by the user root and other users can only access it using sudo. The docker daemon always runs as the root user.

If you don’t want to use sudo when you use the docker command, create a Unix group called docker and add users to it. When the docker daemon starts, it makes the ownership of the Unix socket read/writable by the docker group.
```

大概的意思就是：docker进程使用Unix Socket而不是TCP端口。而默认情况下，Unix socket属于root用户，需要root权限才能访问。因此在docker命令前加上sudo获取权限再执行。