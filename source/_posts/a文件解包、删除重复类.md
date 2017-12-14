---
title: .a文件解包、删除重复类
date: 2017-12-09 18:57:50
tags:
---

### 步骤

1、解包：
file staticLibrary.a
2、抽离单个架构的.a
lipo staticLibrary.a -thin armv7 -output v7.a
3、抽离.a中的object
ar -x v7.a
4、从.o文件获取.m文件
nm View.o > view.m

删除重复定义
如果在项目中加入多个第三方库后出现类似下面的问题（XXX.o重复定义）：
duplicate symbol _OBJC_CLASS_$_EAGLView in:
/Users/XXXname/Library/Developer/Xcode/DerivedData/XXObjext-gcnzomsbbunnlyfihyndulghsulr/Build/Products/Debug-iphonesimulator/libXXX.a(EAGLView.o)
/Applications/Xcode.app/Contents/Developer/Library/Frameworks/NChart3D.framework/NChart3D(EAGLView.o)
ld: 2 duplicate symbols for architecture i386
clang: error: linker command failed with exit code 1 (use -v to see invocation)
我们可以把其中一个.a中的.o删除掉来解决这一问题，解决方法步骤：
把一个.a命名为libx.a，在终端中cd到.a所属文件夹下，输入命令来查看包（.a）信息：lipo -info libx.a
如果提示fat file，那么代表这个包是支持多平台的，例如armv7，armv7s，i386等，这需要我们逐一做解包重打包操作。否则我们只需要做一次[1-6]操作即可。
1、创建临时文件夹，用于存放armv7平台解压后的.o文件：mkdir armv7
2、从.a中取出armv7平台的包：lipo libx.a -thin armv7 -output armv7/libx-armv7.a
3、查看armv7平台的包中所包含的文件列表：ar -t armv7/libx-armv7.a
4、从armv7平台的包中解压出object file（即.o后缀文件）：cd armv7 && ar xv libx-armv7.a
5、找到冲突的包（EAGLView），删除掉rm EAGLView.o
6、重新打包object file（armv7平台包）：cd .. && ar rcs libx-armv7.a armv7/*.o，可以再次使用[2]中命令确认是否已成功将文件去除
7、将其他几个平台(armv7s, i386)包逐一做上述[1-6]操作
8、重新合并为fat file的.a文件（去掉重复.o的）：lipo -create libx-armv7.a libx-armv7s.a libx-i386.a -output libMiPushSDK-new.a
9、拷贝到项目中覆盖原来的.a文件就可以了。
我的问题虽然只有在i386下重复定义，但我还是把armv7和arm64下的也删除掉了。