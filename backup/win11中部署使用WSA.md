**环境和背景**
新电脑和win11系统，尝试部署使用，将过程记录如下：

**准备工作**
1、启用Windows虚拟化支持
        进入设置➡应用➡可选功能➡更多Windows功能，找到并开启勾选“Hyper-v”和“虚拟机平台”两个选项，点击确定之后按照提示重启电脑即可完成功能安装。

2、安卓ADB命令行调试配置
1）ADB工具-Android SDK Platform Tools
Android SDK Platform Tools就是人们常说的adb命令行工具。它是安卓设备与电脑交互沟通的桥梁，没有它很多软件无法正常工作。可以通过以上链接根据自己的系统下载安装。

2）下载配置adb驱动环境
        将下载的adb命令行工具解压到合适的位置，笔者这里选择“D:\Program Files\platform-tools”；由于如需在命令行中使用adb，需要在Windows中设置环境变量，下面将以笔者的目录为例，读者根据自己的选择自行更改。

        添加环境变量：右键点击“此电脑”➡属性➡高级系统设置➡高级➡环境变量，在“系统变量”中点击新建，填入变量名：adb，变量值：D:\Program Files\platform-tools，之后点击确定即可。

        添加Path变量：在“系统变量”中找到“Path”项目，点击“编辑”➡“新建”，在其中输入%adb%，随后点击确定保存。

3）检查adb工具是否部署成功
        打开Windows终端或者cmd命令行，输入adb version命令并回车，如果环境变量配置正确，那么就会显示adb的版本号。
```
C:\Windows\System32>adb version
Android Debug Bridge version 1.0.41
Version 31.0.3-7562133
Installed as D:\Program Files\platform-tools\adb.exe
```
如果有错误，请仔细检查文件解压位置和系统变量是否一致以及环境变量和Path值是否填写正确。

**win11正式版安装WSA**
在win11正式版中，可以通过“下载WSA离线安装包”的方法安装部署安卓子系统，从而直接绕过地区和测试版限制。

微软应用商店安装包提取: 
https://store.rg-adguard.net/
输入
https://apps.microsoft.com/detail/9P3395VX91NR?hl=en-us&gl=US
后点击对勾，最下方找到文件
“……_2311.40000.5.0_neutral_~_8wekyb3d8bbwe.msixbundle”
（更新后文件名可能不同）进行下载。

右键点击下载的文件选择“复制文件地址”，之后以管理员身份运行PowerShell，使用Add-AppxPackage命令进行安装，具体如下：
```
Add-AppxPackage "C:\Users\5002278\Downloads\MicrosoftCorporationII.WindowsSubsystemForAndroid_2407.40000.4.0_neutral_~_8wekyb3d8bbwe.msixbundle"
# 引号中内容为文件路径名称，根据自己的需求填写回车并等待安装完成即可。
```

**使用ADB命令在win11上安装apk**
打开WSA安卓子系统设置页面，打开“开发人员模式”选项，记下下图中的WSA的内部IP地址和端口号。
打开Windows命令行，输入以下命令：

```
#测试adb命令是否正确配置
adb version
```
#执行命令能看到adb版本号则为正确，否则请检查adb设置
```
#链接WSA（后方IP为刚刚显示的IP和端口号）
adb connect 127.0.0.1:58526

```
```
#安装APK
adb install “APK完整路径“
# 注意 .apk 的路径最好无中文且无空格，否则需要用英文双引号包裹。
# 你可在资源管理器上右键点击 apk 文件选「复制文件地址」获取完整路径
```
这样就能使用adb命令安装apk文件到WSA了，为使用方便，可以安装国内应用市场，比自带的亚马逊应用市场更为好用。

**遇到的问题**
1、adb无法连接
某次尝试连接adb时提示“cannot connect to 127.0.0.1:58526: 由于目标计算机积极拒绝，无法连接。 (10061)”
其原因是端口没有真正被启用，大概解决方法是在WSA中运行一个程序，曲线启动对应端口，具体操作参考下面博主的文章：

https://www.cnblogs.com/yuhangch/p/wsa-cant-connect-58526.htmlicon-default.png?t=N7T8https://www.cnblogs.com/yuhangch/p/wsa-cant-connect-58526.html