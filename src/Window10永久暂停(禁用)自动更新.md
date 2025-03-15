

终于彻底设置window10不自动更新了（禁用自动更新）


# 设置成功后的标识
设置成功后，重启电脑再打开就会显示这样的，这个才是禁用成功的标识；
![ ](https://i-blog.csdnimg.cn/direct/7a68eb81227d4a6384f4a96f9ae3866d.png)


之前安装了window 10 ,但是window 10 总是会隔断时间后就更新一次，每次启动时候也会检查是否更新；影响了我打开电脑的速度和影响我使用时的速度，所以我找了一下怎么去停止更新。每次看任务管理都会有window update的服务去占用内存。
![禁用后的任务管理界面](https://i-blog.csdnimg.cn/direct/cd96f03a350b423c95f0a3d74f55ee28.png)


默认window 10 只能的最多暂定更新5周，5周后还是会自动更新。

可以使用下面的这个方法来永久停止更新，我试了一下是比较简单而且有效的。

一共需要操作三个部分的功能；



# 设置步骤
## 一、服务中关闭Windows Update（window更新服务）


 1. 按下「Win+R」打开window运行窗口， 然后输入`services.msc`后按Enter键确认，打开服务列表窗口![ ](https://i-blog.csdnimg.cn/direct/59fb3030359a45719c46a5760c4fe1be.png)


2. 在“服务”列表中找到并且选中“Windows Update”选项，双击，进入详情
 ![ ](https://i-blog.csdnimg.cn/direct/d2503eb9490c43a789a3b2024c933ffc.png)


3. 在“常规”选项卡，将“启动类型”修改为“禁用”；在“恢复”选项卡，将 第一次失败，第二次失败，后续失败都更新为“无操作”；然后点击“应用”和“确定”即可
![ ](https://i-blog.csdnimg.cn/direct/70332b0a888d4c95a75f5c7dd67451cc.png)  

![ ](https://i-blog.csdnimg.cn/direct/3af4be54eeea44b3822c09a8ddf57c4e.png)

## 二、在注册表中禁用Win10自动更新

为了防止Win10自动更新还会死灰复燃，需要在注册表设置中巩固一下。

按下「Win+R」打开window运行窗口，打开运行对话框，输入命名 `regedit`，点击下方确定打开注册表，如下所示
![ ](https://i-blog.csdnimg.cn/direct/0005fbf84b2f46de862b026df8cd7276.png)
在注册表中，找到并定位到 `计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\UsoSvc`，然后在右侧找到Start键，双击修改，把start值改成16进制，值改为4，点击确定，如下所示
![ ](https://i-blog.csdnimg.cn/direct/b80d3469d9524799b1b44ab465c5dada.png)
再在右侧找到 FailureActions键，双击修改该键的二进制数据，将“0010”、“0018”行的左起第5个数值由原来的“01”改为 00，点击下方的 “确定”，如图所示
![ ](https://i-blog.csdnimg.cn/direct/483d401ed2c14c019337d546d2799c13.png)

## 三、在组策略禁用Win10自动更新服务

按下「Win+R」打开window运行窗口，打开运行对话框，输入命名 `gpedit.msc`，点击下方确定打开注册表，如下所示
![ ](https://i-blog.csdnimg.cn/direct/567f5c9c61f04b4ca9450f16962f2b1c.png)
在组策略编辑器中，依次展开 计算机配置 -> 管理模板 -> Windows组件 -> Windows更新 ，在右侧配置自动更新设置中，将其设置为已禁用并点击下方的确定保存即可，如下所示

![ ](https://i-blog.csdnimg.cn/direct/2c187f1512da468e99d61651af262610.png)
在Window 更新下，再选中 “删除使用所有Window 更新功能的访问权限”设置中，将其设置为“已启用”并点击下方的确定保存即可，如下所示

![ ](https://i-blog.csdnimg.cn/direct/a3610a1a7ba74887b7f7339698903136.png)

## 四、在任务计划中禁用Win10自动更新服务
按下「Win+R」打开window运行窗口，打开运行对话框，输入命名 `taskschd.msc`，点击下方确定打开注册表，如下所示
![ ](https://i-blog.csdnimg.cn/direct/d2dabf3f4b994060961103e0682e290e.png)
在任务计划程序的设置界面中，依次展开 `任务计划程序库 -> Microsoft -> Windows -> WindowsUpdate`，把里面的项目都设置为禁用

![ ](https://i-blog.csdnimg.cn/direct/44818703dcfa48b1b83eb78f15584910.png)
如果出现提示：**你所使用的用户账户没有禁用此任务的权限**，请点击这里查看解决方法：你所使用的用户账户没有禁用此任务的权限

如果上述：你所使用的用户账户没有禁用此任务的权限所使用的方法提示：**任务计划程序不可用 任务计划程序将尝试重新与其建立连接**，则点击这里查看解决方法：任务计划程序不可用 任务计划程序将尝试重新与其建立连接的解决办法

当你完成上面的四个步骤，那么你的Windows10自动更新就是彻底被禁用了的，然后你需要重启电脑，可以查看`任务管理器`和点击`配置`，查看是否还有Window Update的进程和对应window更新的界面；禁用成功之后，再打开window更新界面则展示如下图所示：

![Window更新禁用成功界面](https://i-blog.csdnimg.cn/direct/2e81cddcea964a27b42f2ce299ff9cba.png)

## 遇到的错误处理

### 你所使用的用户账户没有禁用此任务的权限
如果你操作任务计划的时候，出现提示 **你所使用的用户账户没有禁用此任务的权限**，则需要（1）右键点击该计划任务项，选择“属性”菜单项

![ ](https://i-blog.csdnimg.cn/direct/64f00a50d21a46b2a45fc8f2e89cc7de.png)
（2）点击“更改用户或组”，点击左下角的“高级”按钮
![ ](https://i-blog.csdnimg.cn/direct/38219e2323f14ccc94ec9339f3a915b7.png)
（2）点击“立即查找”，选中用户Administrator，然后点击确认
![ ](https://i-blog.csdnimg.cn/direct/1533763a8b894f89a87643c08d8a948d.png)
（4）可以看到在选择用户或组窗口中已添加了该用户，点击“确定”按钮。

![](https://i-blog.csdnimg.cn/direct/73101294084d4c7a97cdf3b392910413.png)

（5）回到计划任务项属性窗口中，勾选“使用最高权限运行”，点击“确定”按钮就可以了

![ ](https://i-blog.csdnimg.cn/direct/09159d1670014b8187359752bec138a8.png)
### 任务计划程序不可用 任务计划程序将尝试重新与其建立连接的解决办法
在设置了用户之后，执行禁用任务计划的时候报这个错误** 任务计划程序不可用 任务计划程序将尝试重新与其建立连接的解决办法**，则需要下载PsTools

PsTools下载地址：[https://learn.microsoft.com/zh-cn/sysinternals/downloads/pstools](https://learn.microsoft.com/zh-cn/sysinternals/downloads/pstools)

有时候这个下载不一定能正常下载，可以直接到这里下载  [https://download.csdn.net/download/weixin_37610397/90486727?spm=1001.2101.3001.9500](https://download.csdn.net/download/weixin_37610397/90486727?spm=1001.2101.3001.9500)

下载后将其解压缩到桌面或者其他简短的文件夹中，方便等会cmd进入这个路径。


按下点击window图标或按window键并输入cmd，搜索到cmd并选择以**管理员身份运行**(注意要以管理员身份运行才可以)：
![ ](https://i-blog.csdnimg.cn/direct/b510f1bca4b649da858de056b4559cab.png)
通过cd命令进入Pstools 所在的文件
```shell
cd C:\Users\Administrator\Desktop\Pstools
```
然后再复制粘贴以下命令到cmd，再点击回车
```shell
psexec.exe -i -s %windir%\system32\mmc.exe /s taskschd.msc
```
输入之后，会自动打开任务计划程序界面，然后以此展开`任务计划程序库 -> Microsoft -> Windows -> WindowsUpdate ->`。**右键单击**要禁用的两项` Refresh Group Policy Cache`和`Scheduled Start`，你会发现就可以禁用了，禁用完之后，再关掉cmd界面

![ ](https://i-blog.csdnimg.cn/direct/605a6476ec7d4524b3e1f3939a7291e0.png)

# 参考 
1. 彻底禁止Windows10系统自动更新方法 [https://www.linfengnet.com/skill/1566.html](https://www.linfengnet.com/skill/1566.html)
