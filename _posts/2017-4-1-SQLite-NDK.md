---
layout: post
title: 解决 SQLite 在 NDK 中的直接使用
categories: [Android]
tags: [SQLite, NDK]
description: 这个问题的起因是，某项目需要在 NDK 中使用 SQLite，并且这个库同时也需要在 iOS 端使用。一开始的开发均很顺利，已有文章予以总结，
---

> 
> 

这个问题的起因是，某项目需要在 NDK 中使用 SQLite，并且这个库同时也需要在 iOS 端使用。一开始的开发均很顺利，已有文章予以总结，[点击查看该文章](http://rarnu.com/index.php/2017/03/17/sqlite_cross_platform/)。

但是当程序运行到 Android N 上时，情况就不对了，整个程序直接崩溃，报的错误是 ```Can not load dynamic library "libsqlite.so"```。保险起见，我检查了一下 ```/system/lib``` 和 ```/system/lib64```，确保了 ```libsqlite.so``` 是存在的。那么问题就变成了，无法调用这个存在的库？

经过一番搜索，找到了问题的原因，[点此查看原文](http://ericsink.com/entries/sqlite_android_n.html)，具体的原因是，Android N 以后，不再允许直接调用 ```/system/lib``` 内的内容，而允许调用的库，如 ```liblog.so```，均被移至 ```vendor``` 下，并符号链接至 ```/system/lib```。

所以，```libsqlite.so``` 既便存在，也无法再直接调用了。再深入讲一句，其实 ```libdl.so``` 也无法再使用了，也就是说，在 NDK 中 ```dlopen``` 和 ```dlsym``` 这类函数也已被禁用。

- - -

既然不能动态调用，那解决方案就是静态调用了，我们需要一个 ```libsqlite.a```，并把它静态链接到目标库里。这一步很简单，下载 SQLite 源码后，将它编译成适用于 Android 的 ```libsqlite.a```。

```
$ curl http://www.sqlite.org/2017/sqlite-amalgamation-3180000.zip > sqlite.zip
$ unzip sqlite.zip
```

此时可以得到 SQLite 的源码，总共 4 个文件，写一个 Android.mk 来编译之：

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)
LOCAL_MODULE            := sqlite3-a
LOCAL_MODULE_FILENAME   := libsqlite3
TARGET_ARCH             := arm
TARGET_PLATFORM         := android-14
LOCAL_SRC_FILES         := ./sqlite/sqlite3.c
LOCAL_C_INCLUDES        := ./sqlite
LOCAL_EXPORT_C_INCLUDES := ./sqlite
LOCAL_CFLAGS            := -DSQLITE_THREADSAFE=1
include $(BUILD_STATIC_LIBRARY)
```

同时还需要再写一个 Application.mk 来使用 STL：

```
APP_ABI := armeabi
APP_CPPFLAGS += -fexceptions -frtti
APP_STL := stlport_shared
```

执行一下 ```ndk-build``` 命令即可得到一个 ```libsqlite3.a```

- - -

要完成静态链接，可以很简单的使用 ```linklib``` 这个宏命令，同时修改 ```sqlite3.inc``` 文件，将 ```external``` 指令后的库名称全部删去。

关于 ```external```，详细的用法是：

```
function sqlite3_libversion(): pansichar; cdecl; external 'libsqlite.so'; // 指定从 libsqlite.so 获取函数
function sqlite3_libversion(): pansichar; cdecl; external;  // 从内链的静态库获取函数
```

关于 ```linklib```，详细的用法是：

```
{$IFDEF ANDROID}
  {$LINKLIB libsqlite3.a}
{$ENDIF}
```

此处需要注意的是，我们仅针对 Andorid 平台进行入理，而其他平台上静态链接并没有意义，因此使用 Android 的定义宏将 linklib 包起来即可。这样在编译时，静态库就链接到目标文件里去了。

- - -

到了这一步，可以说是成功了一半，这个时候运行程序，还是会崩的，主要会崩的地方有以下几个：

```
sqlite3.c: 23167: __sync_synchronize();
sqlite3.c: 23842: __sync_synchronize();
```

这两个函数的调用，须注释掉，在这里并不需要使用，而且放着会引起找不到函数的运行时异常。

另一处崩溃在于 Android 老版本的兼容，在 Android M 以后，调用 NDK 时，不再检查 ```__aeabi_d2ulz``` 和 ```__aeabi_d2lz```（虽然这两个函数具体做了什么我也不知道，但是反编译看函数体，是可以直接留空的），而老版本的 Android 会在调用 NDK 时进行导出函数检查，从而引发一个崩溃。要解决这个问题，只需要造出这两个函数即可：

```
{$i debug.inc}
unit osfunc;
{$mode objfpc}{$H+}

interface

{$IFDEF ANDROID}
procedure __aeabi_d2ulz(); cdecl;
procedure __aeabi_d2lz(); cdecl;
{$ENDIF}

implementation

{$IFDEF ANDROID}
procedure __aeabi_d2ulz; cdecl; begin end;
procedure __aeabi_d2lz; cdecl; begin end;
{$ENDIF}

end.
```

这样就完成了对老版本 Android 的兼容。到了这一步，在 Android N 以上以 NDK 调用 SQLite 即告完成。