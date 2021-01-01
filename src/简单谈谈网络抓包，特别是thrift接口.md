
> 按照惯例先谈谈最近情况，最近不是刚好跨年吗？看到很多人都在写年度总结，所以我也在写年度总结文章（其实之前我基本没有写过的，今年有点感触，也想记录一下），结果发现写起来有点多，之前还想着元旦前发出来，结果元旦过后，今天我也还在写，而且还写不完，不过也差不多了，预计明天会发出来。然后最近需求比较多，工作比较忙，同时遇到了一个比较麻烦的需求，就想着记录一下，先充当一下这周的文章任务😂。

###  背景
最近，我在公司做这样的一个事情，整理老旧的api服务，需要统计接口的每个请求的ip和请求参数，找出对应的调用的ip，以便通知业务方修改为新的api服务。其中我们的这个api服务调用有分为thrift调用和http调用，由于之前的旧的服务设计比较简陋，没有通过dubbo 等注册中心调用，也没有完整的监控（不要问我为什么没有，我也很无奈😭），导致我想找出调用ip也找不到，日志里面没有记录。

### 具体操作方法
#### 方法一，直接修改业务代码

刚开始我们是想修改原业务代码，加上调用ip和请求参数的日志打印的。原本以为不会有什么问题，但是我们准备开始动手改造的时候，发现http协议提供的接口，比较好改造，直接 httpRequest 可以获取到客户端ip ，请求参数也很好打印。接着我们打算动手也修改一下thrift 接口的那部分，原本以为都是接口，估计thrift框架也会提供对应的方式获取（我们使用的原生的apche thrift 框架来提供服务），于是网上找了一下，发现thrift 接口好像并 http 接口那么好改造，网上说需要加个监听类，监听请求，类似以下情况：

```java
     /**
	 * 调用RPC服务的时候触发
	 * 每调用一次方法，就会触发一次
	 */
	public void processContext(ServerContext serverContext, TTransport inputTransport, TTransport outputTransport)
	{
		/**
		 * 把TTransport对象转换成TSocket，然后在TSocket里面获取Socket，就可以拿到客户端IP  (这里有可能转化不成功的，inputTransport 不一定为 TSocket 类来的，还有可能为 TFrmef ，这里依赖于thrift 提供服务的方式 )
		 */
		TSocket socket = (TSocket)inputTransport;
		System.out.println(socket.getSocket().getRemoteSocketAddress());
	}

```
参考：两种方式获取Thrift调用的客户端IP地址
 https://blog.csdn.net/mn960mn/article/details/50585278

但是，我们实际操作的时候，建了这个监听类，起了自己的本地服务，并用单元测试去测试的时候，发现报了以下这样的一个错误。
```java
org.apache.thrift.transport.TFramedTransport cannot be cast to org.apache.thrift.transport.TSocket
```
说TFramedTransport 不能转成 TSocket 这个类 ，估计是实例类跟要强转的类不一致。

一开始，还以为自己的写错了，对着网上的博客检查了以下没有问题。只是发现这个实际实例化出来的类不是 TSocket 类型。试了几次后，发现还是不行，于是有根据这个错误在网上找了一下，TFramedTransport 获取 IP<font size=1>(参考这个错误描述： https://www.oschina.net/question/858822_2182069) </font>。这才发现，我们的thrift接口提供方式跟网上说的不一样，而我们这种情况，如果需要获取Client Ip ，有这几种方式：1. 修改thrift 框架源码， 2. 重新继承TFramedTransport类重写里面的一个方法。

看到这两种方法，我们一下子就放弃了，原本想着只是简单加个监控，没想着修改太多东西，如果为了见这样一个监控，改动那么大的，害怕影响业务（我们这个api服务还是挺多业务方在使用的）。

#### 方法二，网络抓包
于是，我们考虑起第二种方案，使用抓包去打印出客户端ip和请求参数。`tcpdump` 是一开始就会想到的工具，当用 tcpdump 去抓包的时候，发现 thrift 接口有些是乱码的。接着就网上找一下是否thrift接口的监控工具，一圈下来，说 thrift-tool  ，wireshark（tshark）可以使用，然后又去试了，安装了挺久的，发现由于我们机器的ubuntu的版本太久了，安装不了最新的 thrift-tool <font size=1>（这个是安装之后，还要依赖tcpdump的版本，tcpdump版本太老也不行，tcpdump命令重装也重装了很久😰，最后发现也安装不了指定版本）</font>和 wireshark ，发现都没有办法使用这两种工具。考虑到我们这个api服务都部署在很多老的物理机上面，考虑兼容性问题和可以方便在其他机器上也部署（我们这个api服务分别部署在10多台的物理机上）。

 于是又重新考虑回tcpdump 命令了<font size=1 color="red">兜兜转转一圈又回到原点, tcpdump 🐮逼</font>。简单实验了一下，抓包的出的thrift请求参数和响应结构，如果是英文的，大部分还是能正常显示的，可以根据 判断是否包含 接口请求参数英文关键字 和判断目标ip和端口，获取到对应的请求参数和的请求ip。如果发送的参数为英文tcpdump直接抓包是能显示大部分参数的，需要完整的参数信息也可以通过直接在业务代码中加监控，thrift 接口在代码中监控发送请求参数还是挺简单的，主要是获取请求ip有点麻烦。

大概命令是这样的（这里假设read为请求参数关键字）
` tcpdump  -i any -A -s 0 'tcp port 9521' | grep -E "read|> [0-9\.]+9521" `

这样就可以获取到 thrift 接口请求到的 9521 端口 的访问ip和请求参数，不过打印出来的日志，还需要自己的去处理一下的。基本上的请求参数的上一行就是该请求的ip。

### 总结
这篇文章主要介绍了解决获取 thrift接口 的请求参数和客户端请求 ip 的问题。算作记录一下自己的解决问题的流程。其中顺便了解一下抓包工具 wireshark 的使用，之前一直有听说过这个软件，但是一直没有实际使用过，感觉的确好用。 另外算作也提醒自己在搭建api服务的时候，记得做好每次接口调用ip和请求ip的监控，不然后面接手的人查起来真的想崩溃，例如：我😪。

这篇也算一篇比较水的文章吧，不过我自己从这次解决问题中还是学到了很多东西的，稍微记录一下：

1. wireshark 过滤可以通过 ` tcp and ip.addr == 127.0.0.1 and thrift` 命令设置在 dispaly filter  来获取目标的请求， thrift 表示协议，不过如果wireshark  版本太低的话，可以识别不了这个协议。
2. wireshark  的命令行版本为： tshark
3. wireshark 的帮助文档地址 https://gitlab.com/wireshark/wireshark/-/wikis/home ,wireshark 的CaptureFilters（捕捉前过滤） 和DisplayFilters（捕捉后筛选） 区别
4.  `sudo tshark -s 512 -i eth0 -n -f 'tcp dst port 9521'  -l` tshark 的命令，捕捉tcp 的请求的，参考： https://www.cnblogs.com/liun1994/p/6142505.html


