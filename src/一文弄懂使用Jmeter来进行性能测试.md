该文章是基于上一次文章的 [软件测试漫谈（web测试，自动化测试，Jmeter）](https://blog.csdn.net/weixin_37610397/article/details/103341856) 的续篇， 主要是详细讲解 Jmeter 的入门教程。

因为上次的文章只是简单地讲解了 Jmeter 的使用和一些概念，所以很多初学者像按照原来的文章操作 Jmeter 进行测试是比较麻烦的，所有才有了这篇后续，以帮助开发者能快速使用 Jmeter 来进行测试。如果还没看过之前那个文章，建议先看一下，学习一下Jmeter的基本概念，以便熟悉该文章操作的一些概念。该文章基于 Jmeter 版本 5.1.1 和 Windows 10。


### 测试需求
模拟20个 谷歌浏览器类型 用户  同时 访问 http://www.baidu.com/  和 https://www.csdn.net/api/articles （该链接是获取 csdn 上的文章，如果正确会返回状态 "status": "true",） 在负载达到30 QPS  时的平均响应时间 和 结果准确度。

### 操作步骤
0. 在 Jmeter 官网上下载 Jmeter 安装包，下载地址： [https://apache.osuosl.org//jmeter/binaries/apache-jmeter-5.2.1.zip](https://apache.osuosl.org//jmeter/binaries/apache-jmeter-5.2.1.zip)，需要先安装了 Jdk ，解压 打开文件 `apache-jmeter-5.1.1\bin\jmeter.bat`

#### 简单设置请求

1. 在测试计划中选择新建线程组，重命名线程组为20个模拟用户线程组。
![新建线程组](https://img-blog.csdnimg.cn/2020040909121816.png)
在 Number of Threads 栏 填写 20 ，设置 线程数为 20。在 Loop Count 栏填写 10000，设置 循环请求10000 次。

![设置线程数](https://img-blog.csdnimg.cn/20200409092220284.png)

2. 在线程组上右键，选择添加 Http 请求的取样器
![添加取样器](https://img-blog.csdnimg.cn/20200409091729188.png)
重命名该取样器为请求百度地址，在 `Server Name or IP` 栏填写 `www.baidu.com` ，`Prot Number` 栏填写 `80` 端口，如果是https 协议，则填写 `443` 端口。

![设置请求百度地址请求](https://img-blog.csdnimg.cn/20200409092520322.png)
3. 再在线程组上填写 HTTP 请求取样器，并按照如下图来进行设置和重命名为 `请求 csdn 文章` 。

![设置请求csdn文章](https://img-blog.csdnimg.cn/20200409092842474.png)
4. 设置结果查看 ，在线程组右键，添加结果树

![新建结果树](https://img-blog.csdnimg.cn/20200409093350702.png)
重命名为 `结果查看`，然后点击 上方的 绿色的运行按钮，即可，查看到发送的请求 HTTP 请求。点击结果树中的，任意一个请求可以看到请求的请求头，请求时间，响应头，响应时间等具体信息。

![设置结果查看树](https://img-blog.csdnimg.cn/20200409093531581.png)
然后，我们可以通过添加 总报表 的监听器 来查看 请求的总体发送情况

![总的报告](https://img-blog.csdnimg.cn/2020040923125381.png)
![总的请求发送情况](https://img-blog.csdnimg.cn/20200409231504267.png)
#### 一些特定参数设置

5. 设置吞吐量/QPS
Jmeter 提供了一个很有用的定时器，叫做 Constant Throughput Timer （常数吞吐量定时器），该定时器可以很方便地控制给定的取样器发送请求的吞吐量。

在线程组上右键新建吞吐量定时器
![添加吞吐量定时器](https://img-blog.csdnimg.cn/20200409230638827.png)

 在 Target throughput( in smples per minute ）（每分钟的目标吞吐量）:  实际填写的数值为：60 * QPS ，所以这里我们填写1200，然后 Calculate Through based on 选择 All active threads 。
 
![设置吞吐量](https://img-blog.csdnimg.cn/20200409231147480.png)
`注意`：这里计算的吞吐量有时候并不准确，一般进行测试的时候，我们会选择提高一些吞吐量的值以达到实际要求的 QPS


6. 设置请求头的参数，设置浏览器类型

对于某些请求，有可能我们需要设置请求的请求头的信息，例如：浏览器类型，鉴权信息 token 等，这时候，我们可以通过添加 http 请求头的配置元件来完成。
![配置元件](https://img-blog.csdnimg.cn/20200409231817842.png)
添加配置元件之后，我们添加上浏览器的请求头信息.
![添加配置信息](https://img-blog.csdnimg.cn/20200409232144752.png)
这时候再点击运行，可以看得到发送的请求中，请求头包含了我们设置的信息：

![发送的请求](https://img-blog.csdnimg.cn/20200409232421431.png)


7. 判断返回结果的是否成功

默认 Jmeter 是根据响应码来判断结果是否请求请求的，非200 的判断为不成功，如果我们想根据响应内容判断，则需要添加断言器，来自己判断内容包含哪些字符等，一般我们添加响应信息的断言器，即可。

![添加断言器](https://img-blog.csdnimg.cn/20200409232457547.png)
我们加上如果请求响应信息中包含了字符 `{"status":"true` 我们则判断该请求成功，否则失败，我们可以这样来设置断言。


![设置结果断言器](https://img-blog.csdnimg.cn/20200409232943514.png)
如果我们想单单针对某个请求来进行断言，我们可以，将断言放到某个请求里面，这样它就会只判断该请求。这里我们只针对 获取 csdn的文章的请求来断言。



#### 最终的结果
我们完成以上配置之后，点击运行，等待结果运行稳定好，查看 总报告情况 可以得到以下 截图

![总结果](https://img-blog.csdnimg.cn/20200409234916331.png)
可以看到，再QPS 约为 5 的情况下，两个请求的总体的情况平均响应时间为 2.371 秒，当前总共发送了 733个请求，错误率 为 3.14%（暂定的时候，后面几个请求会断开，也算入了错误率的）。由于这两个请求的响应时间都比较慢和定义的用户数只为20个，所以QPS 比较难达到20 . 读者后面可以自己提高 线程数 或者替换请求地址来测试看看。

### 注意事项
1. 一般而已如果需要进行很高的QPS 的性能测试，不要使用界面来测试，配置完成后，保存jmx 文件之后，再使用命令行来测试会更好，因为界面运行的话，会占用很大资源，实际发送的请求可能得不到自己想要的QPS。界面运行命令，启动 jmeter 的时候就有提示。`jmeter -n -t [jmx file] -l [results file] -e -o [Path to web report folder]`
![运行命令](https://img-blog.csdnimg.cn/20200409233625117.png)

## 参考资源

1.  [Jmeter 中文教程 板书](//download.csdn.net/download/weixin_37610397/12313606)


