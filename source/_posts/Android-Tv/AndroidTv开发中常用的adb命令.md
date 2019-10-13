---
title: AndroidTv开发中常用的adb命令
date: 2017/09/31 21:30:25
categories:
- Android-TV
---

盒子应用开发时，调试比手机上的开发比较麻烦一点，而且需要经常跟 adb 打交道，不管是 wifi 连接调试，还是应用删除安装等。这里记录一些常用的操作，方便查阅。  

#adb wifi连接调试  

####方法一：需要root权限  
在网上下载超级终端工具，然后输入下面命令：  
```  
su  
setprop service.adb.tcp.port 5555  
stop adbd  
start adbd  

```  
超级终端工具在各大应用市场中就可以下载，或者编译运行 github 上的终端应用，附上链接：[Android-Terminal-Emulator](https://github.com/jackpal/Android-Terminal-Emulator)   

如果不想下载终端自己输入命令，可以网上搜索一些别人封装好的工具直接运行，如我自己写的小工具，下载项目编译安装在盒子上运行一下即可。  
[adb](https://github.com/woshidasusu/DasusuDemo/tree/master/Adb)  
如果也不想编译项目，那么试试看可不可以直接下载apk安装，[下载地址](https://github.com/woshidasusu/DasusuDemo/blob/master/Adb/app/build/outputs/apk/app-debug.apk)  


####方法二：需要 usb 连接，不需要 root 权限  
这是针对手机的情况，毕竟盒子如果可以有线连接调试就不用搞什么wifi这么麻烦了，具体步骤见最后的参考链接，这里不介绍了。  

#adb 常用调试  
可以借助 adb 来查看数据库文件等数据，这方面内容感兴趣的可以查阅我之前的博客[【Android】你应该知道的调试神器--adb](http://www.jianshu.com/p/a6dcdb2c74c3)  

#adb 修改 ect/host 文件  
Tv项目的正式上线，预发布还有测试时的服务器地址通常不一样，有时是根据盒子的 host 文件来决定，因此开发期间，通常会有测试和预发布的 host 文件，需要覆盖在盒子的 etc 目录下。但 etc 目录是只读权限的，所以需要 root 权限，而且简单的使用 chmod 命令无法更改 etc 目录的读写权限，需要重新挂载。总之，命令如下：  

```  
adb root  
//命令执行会有提示：adbd is already running as root

adb remount    
//命令执行会有提示：remount succeeded  

adb pull /system/etc/hosts  
//可选，备份原有Host  

adb push ./hosts /system/etc  

```  

#adb 删除系统应用  
如果做的Tv应用是盒子厂商定制的系统应用，那么在开发时需要将盒子原有的系统应用卸载，才能安装你开发的应用，步骤如下：  

```  
1、  mount -o rw,remount /system	卸载系统应用时先运行这句
2、 后把 /system/app 和 /data/data 下的相关文件删掉
3、 reboot重启盒子
4、 安装debug应用 
添加一下、system目录的权限，就能删了

```  

# adb 启动任意 Activity  
一个应用的不同 Activity 可能需要不同的场景下才能打开，比如6分钟不操作出现的待机页、广播打开的页面等等。某些 Activity 如果想按正常场景步骤下打开会特别麻烦，所以可以借助 adb 命令来打开指定页面，或者发送特点广播。  

```  
adb shell am start -n com.vilyever/com.vilyever.TestActivity  
//启动指定的Activity  

adb shell am start -a android.intent.action.VIEW -d vilyever://testactivity  
//启动隐式的Intent  -d 表示发送的data  
```  
命令参数的具体解释参考最后附上的链接，或自行网上查找。  

#参考链接  
[ADB连接方式： wifi与usb](http://blog.csdn.net/dabaoonline/article/details/50802952)  
[Andoird开发调试时不修改Manifest直接启动任意Activity的方法](http://www.jianshu.com/p/54fd9627860a)  
