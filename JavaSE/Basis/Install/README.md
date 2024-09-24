# 安装 JDK

- [Windows 安装 JDK](Windows/README.md)
- [MacOS 安装 JDK](MacOS/RE)

 卸载JDK 

根据需求再决定是否需要卸载（在控制面板中卸载程序，而不是直接删除安装目录），我们目前要求统一安装JDK17版本，每个同学安装的JDK版本必须统一，如果已经正确安装JDK的同学可以略过安装步骤。

 6.3.2下载JDK 

下载：Download J2SDK （Java 2 Software Development Kit）FROM    [http://java.sun.com](http://java.sun.com/)    OR   [http://www.oracle.com](http://www.oracle.com/)

注意: 下载官网中的内容都是英文，高级浏览器中可以把英文翻译为中文。

 6.3.3安装JDK 

找到JDK安装程序，双击进行安装操作。

用户帐户控制

您要允许以下程序对此计算机进行更改吗?

程序名称:

JAVA(TM) SE DEVELOPMENT KIT 17.0.4

已验证的发布者:ORACLE AMERICA,INC.

文件源:

已从INTERNET下载

是(N)

否(N)

显示详细信息(D)

更改这些通知的出现时间

![图片1.png](https://cdn.nlark.com/yuque/0/2023/png/21376908/1691738300781-3c56f266-4e6a-4434-9172-d819cfc3d786.png?x-oss-process=image%2Fformat%2Cwebp)



然后点击“是”，进入下一步的安装。

JAVA(TM) SE DEVELOPMENT KIT 17.0.4(64-BIT)-安装程序

JAVA

ORACLE

欢迎使用JAVASE开发工具包17.0.4的安装向导

本向导将指导您完成JAVASE开发工具包17.0.4的安装过程.

下一步(N)

取消

![图片2.png](https://cdn.nlark.com/yuque/0/2023/png/21376908/1691738341287-20ff0113-da08-4d37-a6b0-d5eae9f27ffd.png?x-oss-process=image%2Fformat%2Cwebp)



接着点击“下一步”，继续进行下一步的安装。

JAVA(TM) SE DEVELOPMENT KIT 17.0.4(64-BIT)-目标文件夹

LAVA

ORACLE

这会安装JAVA(TM)SE DEVELOPMENTKIT 17.0.4(64BIT).它要求硬盘驱动器上

有420MB空间.单击更改按钮可更改安装文件夹.

设置JDK安装的位置(路径不允许中有中文和特殊字符)

将 JAVA(TM)SE DEVELOPNENTKIT 17.0.4 (64-BIT)安装到:

D:ISOFTWARE JAVALJDK1.17\

更改..

下一步(N)

上一步(B)

取消

![图片3.png](https://cdn.nlark.com/yuque/0/2023/png/21376908/1691738377490-46637754-44ea-42ba-94c9-cd37755fe3de.png?x-oss-process=image%2Fformat%2Cwebp)



在此处我们需要设置JDK安装的位置，要求JDK安装目录不允许中中文和特殊字符，并且JDK安装的目录必须记住，后期还需要配置JDK的环境变量。

JAVA(TM) SE DEVELOPMENT KIT 17.0.4(64-BIT) -完成

JAVA

ORACLE

JAVA(TM)SE DEVELOPMENTKRT 17.0.4(64-BIT)已成功安装

单击后续步骤"访问教程,API文档,开发人员指南,发布说明及更多内容,帮助您

开始使用JDK.

后续步骤(N)

关闭(C)

![图片4.png](https://cdn.nlark.com/yuque/0/2023/png/21376908/1691738414411-1103aa35-26e9-4e81-a2ee-8e6451d1ec8e.png?x-oss-process=image%2Fformat%2Cwebp)



点击“确定”按钮，则意味着JDK安装成功。



![img](https://cdn.nlark.com/yuque/0/2023/jpeg/21376908/1692002570088-3338946f-42b3-4174-8910-7e749c31e950.jpeg?x-oss-process=image%2Finterlace%2C1%2Fformat%2Cwebp%2Fresize%2Cw_1066%2Climit_0%2Finterlace%2C1)



 6.3.4验证JDK 

在JDK安装目录的bin目录下，我们输入命令：java –version

出现如下图所示，则安装成功：

D:\SOFTWARE JAVA JDKL.17>JAVA -VER

-VERSION

JAVA VERSION "17.0.4" 2022-07-19 LTS

JAVA(TM> SE RUNTIME ENVIRONMENT (BUILD 1778

D 17.0.4+11-LTS-179>

JAVA HOTSPOT(TH) 64-BIT SERVER UM (BUILD 17.8.4+11-LIS-179, NIXED MODE, SHARING>

![图片5.png](https://cdn.nlark.com/yuque/0/2023/png/21376908/1691738493975-6c1852ad-0ed0-41fc-bb09-f79c31cfc4a0.png?x-oss-process=image%2Fformat%2Cwebp)



 6.4环境变量配置 

 6.4.1配置path环境变量 

JDK安装完毕后，我们想要操作JDK安装bin目录中的可执行程序，则必须通过DOS命令窗口切换到JDK安装目录中才能使用，如果想要在任意目录中都能操作JDK安装bin目录中的可执行程序，则必须配置path环境变量，其配置步骤步如下：

1步骤1：在桌面上找到“计算机”图标，右键找到“属性”后，点击属性。

2步骤2：在属性界面“左侧”、“中间”或“右侧”找到“高级系统设置”。

3步骤3：点击“高级系统设置”，打开系统属性，找到“环境变量”后并打开。

4步骤4：打开“环境变量”后，在系统变量中找到“Path”变量，然后双击打开

5步骤5：点击“新建”，并添加JDK安装bin目录，并移动到最前面，并点击“确定”按钮。

【注意】：配置好path环境变量后，则一定要点击“确定”按钮来保存，然后再重启DOS命名窗口，重启后新配置的环境变量才能生效。

【测试】：打开DOS命令窗口，在任意文件目录（未必在bin目录下）下输入命令：javac 

出现如下图所示，则配置成功：

C:WSERS ADMINISTRATOR>.

JAVAC

用法:JAVAC

<SOURCE FILES>

AC<OPTIONS>

其中,可能的选项包括:

生成所有调试信息

-G

不生成任何调试信息

-G:NONE

中国工业有限公司

不只不输输

主生出出指指指指覆

-G:LINES,VARS,SOURCE>

钱荷警告

-NOWAPN

器正在执行的操作的消,息

关编译器正

-VEPBOSE

使用已过时的 APT的源位置

DEPRECATION

-CLASSPATH<路径>

<路径>

-CP

主查找输入源文件的位置

H<路径>

-SOURCEPATH

-BOOTCLASSPATH <路径>

<目录>

覆盖所安装扩展的位置

-EXTDIRS

控制意奢象职控转贷通射位及精译.

<目录>

ENDORSEDDIRS

PROC:{NONE,ONLYY

OR(CLASS1>(CLASS27,<C1ASS37..1要运行的注释处理程序的名称;绕过默

PROCESSOR

人的搜索进程

<路径>

指定查找注释处理程序的位置

PROCESSORPATH

生成元数据门由千方注鑫数的反射

![图片6.png](https://cdn.nlark.com/yuque/0/2023/png/21376908/1691738597370-b200c658-8a7f-4811-ac93-e81713145ebd.png?x-oss-process=image%2Fformat%2Cwebp)







 配置JAVA_HOME环境变量 

每次JDK的安装目录发生了变化，则都需要在path环境变量中修改JDK安装bin目录中的路径，那么这样就有可能误改path环境变量中的别的路径，从而造成某些程序的环境变量失效。

为了解决这个问题，我们就需要配置JAVA_HOME环境变量，设置JAVA_HOME环境变量的值为“JDK安装目录”，然后再把path环境变量中的路径设置为“%JAVA_HOME%\bin”即可，也就是通过“%JAVA_HOME%”来引用JAVA_HOME的变量值，具体步骤如下：

1步骤1：在桌面上找到“计算机”图标，右键找到“属性”后，点击属性。

2步骤2：在属性界面“左侧”、“中间”或“右侧”找到“高级系统设置”。

3步骤3：点击“高级系统设置”，打开系统属性，找到“环境变量”后并打开。

4步骤4：在系统变量中新建“JAVA_HOME”变量，设置变量值为“JDK安装目录”。

5步骤5：然后打开path环境变量，将JDK环境变量设置为“%JAVA_HOME%\bin”即可。

【注意】：配置好JAVA_HOME和path环境变量后，则一定要点击“确定”按钮来保存，然后再重启DOS命名窗口，重启后新配置的环境变量才能生效。

【测试】：打开DOS命令窗口，在任意文件目录（未必在bin目录下）下输入命令：javac 

出现如下图所示，则配置成功：

![图片7.png](https://cdn.nlark.com/yuque/0/2023/png/21376908/1691738662991-a0d68a1d-648d-4b02-b8fc-9f6780a1f337.png?x-oss-process=image%2Fformat%2Cwebp)