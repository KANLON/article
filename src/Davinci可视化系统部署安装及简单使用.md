
Davinci 是一个目前比较热门的国内开源BI系统，功能比较完善，各种可视化效果也挺不错。主要获取数据的方式是通过编写SQL 创建数据视图来展示各种图表的。
>Davinci面向业务人员/数据工程师/数据分析师/数据科学家，致力于提供一站式数据可视化解决方案。既可作为公有云/私有云独立使用，也可作为可视化插件集成到三方系统。用户只需在可视化UI上简单配置即可满足多种数据可视化需求，并支持高级交互/行业分析/模式探索/社交智能等可视化功能。
>github地址：https://github.com/edp963/davinci

![davinci登录页面](https://img-blog.csdnimg.cn/2021052320291565.jpg)
![davinci拖动演示](https://img-blog.csdnimg.cn/20210523202759895.gif#pic_center)


## 安装部署
> 基于 Ubuntu 16.04 LTS  ，davinci0.3.1 构建，chrome 和 chromedriver 版本 89.0.4389.114

### 初始化MYSQL数据库
首先安装好一个自己的MYSQL数据库，这里 建议命名为 `davinci`  ，然后 在这个数据库上执行 `davinci0.3.1.zip` 压缩包（压缩包可以用官网的，也可以在下面`下载安装资源`中找到）中 bin 目录的 `davinci.sql` 文件来创建对应的表。


### 下载安装资源
访问这个地址 [https://github.com/KANLON/picture-bed/tree/main/resource/artical/Davinci%E5%8F%AF%E8%A7%86%E5%8C%96%E7%B3%BB%E7%BB%9F%E9%83%A8%E7%BD%B2%E5%AE%89%E8%A3%85%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8-davinci-install-package](https://github.com/KANLON/picture-bed/tree/main/resource/artical/Davinci%E5%8F%AF%E8%A7%86%E5%8C%96%E7%B3%BB%E7%BB%9F%E9%83%A8%E7%BD%B2%E5%AE%89%E8%A3%85%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8-davinci-install-package)  将该github 仓库目录下的:
1. chromedriver_linux64.zip  （这个是截图的chromedriver包）
2. davinci0.3.1.zip  （这个是davinci的安装包）
3. google-chrome-stable_current_amd64.deb （这个是Ubuntu版本的chrome浏览器）

这是三个文件放到的 自己的服务器上的  `/data/services/davinci-service` 目录下


可以直接执行以下命令，这个是直接从github上下载，如果速度慢，可以自动手动下载。如果失败了，可以试多几次。
```shell
sudo mkdir -vp /data/services/davinci-service && cd /data/services/davinci-service/ && echo "下载application.yml" && sudo curl -LO 'https://raw.githubusercontent.com/KANLON/picture-bed/main/resource/artical/Davinci%E5%8F%AF%E8%A7%86%E5%8C%96%E7%B3%BB%E7%BB%9F%E9%83%A8%E7%BD%B2%E5%AE%89%E8%A3%85%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8-davinci-install-package/application.yml'&& echo "下载chromedriver" && sudo curl -LO 'https://github.com/KANLON/picture-bed/raw/main/resource/artical/Davinci%E5%8F%AF%E8%A7%86%E5%8C%96%E7%B3%BB%E7%BB%9F%E9%83%A8%E7%BD%B2%E5%AE%89%E8%A3%85%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8-davinci-install-package/chromedriver_linux64.zip' && echo "下载davinci安装包" && sudo curl -LO 'https://github.com/KANLON/picture-bed/raw/main/resource/artical/Davinci%E5%8F%AF%E8%A7%86%E5%8C%96%E7%B3%BB%E7%BB%9F%E9%83%A8%E7%BD%B2%E5%AE%89%E8%A3%85%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8-davinci-install-package/davinci0.3.1.zip' && echo "下载chrome安装包" && sudo curl -LO 'https://github.com/KANLON/picture-bed/raw/main/resource/artical/Davinci%E5%8F%AF%E8%A7%86%E5%8C%96%E7%B3%BB%E7%BB%9F%E9%83%A8%E7%BD%B2%E5%AE%89%E8%A3%85%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8-davinci-install-package/google-chrome-stable_current_amd64.deb' && echo "下载csv模板" && sudo curl -LO 'https://raw.githubusercontent.com/KANLON/picture-bed/main/resource/artical/Davinci%E5%8F%AF%E8%A7%86%E5%8C%96%E7%B3%BB%E7%BB%9F%E9%83%A8%E7%BD%B2%E5%AE%89%E8%A3%85%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8-davinci-install-package/%E6%B5%8B%E8%AF%95davinci%E7%B3%BB%E7%BB%9F%E7%9A%84csv%E6%96%87%E4%BB%B6.csv'
```


### 下载并修改对应的application.yml配置文件
访问这个的github仓库 [https://github.com/KANLON/picture-bed/tree/main/resource/artical/Davinci%E5%8F%AF%E8%A7%86%E5%8C%96%E7%B3%BB%E7%BB%9F%E9%83%A8%E7%BD%B2%E5%AE%89%E8%A3%85%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8-davinci-install-package](https://github.com/KANLON/picture-bed/tree/main/resource/artical/Davinci%E5%8F%AF%E8%A7%86%E5%8C%96%E7%B3%BB%E7%BB%9F%E9%83%A8%E7%BD%B2%E5%AE%89%E8%A3%85%E5%8F%8A%E7%AE%80%E5%8D%95%E4%BD%BF%E7%94%A8-davinci-install-package) 下载获取 `application.yml` 文件 ，并修改其中的 `MySQL 数据库`（这里要注意一下jdbc链接后面的某些参数不能删除）和`邮件配置`，然后把该配置文件放到 `/data/services/davinci-service` 目录（主要是为了方便后面的执行脚本方便执行，自己可以另外使用目录，不过要重新修改一下下面的脚本）下

###  安装脚本
完成了上面的数据库安装和安装资源下载，然后切换到`root` 账号，执行一下脚本 ，即可安装启动 `davinci` 系统

```shell
echo "复制安装包"
mkdir -pv /data/services/davinci-service
cd /data/services/davinci-service
cp /data/davinci-install-package/davinci0.3.1.zip  /data/services/davinci-service
unzip davinci0.3.1.zip 
mv davinci-assembly_0.3.1-0.3.1-SNAPSHOT-dist-rc davinci0.3.1
echo "复制配置"
/bin/cp -rf /data/davinci-install-package/application.yml /data/services/davinci-service/davinci0.3.1/config
export DAVINCI3_HOME=/data/services/davinci-service/davinci0.3.1
echo "安装Chrome的驱动方便截图"
mkdir -pv /data/zhangcanlong
sudo apt-get update
cd /data/zhangcanlong
echo "安装 chorme"
cp /data/davinci-install-package/google-chrome-stable_current_amd64.deb ./
sudo dpkg -i google-chrome-stable_current_amd64.deb
echo "强制安装所有缺少的依赖"
sudo apt-get install -f -y
sudo /etc/init.d/dbus start
sudo apt-get install libnss3 -y
echo "测试chrome是否成功启动"
sudo google-chrome --no-sandbox --headless --disable-gpu  --user-data-dir  https://www.baidu.com
cp /data/davinci-install-package/chromedriver_linux64.zip /data/zhangcanlong/
unzip /data/zhangcanlong/chromedriver_linux64.zip 
cp /data/zhangcanlong/chromedriver /usr/bin/
chromedriver --version
echo "启动服务"
/data/services/davinci-service/davinci0.3.1/bin/start-server.sh
```

如果压缩一条命令就是下面这样了，基本先下载了上面的安装包资源，再运行下面这条安装命令就可以完成安装davinci了
```shell
echo "复制安装包" && mkdir -pv /data/services/davinci-service && cd /data/services/davinci-service && cp /data/davinci-install-package/davinci0.3.1.zip  /data/services/davinci-service && unzip davinci0.3.1.zip  && mv davinci-assembly_0.3.1-0.3.1-SNAPSHOT-dist-rc davinci0.3.1 && echo "复制配置" && /bin/cp -rf /data/davinci-install-package/application.yml /data/services/davinci-service/davinci0.3.1/config && export DAVINCI3_HOME=/data/services/davinci-service/davinci0.3.1 && echo "安装Chrome的驱动方便截图" && mkdir -pv /data/zhangcanlong && sudo apt-get update && cd /data/zhangcanlong && echo "安装 chorme" && cp /data/davinci-install-package/google-chrome-stable_current_amd64.deb ./ && sudo dpkg -i google-chrome-stable_current_amd64.deb && echo "强制安装所有缺少的依赖" && sudo apt-get install -f -y  && sudo /etc/init.d/dbus start && sudo apt-get install libnss3 -y && echo "测试chrome是否成功启动" && sudo google-chrome --no-sandbox --headless --disable-gpu   --user-data-dir  https://www.baidu.com && cp /data/davinci-install-package/chromedriver_linux64.zip /data/zhangcanlong/ && unzip /data/zhangcanlong/chromedriver_linux64.zip  && cp /data/zhangcanlong/chromedriver /usr/bin/ && chromedriver --version && echo "启动服务" && /data/services/davinci-service/davinci0.3.1/bin/start-server.sh
```



命令执行完之后，等几分钟后，看到以下截图则表示启动成功了，然后访问 `https://127.0.0.1:8080` 即可访问 `davinci` 系统
![danvinci启动完成](https://img-blog.csdnimg.cn/20210517094042611.png)



## 简单使用

在安装完成之后，打开登录页面 `localhost:8080` ，输入以下默认的账号和密码即可登录成功
用户名 guest，密码 123456

然后接着就是  创建数据源(source)  -> 创建数据视图(view) -> 创建可视化组件(widget)  

> 注意davinci 创建的数据的时候都是基于SQL编写的，创建之前需要了解常用的一些SQL语法

基于 可视化组件  可以组成 仪表盘（Dashboard） 或大屏 (Display)

另外还可以将创建的可视化组件通过截图或者Excel 方式定时发送到邮件中（Schedule）

基本照着 官方文档的操作就没有什么问题了，官方文档说明的还是比较简洁明了的

davinic创建的官方文档地址： [https://edp963.github.io/davinci/docs/zh/1.2-quick_start](https://edp963.github.io/davinci/docs/zh/1.2-quick_start)

我这里简单展示一下自制的教程动图吧，如果有不懂的，可以看一下官方文档里面的教程

教程视频链接：[https://www.bilibili.com/video/BV1AU4y1b7Ms](https://www.bilibili.com/video/BV1AU4y1b7Ms)

## 总结

Davinci目前是我们国内一个比较热门的开源报表系统了，刚开始安装它的时候是比较简单的，因为是Java写的，我比较熟悉，但是遇到了一个小问题，就是要安装截图驱动，这个弄了挺久了。当然如果用docker部署的话，会快很多的，但是由于我公司的服务限制，如果要下载其他人的镜像去部署的话很麻烦，只能使用我们已经有了的镜像基础上去部署的🙃，所以就只能用Linux系统脚本从头开始部署了。

谈完部署，说一下它的使用体会，Davinci感觉交互方面做得还是挺不错的，各种可视化的交互做得挺好。另外组成页面的有两种方式的：一种是普通报表形式，只能放数据组件；另外一种是特制大屏页面形式，除了可以放一些数据组件之外还可以的单独拖放一些文本，日期，视频等组件，更具有个性化。不过数据那里生成是只能用SQL生成，并没有提供一些简单界面根据表来自动生成SQL去查询，对于不懂SQL的语法的用户有一定的使用门槛，而且数据告警的功能的也没有，就是针对数据浮动大于多少的时候，才发送邮件或短信告警，目前Davinci 只支持定时任务告警，并没有结合这个数据波动情况告警的。


总的来说，Davinci是目前挺不错的开源报表系统，适用于大部分公司，而且是Java作为后端开发的，改造起来也很方便。

## 参考

1. Davinci官方安装文档 https://edp963.github.io/davinci/docs/zh/1.1-deployment


> 文章杂谈
> 好像真的挺久没更新了，我每周一篇目标落下了很多😪，这篇文章也写了挺久，主要是要弄一个视频教程，感觉自己应该好好学习一下视频剪辑了。后面会陆续更新一下其他国内的开源报表系统的部署和简单演示，敬请关注！

我是CurdBoys  ，一个**经常在做CURD而不仅仅满足于CURD**的程序猿。


![CrudBoys](https://img-blog.csdnimg.cn/20201114143851114.png)
