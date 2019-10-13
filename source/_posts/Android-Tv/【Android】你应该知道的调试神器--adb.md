---
title: 【Android】你应该知道的调试神器--adb
date: 2016/09/12 12:46:25
categories: 
- Android-TV
---

> **本篇文章已授权微信公众号 dasu_Android（大苏）独家发布**  

***  

最近跟着一个前辈在做TV应用，因为不能通过usb连接调试，接触到了adb，突然间觉得自己似乎发现了另外一个世界，借助adb shell命令对应用进行调试，简直方便得不行。更重要的是，这是命令行操作啊！！！装逼神器啊，还没学的赶紧来试试看吧。  

***  

#效果  

老规矩，先上几张截图看看效果，这是查看xml文件数据，和sqlite数据库数据的效果  
  
![](http://upload-images.jianshu.io/upload_images/1924341-d05d351483037aee.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/1924341-71efb7b5e6663e0b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  


#介绍  

adb,网上介绍其实很多，就是用来对安卓系统进行一些命令操作的工具。如果你的应用需要经常查看 **sharePreference文件数据**、**Sqlite 数据库数据**，以及**本地的各种数据**的话，那么使用adb将会非常方便。  

如果你需要从电脑上发送一些文件到手机里，或者从手机获取一些文件到电脑上（比如视频之类的应用，需要经常把应用存在手机里的视频文件发送到电脑），那么借助adb也可以很方便实现。  

如果你想做一些TV应用的话，那么就应该要学学ADB了，学学如何通过wifi连接调试，如果pull,push文件等等了。  

#使用  

好了，现在就来看看一些常用的命令了，adb 的命令其实很多，不用特意去记，平常要用时上网搜下，等用熟悉了，自然就把一些常用的命令给记住了。下面，稍微介绍一些我经常使用到的命令：  

##基本命令：ls、cd、cat、rm、cp、mkdir  
这些命令是linux系统上的一些基本命令，至少要对 **ls**、**cd**、**cat**这几个命令熟悉点，才能很流畅的使用adb工具，如果你还不熟悉，建议先去了解下这几个命令吧。  

##①adb shell  
这个是进入手机shell操作的一个命令。通常情况下，你调试用的模拟器或者手机通过usb连接电脑后，在win上通过`Ctrl + R`，输入`cmd`，在dos窗口内执行该命令即可进入手机的shell操作。  

如果你连接当前电脑的手机不止一部时，这时就需要借助参数来进行选择指定的设备了。如下图：  

![](http://upload-images.jianshu.io/upload_images/1924341-741a7e66e5959591.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

##②借助ls、cd命令进入应用数据地址：/data/data/{包名如:coder.dasu.meizi}/

该目录下就是存放该应用的 xml数据，cache数据，file数据，以及sqlite数据库数据了，如下：  

![](http://upload-images.jianshu.io/upload_images/1924341-792c43e57224e984.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##③cat命令查看SharePreference的xml数据  

xml中经常保存一些应用的配置数据，比如用户是否首次启动app，用户账户，用户对应用操作的一些设置啊，比如关闭消息推送等等。  
这些数据在开发时，都可以通过log方式打印出来，查看效果是否正确。但有时，如果想要查看较多的xml数据时，又懒得一个个的敲代码，或者log信息太杂，忘记以前写的过滤条件时，这时就可以借助adb来实现了。  

![](http://upload-images.jianshu.io/upload_images/1924341-d05d351483037aee.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##④神器： sqlite3  ***.db命令查看数据库  

以上介绍的一些功能其实就算不借助adb，也可以使用打印log等方式实现。但如果我们开发过程中，需要经常查看一些数据库内的数据时，也可以使用ddms，把db文件导出来借助工具查看，但这样总会麻烦了点，需要每次都进行导出db文件。所以，这时候，如果借助 `sqlite3 `这个命令，将会非常方便。  

执行完 `sqlite3 meizi.db` 后，会进入一个sqlite命令状态，在这里可以使用sql语言来进行查询，也可以使用.help来查看sqlite3提供的一些快速命令.

![](http://upload-images.jianshu.io/upload_images/1924341-d3c85d711dfc4e9c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如，执行 `.table` 可以查看当前数据库所有的表，执行 `.schema` 可以查看创建数据库的sql命令  

![](http://upload-images.jianshu.io/upload_images/1924341-3214de6c4fa35343.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  

上面那图中有两张表，我们看看USER表中有什么数据，可以使用sql命令查询  

![](http://upload-images.jianshu.io/upload_images/1924341-71efb7b5e6663e0b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
ps:查询结果默认是一个记录一行的，也可以设置成list、或上图等各种显示方式，通过`.mode  .header`命令来执行，这些命令都可以通过`.help`来查看说明。  

虽然需要执行sql命令才能查询，但其实也就`select`一下，并不会很复杂，而且还可以借机多接触一下sql语言，学习一下。更重要的是，这很装逼，有没有O(∩_∩)O。不管在同学面前操作，还是操作给不懂这个的老板看，都会让对方觉得你很吊的。  

哈哈，反正我是喜欢上用这个工具就是了，因为最近开发负责的部分很多跟数据库操作相关，而且还经常出现一些bug，需要经常查看数据库内容来定位以及解决bug，所以这个用着是特别方便，相比于以前用导出db文件的方式来的话。  

如果你也有调试数据库这方面的需求，建议你也可以使用这个工具试试看。

##其他功能  

我使用adb工具更多的是用它来查看应用的一些数据。但其实，它还是有很多其他实用的功能的。  

###wifi连接调试  adb connect  {ip}
如果你不想用usb连接调试，可以选择使用adb 连接调试，命令是 `adb connect {ip}` ，需要在同一个局域网内。这个功能也比较实用，但首次连接时，需要另外一些配置，建议可以网上搜索下**adb wifi连接手机**等关键字看看。  

###屏幕截屏  screencap -p  {图片存储地址}  
这个其实直接通过手机截屏再发送到电脑就可以了，但我开发的是TV应用，在盒子上没法截屏，所以这个命令对我来说还是较实用的。  

###获取或推送文件  adb pull/push  
这个也挺实用的，获取手机指定位置的文件到电脑上，或者从电脑发送文件到手机上  

***  
如果上面有什么错误，欢迎指正一下。如果你还知道其他更实用的功能，也欢迎告知一下，题主也是个新手，一起好好学习学习。  
