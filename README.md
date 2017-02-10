#说明

1、基于微信iOS 6.5.4版本

2、依然需要借助越狱的设备实现（打包后可以在非越狱设备安装）

3、本文大部分取自[(一步一步实现iOS微信自动抢红包(非越狱)](http://www.jianshu.com/p/189afbe3b429)，在其文的基础上跑了一遍流程；也根据理解简化了其中的部分工作：）

4、仅用于学习交流使用

5、本文不讲解如何找到抢红包的方法

#准备

##一、硬件

1、Mac 电脑

2、已越狱的iPhone 5 （iOS 7.1.2）

3、未越狱的iPhone 6s (iOS 10.2.1)

##二、软件

Mac电脑需要准备的：

1、Mac OS自带的终端

2、[yoyolib](https://github.com/KJCracks/yololib)（用于向iOS的可执行文件中注入dylib）

3、[dumpdecrypted](https://github.com/stefanesser/dumpdecrypted) （用于解密iOS的可执行文件，即砸壳-可不需要）

4、[iOSOpenDev](http://iosopendev.com/download/) （Xcode增强工具，需要通过它生成用于注入的dylib）

5、[iTools](http://pro.itools.cn/pro_mac/) （通过它，将打包后的安装包安装到手机里）

6、otool  （一般Mac OS X自带，用于查看解密后文件的解密情况）

7、[iOS App Signer](https://github.com/Urinx/iOSAppHook/releases)  （重新签名并生成ipa文件）



已越狱的iPhone需要准备的

1、OpenSSH (越狱后通过Cydia安装，安装后可以实现远程登录)

2、Cycript （越狱后通过Cydia安装，这个工具可以在命令行下实现与应用的交互）

3、PP助手   (用于在手机上安装PP助手中的微信客户端)

##三、其他

1、苹果开发者账号

开始

准备工作完成后，可以工作了

一、砸壳

砸壳可以选择两种方式：自己砸壳 和 利用PP助手

第一种：自己砸壳（不推荐）

1、首先使用已越狱的iPhone，通过App Store下载微信客户端（当前是6.5.4版本）


安装微信客户端
2、确保已越狱的iPhone和Mac处于同一局域网，打开Mac OS的终端命令行工具

3、输入ssh root@192.168.1.121远程登录已越狱的iPhone，root密码默认为alpine（其中192.168.1.121是iPhone的局域网IP地址）


远程登录越狱的iPhone
4、在iPhone上运行一下微信，之后执行ps -e | grep WeChat查找WeChat可执行文件的路径，并记录为：可执行文件路径


查找微信可执行文件的位置
5、通过Cycript查找到WeChat的Documents路径，输入cycript -p WeChat,进入cycript命令行状态


cy命令行状态
6、输入NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,NSUserDomainMask,YES)[0],就可以获取到WeChat的Documents路径了，并记录为：Documents路径


WeChat Documents路径
记录下刚才的两个路径，接下来需要将砸壳工具dumpdecrypted拷贝到WeChat的Documents目录下用于砸壳

7、将命令行切换回Mac OS X

输入scp dumpdecrypted.dylib root@192.168.1.121:Documents路径


拷贝dumpdecrypted到Documents目录
8、重新远程登录到iPhone，使用dumpdecrypted.dylib砸壳，具体用法：

DYLD_INSERT_LIBRARIES=/Documents路径/dumpdecrypted.dylib 可执行文件路径


dumpdecrypted砸壳过程
出现了如截图页面则表示砸壳成功，会在命令行执行的当前路径下生成WeChat.decrypted文件


生成的WeChat.decrypted文件
9、将生成的WeChat.decrypted使用scp命令拷贝到Mac电脑上，和之前从Mac电脑拷贝dumpdecrypted.dylib到手机上类似的语法。(下图中的黑色覆盖部分替换为你的Mac登陆用户名)


拷贝解密文件到Mac
10、为保险起见，可以对WeChat.decrypted文件检查一下，查看砸壳是否成功

如截图第一个cryptid 0表示armv7架构已成功，第二个cryptid 1表示arm64未成功

理论上只要把最老的架构解密就可以了，因为新的cpu会兼容老的架构；所以这里arm64未成功不影响


查看砸壳情况
11、再次远程连接iPhone，拷贝出WeChat.app待用（注意使用scp -r）


拷贝WeChat.app到Mac电脑
第二种：利用PP助手（推荐）

在已越狱的iPhone上安装PP助手后，从PP助手中下载微信客户端

PP助手上面的App的可执行文件都是经过了砸壳的,所以可以省去自己砸壳的步骤

我们可以按照自己砸壳的步骤，导出PP助手的微信可执行文件，利用otool查看砸壳情况


PP助手中WeChat砸壳情况
这表示armv7和arm64均已解密

同样再次远程连接iPhone，拷贝出WeChat.app待用

为什么需要砸壳？

因为苹果会将上线的iOS App进行加密，导致无法往其中写入编写好的抢红包dylib。
二、生成并注入dylib

一、下载并安装iOSOpenDev


安装iOSOpenDev
因为我使用的是Xcode 8，直接安装iOSOpenDev会失败。


安装iOSOpenDev失败
如果你们也遇到类似的问题，可以尝试下载iOSOpenDev_Patches，然后按照如下步骤：

1、把Specifications1文件夹重命名为Specifications放到/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/Library/Xcode/

2、把Specifications2文件夹重命名为Specifications放到/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/Library/Xcode/

3、把usr3重命名为usr放到/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/

4、安装iOSOpenDev

二、创建dylib工程并生成dylib

安装完iOSOpenDev后，在Xcode中选择Cocoa Touch Library。


创建Cocoa Touch Library工程
点击Next开始新建工程，将product name命名为autoGetRedEnv；

删除autoGetRedEnv.h文件，修改autoGetRedEnv.m为autoGetRedEnv.mm，然后在项目中加入CaptainHook.h；

从github中下载autoGetRedEnv工程源码中的CaptainHook.h和autoGetRedEnv.mm拷贝源码，复制到工程中对应文件中，完成后进行(真机)编译，即可得到libautoGetRedEnv.dylib文件

三、注入dylib

通过yoyolib向WeChat注入dylib （相当于告知WeChat有这么个东西了，到时候安装包你可以加载这个dylib）

./yololib 目标可执行文件 需注入的dylib

将WeChat.decrypted文件重命名为WeChat（建议备份好WeChat.decrypted文件，以便后续使用）


注入dylib
上图效果则代表注入成功了

三、重新签名并安装ipa

一、重新签名并生成ipa

在重新签名之前，需要将三个文件，放入到之前备份好的WeChat.app文件夹中：

1、embedded.mobileprovision

2、libautoGetRedEnv.dylib

3、注入libautoGetRedEnv.dylib后的WeChat文件

embedded.mobileprovision文件就是苹果开发者网站中的Provisioning Profiles文件,如果之前创建好了直接下载重命名为embedded.mobileprovision即可。

下载后双击导入Xcode确保iOS App Signer能够选择到这个mobileprovision

之后使用iOS App Signer重新签名

Input File  -> 选择刚才已经放入了新的可执行文件的WeChat.app

Signing Certificate -> 开发者签名证书

Provisioning Profile -> 选择下载并导入的mobileprovision


重新签名
点击Start开始进行打包，如下图表示生成成功


生成WeChat.ipa

二、使用iTools安装打包的ipa


安装打包的WeChat.ipa
抢红包开始，基本都是0秒抢到：




0秒抢红包



开启红包插件