# 最火的分布式调度系统 XXL-JOB 安装和简单使用

> 唉，在谈文章之前先说一下自己的情况。原计划是在上周六写完这篇文章的，然而周六的时候打开电脑的，按照平常"惯例",先补一些 “黑色五叶草”/“进巨”的番，然后刷着刷着就忘记写😔，接着就逛逛知乎，逛逛b站。啪地一下，趁我不注意🙃，周六日就过去，于是就拖到了这周才来更新（立下的flag总是被自己打倒🤐）。这周六日我有也有写部分，不过发现做一个这样的教程类文章也是挺耗时间（这篇文章差不多耗了我两个周末的时间），要先自己搭建完好之后，然后把其中每一步的操作都记录下来，相当于为了写这篇文章，我搭了2,3次 xxl-job 🤣。 不过写完之后，感觉对于自己收获也挺大的，顺便复习了一下 Shell 的相关命令和 Java 项目的命令行启动。以后还是要加油啊！尽量坚持周更！向着大神们学习。

![加油](http://ww2.sinaimg.cn/bmiddle/006r3PQBjw1fbrk6urvr5g302i02iq3n.gif)

上一个篇文章 [分布式调度任务系统调研及选型](https://mp.weixin.qq.com/s/-nLMCkqEoCfrsVq1ByHmog)，我们介绍了常用的一些开源的分布式调度系统，知道了`xxl-job` 目前基本是中小型公司的第一选择。所以这篇文章，就来介绍一下`xxl-job` 的安装和简单使用。
> 这里安装是基于 Ubuntu 16.04 安装的。


另外我可没有标题党，我看了一下 GitHub 上开源的分布式调度系统，目前是 xxl-job 项目是分布式调度开源系统中最多 starts 的（当然也有可能因为这个 xxl-job 出来挺久了）

![xxl-job的GitHub](https://s3.ax1x.com/2020/12/21/rda4c4.png)

## window ，mac 或docker 安装

如果为 window，mac 等安装，可以先根据github 上的 SQL 脚本创建好数据库和表，然后直接将github 上的源码 导入到 idea 按照Spring Boot 应用来启动即可。

如果为 docker 的安装，可以直接pull 官方镜像，然后运行时指定 MySQL 和存储日志目录即可。

下载镜像
```shell
# Docker地址：https://hub.docker.com/r/xuxueli/xxl-job-admin/     (建议指定版本号)
docker pull xuxueli/xxl-job-admin
```

创建容器并运行
```shell
# 如需自定义 mysql 等配置，可通过 "-e PARAMS" 指定，参数格式 PARAMS="--key=value  --key2=value2" ；
# 配置项参考文件：/xxl-job/xxl-job-admin/src/main/resources/application.properties
# 如需自定义 JVM内存参数 等配置，可通过 "-e JAVA_OPTS" 指定，参数格式 JAVA_OPTS="-Xmx512m" ；

docker run -e PARAMS="--spring.datasource.url=jdbc:mysql://127.0.0.1:3306/xxl_job?useUnicode=true&characterEncoding=UTF-8&autoReconnect=true&serverTimezone=Asia/Shanghai" -p 8080:8080 -v /tmp:/data/applogs --name xxl-job-admin  -d xuxueli/xxl-job-admin:{指定版本}
```


##  linux Ubuntu 16.04 安装 xxl-job

在linux下直接安装 xxl-job 会有点麻烦， 因为 xxl-job 并没有像 [nacos](https://github.com/alibaba/nacos) 之类的直接提供了一个启动包，直接修改一下配置文件，然后运行启动脚本即可那种。xxl-job 是只提供了的源码文件，如果要运行，需要自己先将 java 源码 通过 maven 打包构建成运行包，然后才能运行。所以接下来我来详细讲述一下如何在 linux 下安装启动 xxl-job，这里使用的 linux 版本为：Ubuntu 16.04。 其他版本也类似。

### 安装 jdk ，maven ，git 环境

首先我们先需要安装 Java 相关的环境，因为xxl-job 是 Java 开发的，在没有提供启动包的基础上，我们需要自己从源码解释编译成启动包，然后启动。


#### 安装 JDK

安装有两种一种是直接根据 apt 的包安装，另一种是手动获取 java 包安装。这里我们为了简单起见使用 apt 直接安装，如果需要更多自定义的配置可以手动安装。执行运行以下两个命令即可安装 jdk8 版本的。

```shell
sudo apt-get update
sudo apt-get install openjdk-8-jdk -y
```

参考：[Ubuntu 16.04安装Java JDK8](https://blog.csdn.net/u012707739/article/details/78489833)

#### 安装 maven

我们这里也直接采用 apt 包的安装
```shell
sudo apt-get -y update 
sudo apt install maven -y
```
待安装完成之后，运行
```shell
mvn -v
```
即可检查是否安装成功了


这里安装了 maven 之后还需要将 maven 的默认的远程仓库地址换成 国内的远程仓库地址，不然下载 maven 依赖的时候会很慢。打开 `/etc/maven/settings.xml` 文件，在 <mirrors>标签里面添加以下内容,即可将默认仓库替换成阿里云的仓库。

```xml
 <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
 </mirror>
```



#### 安装 GIT 
我们这里还需要安装 GIT ，主要是为了可以方便复制源码到本机，当然也可以通过直接 curl 获取到文件，然后解压。不过 git 总是有用，所以我们这里也安装了

```shell
sudo apt-get -y update
sudo apt-get install git -y  
```

### 安装数据库

初始化数据库的脚步为xxl-job的git 项目中的 `/xxl-job/doc/db/tables_xxl_job.sql`的文件，在已经安装了 MySQL 的机器上连接上，然后执行即可  

默认xxl-job 是支持集群部署的，只要连接相同的数据库即可

另外如果 MySQL 有做主从部署，则 xxl-job 连接的一定要为主库，也不能使用读写分离的账号，否则在调度的时候可能会出现问题

### git clone 源码文件，打包构建

```shell

git clone https://gitee.com/xuxueli0323/xxl-job.git
cd xxl-job/

mvn -B -Dmaven.test.skip=true clean package
cd xxl-job-admin/target
# 这里 根据不同的xxl-job的版本，文件名不一样
unzip xxl-job-admin-2.3.0-SNAPSHOT.jar

# 解压之后，进入BOOT-INT/target/class/ 目录即可看到调度中心的配置文件 `application.properties`，修改里面的数据库链接和密码即可，如果考虑调度系统的调用的安全性可以加上, xxl.job.accessToken= 参数

# 修改完成之后，然后进入到 /xxl-job/xxl-job-admin/target 目录下执行以下命令来启动 xxl-job 
/usr/local/java/bin/java  -classpath ./:./lib/*  org.springframework.boot.loader.JarLauncher &
```

执行之后，如果启动成功会显示如下图这样

![调度中心启动成功例子](https://s3.ax1x.com/2020/12/21/rdd3CT.png)


然后打开链接： http://127.0.0.1:8080/xxl-job-admin  (该地址执行器将会使用到，作为回调地址)
 
输入默认的账号和密码， “admin/123456”，登录即可看到调度中心。（需要将127.0.0.1 换成自己的机器的ip） 

![调度中心管理页面](https://s3.ax1x.com/2020/12/21/rddJv4.png)

### 安装执行器项目

安装完成调度中心之后，我们还需要安装执行器项目，调度中心本身是不负责执行项目，要安装执行器之后，才能通过调度中心去执行器服务上执行。安装执行器项目很简单，基本上就是安装调度中心那样安装启动即可，我们这里为了简单起见，可以直接启动 xxl-job 自带Spring Boot的简单执行器。基于上面克隆下来的项目，我们进入 `/xxl-job/xxl-job-executor-samples/xxl-job-executor-sample-springboot/target` 目录。如果要修改调度中心的地址的信息、调度中心的验证的key等，可以修改配置文件，该执行器的配置文件为 `/xxl-job/xxl-job-executor-samples/xxl-job-executor-sample-springboot/target/BOOT-INF/classes/application.properties`。

然后执行以下命令即可启动执行器项目：
```
/usr/local/java/bin/java  -classpath ./:./lib/*  org.springframework.boot.loader.JarLauncher &
```


## 简单使用 xxl-job

经过上面的步骤，我们就有一个简单的调度中心系统了。基于这个调度系统，我们可以随便创建调度任务。这里我示范一个创建一个每分钟定时请求 http://t.weather.itboy.net/api/weather/city/101280101  链接获取广州天气的任务。参考：[免费天气API，天气JSON API，不限次数获取十五天的天气预报](https://www.sojson.com/blog/305.html)

### 登录进入任务管理页面新建任务

访问，http://127.0.0.1:8080/xxl-job-admin ，输入 admin/123456 。登录上面我们搭建成功的xxl-job的分布式调度系统。然后进入"任务管理" 页面，进行新建任务，

![xxl-job 任务管理页面新建操作](https://s3.ax1x.com/2020/12/21/r0IuZR.png)

### 填写任务的基本信息

这里填写的是调度任务的基本信息，例如：任务失败的时候重试的次数，该任务的负责人、应该依赖的上一个任务id。

在弹出的新建任务的窗口，按照如下填写：

![xxl-job新建任务例子](https://s3.ax1x.com/2020/12/21/r0I4YV.png)

### 编辑任务的具体执行的内容

我们可以通过xxl-job 自带的 IDE  来编辑我们需要运行的shell脚本。

按照上面填写好任务的基本信息后，我们接着来编辑要执行的任务脚本，
![操作中打开GLUE IDE 去编写运行脚本](https://s3.ax1x.com/2020/12/21/r0okTI.png)

![编辑shell脚本获取广州天气](https://s3.ax1x.com/2020/12/21/r0bjzV.png)

### 启动任务并查询执行记录

点击保存，然后返回任务列表，启动该任务。然后就可以到"调度日志"页面，查看该任务的执行记录了，点击具体运行的某个实例的“查看日志”能看到该实例执行记录的详细日志信息。

![查看任务的执行的实例](https://s3.ax1x.com/2020/12/21/r0qylV.png)


![某个实例具体执行的日志.PNG](https://s3.ax1x.com/2020/12/21/r0L96f.png)


## 总结

本篇文章介绍了分布式调度任务系统 xxl-job 的安装 和简单使用。其实整篇下来，我觉得挺水的😅；算作当做自己记录一下吧，这个分布式调度系统的，其实功能挺简单的，如果运行比较少的任务的会比较轻松和容易上手。不过如果要运行一些复杂的任务，特别是包含一些例如，按小时汇总到天 然后触发依赖 之类逻辑较多的依赖或者是想延迟执行、指定重跑某个数据时间的实例 等等复杂逻辑，就比较困难了。另外这个分布式开源的调度系统的任务实例界面和任务管理界面的交互并不是很好，都是展示成列表的形式。如果把任务实例界面修改为按照单元格日期 这样来选择会比较好些。因为我们公司也有类似这样的分布式调度系统，不过我们公司的由于业务比较复杂，所以我们的分布式调度任务系统的功能比较多，相对于 xxl-job 有一定优势，但是也有挺多缺点的。

总而言之，对于绝大部分中小公司的调度任务或者是比较简单的调度任务用xxl-job 是完全没有问题的。如果是要作为企业级系统来使用估计还需要基于 xxl-job 来修改挺多模块的。最后，还是挺感谢 xxl-job 能开源，对于我平常用来管理一下自己的调度任务也很不错😊。

![微信公众号图片](https://s3.ax1x.com/2020/11/25/DdddOS.png)




