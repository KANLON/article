
> 软件测试，就是一个过程或一系列过程，用来确定计算机代码完成了其应该完成的功能不执行其不该有的操作。

简单说就是找bug的过程。


### 测试分类


（1）按测试方式分类：静态测试、动态测试

（2） 按测试方法分类：白盒测试、黑盒测试

（3） 按测试自动化程度分类：人工测试、`自动化测试`

（4） 按测试阶段分类：单元测试、集成测试、确认测试、系统测试、验收测试

（5） 按测试类型分类：`功能测试`、`界面测试`、`性能测试（负载测试、强度测试）`、容量测试、压力测试、安全测试、兼容性测试、安装测试、文档测试、易用性测试、边界测试、健壮性测试、本地化测试、无障碍测试、回归测试、冒烟测试、Monkey测试、A/B测试

（6） 按测试模式分类：瀑布模型、敏捷测试、基于脚本的测试、基于风险的测试、探索式测试

（7） 按测试的对象分：`web测试`、移动测试。。。

。。。



功能测试（黑盒测试），功能测试就是对产品的各功能进行验证，根据功能测试用例，逐项测试，检查产品是否达到用户要求的功能


自动化测试，一般是指软件测试的自动化，软件测试就是在预设条件下运行系统或应用程序，评估运行结果，预先条件应包括正常条件和异常条件。


### web界面测试、性能测试、接口测试  框架简单介绍

#### web界面测试：

- [Selenium](https://selenium.dev/)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191202092927901.jpg)

IDE是：selenium-server、selenium-java包（Java版本）

简单来说就是做web自动化测试框架,可测试不同的浏览器.


- [katalon](https://www.katalon.com/) 

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191202093047652.PNG)

新兴起的自动化测试软件，更易使用，IDE是katalon studio。



- 两者的区别

1. Katalon的用户群体范围更广，用户甚至不需要任何编程基础，只需要知道自己想做什么即可，而Selemium的用户群体是开发或者测试人员
2. Katalon安装和使用都更简单，只要下载软件，然后点击exe，登录即可。Selenium需要先安装jRE，selenium-server、selenium-java包，如果是在谷歌浏览器使用还要下载Chromedriver包，而且界面没有katalon更易操作和美观，主要是通过命令行和代码实现录制和测试。另外是Katalon recode（脚本录制）比Selenium IDE（脚本录制）更简单，识别准确度也更加高


>（Katalon Recorder（katalon录制脚本的一个工具，直接在google的应用商城搜索安装插件即可）。它可以录制自己在浏览器的记录，然后可以回放操作、控制速度和导出java等代码的执行脚本，对于界面测试很有帮助）(演示)



#### 接口测试：

Jmeter，loadrunner，Postman，restclient,


Jmeter和loadRunner在这里的接口测试，是指其能录制脚本，并记录下所有的请求，并且能通过正则匹配等方式来验证返回的结果是否正确，对于批量测试接口和已经现成的网站来获取接口来测试很方便。

Postman和Restclient则是专门的接口测试工具了，对于构造请求参数和发送请求都很方便，对于开发阶段测试较方便。

*Jmeter的脚本录制需要注意的点的，对于https协议的话，你需要将其bin目录下生成证书放到其根目录下才能进行访问网站，否则可能访问不了*


#### 性能测试

Postman,   Jmeter,   ab

##### Postman自带的测试（Runner测试，不能并发测试）：

在Runner界面下，可以设置之前自己构造的请求发送多少次请求， 但是发送的请求是串行的，即等上一个请求发完才能继续发送。这个功能适合简单测试，因为平时测试一般都是使用postman来构造请求测试，所以这种性能测试几乎是不用额外的工作量，就可以测试出程序在多个请求发生时的情况（程序内存，cpu使用率等），从而针对程序进行优化。缺点是：不能模拟并发场景，适用于没有较少并发场景的接口，例如一些比较少用的保存，接口，可以利用该功能模拟程序运行久之后，运行情况。（演示）


##### Jmeter测试

Jmeter是一个专业开源的性能测试软件，纯Java开发，可以用于对静态的和动态的资源（文件，Servlet，Perl脚本，Java对象，数据库查询，FTP服务器或是其他资源）的性能进行测试。

它提供了图形化界面来供用户适用，也可以通过引入插件方式来给测试的指标更丰富的可试化功能。


它提供的功能包含脚本录制，参数化，集合点，检查点，关联，多协议，多线程，报告生成与导出等，几乎能满足一切的性能测试的要求，所有目前这个性能测试框架是比较多公司在使用的，像阿里的产品 性能测试 PTS（Performance Testing Service）中就支持Jmeter导入测试。https://help.aliyun.com/document_detail/29262.html


这个在后面在具体讲解专门使用


##### ab（apachebench）

ab是一种用于测试Apache超文本传输协议（HTTP）服务器的工具。（演示）

>   ab的原理：ab命令会创建多个并发访问线程，模拟多个访问者同时对某一URL地址进行访问。它的测试目标是基于URL的，因此，它既可以用来测试apache的负载压力，也可以测试nginx、lighthttp、tomcat、IIS等其它Web服务器的压力。

 优点对发出负载的计算机要求很低，它既不会占用很高CPU，也不会占用很多内存，使用简单。适合测试简单接口的性能。缺点：对于一些接口要请求头会经常变或者要测试多个接口的时候就很不方便。

命令示例：`ab -c 10 -n 100 http://www.myvick.cn/index.php　：同时处理100个请求并运行10次index.php`（演示）

##### 其他

LoadRunner太重量级且收费，Gatling（Scala语言）需要写代码非界面操作，都不太适用


### Jmeter测试框架具体介绍和简单展示（主要是后端部分）

#### Jmeter相关概念：

1. 执行计划：与本次性能测试相关的所有的内容。

2. Treads（Users）线程 用户  ，分为setup thread group  (预测试操作),  teardown thread group （测试后的动作），`thread group （线程组）`核心，可以看成一个虚拟用户组。

3. 定时器 （Timer） 用来控制等待时间，控制QPS，或者发送的Constant Throughput Timer(吞吐量)

4. 配置元件（Config Element），提供动态请求数据的配置或者是请求头等的配置

5. 断言（Assertions），一般用来测试得到的结果是否正确，判断结果状态。

6. 监听器（Listener），用户对测试结果数据的的处理和可视化展示的一系列元件。包含结果汇总，查看结果树，聚合报告等。

7. 取样器（sample），是性能测试中向服务器发送请求，记录响应的信息，记录响应时间的最少单元，可以认为它是主要去执行任务，发送请求的。

8. 逻辑控制器。控制发送请求的逻辑顺序，如只执行一次，执行多少次，在什么条件（if）下才执行。

9. 测试片段、前置处理器、后置处理器、逻辑控制器(这些用的比较少，具体介绍可以查看官方文档[https://jmeter.apache.org/usermanual/get-started.html](https://jmeter.apache.org/usermanual/get-started.html))

#### 如何创建一个http请求测试


安装Java后，下载Jmeter，解压，然后运行bin/jmeter.bat文件。可以看到：


![jmeter界面概览](https://img-blog.csdnimg.cn/20191202093142295.PNG)

具体过程：

![测试简单的http请求](https://img-blog.csdnimg.cn/20191202094107184.gif)


### 总结，如何简单使用测试

1. katalon recoder(界面录制和测试)

2. jmeter（接口性能测试）

3. postman （接口简单测试和连续请求测试，较为简单，本地使用接口调试和顺便测试）


&emsp;&emsp;自动化测试还有很多东西，这些内容只是冰山一角，之前想着把自动化测试的总体流程都说一下，然后学习之后发现涉及的东西太多，例如：完整的测试用例设计，浏览器驱动webdriver，持续集成等等。
短时间内容几乎不可能学完，所以就以这些目前自己工作中能方便使用到测试工具进行了一下简单讲解。如果有说明得不对的地方，望体谅。

最后，测试用得好，代码bug少。



### 参考及资源

1. 测试社区： [https://testerhome.com/](https://testerhome.com/)

2. [https://blog.csdn.net/xuecancan/article/details/79529155](https://blog.csdn.net/xuecancan/article/details/79529155)

3. 软件测试的艺术，Glenford J Myers  [book]

4. [linux下ab压力测试](https://www.cnblogs.com/myvic/p/7703973.html)


5. [压测相关参数](https://blog.csdn.net/timchen525/article/details/80113749)

	https://www.cnblogs.com/a00ium/p/10462892.html

	https://www.yuque.com/darry/skills/ef9kmx

