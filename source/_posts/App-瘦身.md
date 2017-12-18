---
title: App 瘦身
date: 2017-12-18 13:50:36
tags:
---
在我们提交安装包到App Store的时候，如果安装包过大，有可能会收到类似如下内容的一封邮件：
![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkvm8zz33j20v00manf6.jpg)

收到这封邮件的时候，意味着安装包在App Store上下载的时候，有的设备下载的安装包大小会超过100M。对于超过100M的安装包，只能在WIFI环境下下载，不能直接通过4G网络进行下载。

在这里，我们提交App Store的安装包大小为67.6MB，在App Store上显示的下载大小和实际下载下来的大小，我们通过下表做一个对比：

| iPhone型号 |     系统    | AppStore 显示大小 | 下载到设备大小 |
|:--------|:--------|:--------|:--------|
| iPhone6 | 10.2.1 |  91.5MB | 88.9MB |
| iPhone6 | 10.1.1 |  91.5MB | 88.9MB |
| iPhone6 |  9.3.5 | 91.5MB | 84.8MB |
| iPhone 5 |  9.2 | 91.5MB | 84.8MB |
| iPhone6 plus | 10.0.2 | 95.7MB | 93.2MB |
| iPhone7 plus | 10.3.0 | 95.7MB | 93.2MB |
| iPhone5C | 9.2 | 83.9MB | 76MB |
| iPhone5S | 7.1.1 | 147MB | 144MB |
| iPhone5C | 7.1.2 | 147MB | 未知 |
| iPhone5C 越狱 | 8.1.1 | 83.9MB | 144MB |

从上表可以看到：

* 在 iOS 9 系统以上的手机上，App Store 上的大小都是做了 App Thinning 操作的。
* 在 iOS 9 以上系统的基础上，plus 手机在 AppStore size 上比其他手机大了 4.2MB，猜测是因为 @3x 图的原因。
* iOS 9 和 iOS 10 虽然在  AppStore 显示的包大小一致，但是最终下载到手机上，大小有区别。
* iOS 9 以下的手机，是直接下载整个安装包的。

优化安装包分为如下几个步骤：

1. 分析安装包的构成，一个安装包分为二进制代码文件，资源，配置文件。需要知道各个方面的占比。
2. 知道各个方向的优化策略，譬如二进制文件如何优化，资源文件如何优化
3. 执行优化，得出结果

## 分析

首先进行第一步，分析安装包的构成： 88M的安装包解压后变成220MB。
ipa是一个压缩包， 安装包里的主要构成是（图片+文档+二进制文件），我们下面的分析

#### 1.图片优化：

![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkvw4ryw7j20t20fcn0d.jpg)
从上面来看，图片的压缩比最小。几乎没有压缩，这也说明每减少一张图片，就实实在在的减少了ipa的大小。 为了验证上面的数据，我们来做一些实验： 我们新建一个项目， 测试资源图片对安装包的大小的影响： 目录结构如下：
![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkvwlr2shj20ei0men2d.jpg)
其中资源信息如下：
![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkvx15pbzj20qu0aigp4.jpg)

然后进行打包Archive→Export.得到IPA文件
![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkvxhd0f7j20mq0bmwgg.jpg)
从上面的结果来看，安装包的大小基本等于图片资源的大小，可以看一下IPA的内容详情视图（下图）， 发现图片确实没有怎么压缩：
![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkvybjhy7j21ho0vktjo.jpg)

###### 结论1：JPG资源图片的压缩比很小，每减少一张图片，就实实在在的减少了ipa的大小。

下面我们进一步使用ImageOptim对图片进行压无损缩优化（如下图）。看能否优化下安装包大小。
![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkvysm33aj20qo0c0jtw.jpg)

压缩后，总文件大小为：屏幕快照 2016-10-25 下午8.58.24.png， 优化掉了1MB的大小。 我们然后进行打包操作，最终的安装包确实也小了0.8MB，从11.6MB变成了10.8MB，。还是有优化的，如下图所示：
![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkvzi0gdpj207c054aaf.jpg)

此时我们看xcode里的工程配置，COMPRESSPNGFILES 是YES的，有一些说法是这个变量的设置和ImageOptim冲突， 这里看起来不是如此。

###### 结论2：ImageOptim有时候还是确实能优化资源大小，进而减少安装包大小的。

是否因此可以完全确定ImageOptim的优化能力， 我觉得看情况而定， 上面的几种图片都是我iPhone手机里的相片导出的。是JPG的格式。

我们再对PNG做一些测试, 找一些资源图片放到工程中（我就不截图了，直截大小）：
![](http://ww1.sinaimg.cn/thumbnail/68e28bf3ly1fmkw13f20sj20tk04cmxj.jpg)
打包后的大小是：3.3M
系统帮我们优化掉1.1MB。 同样我们队图片进行一轮无损压缩优化， 经过ImageOptim优化后效果：
我们进行打包，得到的安装包的大小是:还是2.2MB（特意将系统优化关闭）：

###### 结论3： 对于我找的这几个png图片，ImageOptim的优化没有起到作用。

在我们项目中也对所有资源图片使用ImageOptim进行了优化，20多MB的资源最后优化掉1MB左右， 这里怀疑ImageOptim对PNG的优化能力一般。 而且如果能优化的PNG资源，系统默认可以进行一些优化。

### 2.文档资源

![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkw2qelafj20rq0f6tbj.jpg)

从上面来看，文档有一定的压缩比，大概40%，也就是如果工程里有40MB的文档，体验到压缩包里大概是40*0.6=24MB。

### 3.二进制安装包

![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkw3e7zfjj214i026jrt.jpg)
二进制代码的压缩率是最高的。

上面都是研究不同的资源对最后安装包大小的影响情况，现在有了一些理论依据，我们就可以开始对安装包进行优化工作了。

## 实践


所以先从图片开始优化吧：

### 1.如何优化图片：
1. 使用一些工具来检测unused 文件。 比较推荐的是github上的一份开源代码：
[https://github.com/tinymind/LSUnusedResource ](https://github.com/tinymind/LSUnusedResource) 编译运行看到的页面是这样的：

![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkw4vpoxij210o0xggq5.jpg)

指定搜索路径， 第二行（EXClude）指定哪些文件夹路径不被扫描。

然后这些资源可以清除掉了。但是存在着图片被误删除的可能性，譬如代码中使用图片的方式是：[UIImage imageNamed:[NSString stringWithFormat:"icon_%d.png",index]]； 这种情况下，图片可能被误删，所以删除的时候不妨10张一组的进行，用眼睛过滤一遍。避免因为图片名字拼接，删除搜索出的结果。

2. 删除完图片，是否还可以对已有的图片做优化呢。

现在网上有非常多的关于ImageOptim对资源图片进行无损压缩的方式， 在文章开头部分我们也对ImageOptim的优化能力进行了一些验证，通过上面的实验，我觉得结论不能确定，但值得一试。

### 2.文档资源的优化

文档资源主要是排查：

1. 是否有不必要的文档资源，如果过期的旧版本所需要的文档资源 清理即可。
2. 优化文档资源大小，主要是优化精简文档内容。

### 3.二进制包优化

二进制包是由各种代码文件，静态库 动态库 经过编译后生成的可执行文件。 以头条二进制包125MB为例， 他是如何组成的?
![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkwaicemwj20w40dcwh2.jpg)

上图可以看到armv7 占可执行文件的58MB。 arm64占可执行文件的66MB。 加起来=125MB。 进一步分析

![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkwazvcv9j21oa150kbc.jpg)

通过右侧的pfile偏移可以大概算出每个段的大小，但不直观， 我们可以通过开启一些编译选项，生成可执行文件结构，然后借助一些工具生成更加直观的

1. XCode开启编译选项Write Link Map File XCode -> target -> Build Settings -> 搜map -> 把Write Link Map File选项设为yes，并指定好linkMap的存储位置：
![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkwbj0pffj21l609q0un.jpg)

2. 编译后到编译目录里找到该txt文件，文件名和路径就是上述的Path to Link Map File。
~/Library/Developer/Xcode/DerivedData/XXX-eumsvrzbvgfofvbfsoqokmjprvuh/Build/Intermediates/XXX.build/Debug-iphoneos/XXX.build/。 这个LinkMap里展示了整个可执行文件的全貌，列出了编译后的每一个.o目标文件的信息（包括静态链接库.a里的），以及每一个目标文件的代码段，数据段存储详情

![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkwbxzzd3j213y0qmn84.jpg)

1. 如上图，weinAppID这个函数占得内存大小为1*16 + 13 = 29字节。 这样看下去也挺麻烦的， 找个工具归类一下。

归类，去[https://github.com/huanxsd/LinkMap](https://github.com/huanxsd/LinkMap) 下载这个mac工程 然后运行。

![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkwczkqhhj20ye0mwq7z.jpg)

![](http://ww1.sinaimg.cn/large/68e28bf3ly1fmkwd3jhpsj20x80v0797.jpg)

譬如内部播放器sdk MediaPlayer在arm64架构下大小为4.92MB， armv7也可以分析另一份armv7 linkmap文件大概4.5MB 二者加起来就是在二进制占据的总大小—10MB左右。 通过对上面的文件进行分析，就知道每个类在最终的可执行文件中占据的大小。 然后有针对性的进行优化就可以了。

### 4.编译选项优化：
如果项目是很早之前（xcode4，5）建立的，迭代到现在 的确可以检查一下有利于减少安装包的编译选项：

1. Optimization Level 使用Fastest, Smalllest
该选项对安装包大小影响几无，但可以提高app的性能。参考wwdc 2013-Session408 Optimize Your Code Using LLVM
2. Strip Linked Product 设置为YES
需要注意的是Strip Linked Product也受到Deployment Postprocessing设置选项的影响。在Build Settings中，我们可以看到， Strip Linked Product是在Deployment这栏中的，而Deployment Postprocessing相当于是Deployment的总开关。记得把Deployment Postprocessing也设置为YES， 该选项对安装包大小的影响非常大， 以头条客户端为例，如果不开启此设置，ipa大小是48MB，上线后appstore上显示的大小是65MB， 我们开启了此配置后，ipa大小变成40MB， appstore上显示45MB。 优化效果还是非常明显的。 PS：Deployment Postprocessing这个配置项如果使用xcode打包，xcode会默认把这个变量置为YES， 如果使用脚本打包，记得设置。
3. Symbols Hidden by Default设置为YES
4. Make Strings Read-Only 设置为YES

### 5.可能无效的办法：
将Enable C++ Exceptions和Enable Objective-C Exceptions设为NO，去掉异常支持； 如果你的项目比较大，有很多try cache， 想去掉这些异常可能是一个比较大的工作量，我在头条项目里尝试去掉了所有异常，打包测试，安装包大小没有 变化， 因为只是一个项目的测试，我只能比较怀疑去掉异常对最终安装包大小的优化能力。

### 6.一些额外的建议
iOS的项目，在安装包没有大到一定程度之前，研发和产品一般都关注较少，我们在平时的开发中也会有一些不好的习惯，在这里枚举一下：

1. 不要为了一片树叶 引入一片森林。 有时候你只想接入一个base64编码的函数，结果接入了一个有几十个类文件的util库，除了你用到的这个函数，其余的可能再也不会有人用到， 另一个研发为了另一个RSA加密需求，结果又引入了另一个一个extensions库 。那些类文件都是成本，类文件，函数，甚至不同长度的函数名字都对最终的安装包大小产生影响。 积少成多， 安装包会越来越大。 而且代码量越大，app启动时候DYLD链接的工作量越大，启动耗时也会变长。
2. 对于要接入到app的资源文件，要check一下大小，一个100*100大小的图片如果几十MB 肯定设计师在切图的时候有些问题，打包后也要check一遍。对于集体打包到Assert.car里的文件，可以找工具解开看一下。 我们遇到过打了一次包后 发现安装包大了几十MB，经过分析，发现是有一张图片意外的大，找人优化后 变成几百k。
3. 注意平时的开发习惯，废弃模块及早清理。
4. 同质的开源库（譬如AFnetworking vs ASIHttpRequest），只接入一种。
5. 建立预警机制， 一般上线后，都是脚本打包，除了正常的生成ipa包之外，也要生成分析文件， 列出相对上一次上线包大了多少，类文件增加了多少。这样的机制也有助于防止安装包悄无声息的变成巨物。
