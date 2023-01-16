---
title: DBUS-CXX相关配置
date: 2023-01-16 16:01:42
tags:
- DBUS
categories: 
- 学习
- Linux
excerpt: DBUS-CXX在ubuntu18.04的环境配置，包括cmake,libsigc++的配置
---
# DBUS-CXX

---

# 介绍

DBUS-CXX是一个dbus协议基础上提供的C++ api封装。所以我们很多地方可以结合文档和例程直接看源码来进行学习，底层的很多都是linux的官方库。

# 参考资料

需先了解DBUS基础知识，建议看DBUS官方文档

DBUS-CXX也是建议看官方文档，各个配置环境也先**仔细看对应的文档**再操作

DBUS-CXX的中文资料较少，耐心看完官方文档

[dbus-cxx: dbus-cxx Library](https://dbus-cxx.github.io/index.html)

[[C++]:libsigc++ programming @ 玄根白丁的部落格 :: 痞客邦 :: (pixnet.net)](https://shangenpoden.pixnet.net/blog/post/249943675)

# 配置环境

## cmake

1. 检查自己是否有cmake并且版本是否大于3.12，若都无，从[该网站](https://cmake.org/download/)下载cmake源码

1. 解压cmake 源码，之后进入cmake目录执行 `sudo ./configure`,然后执行`sudo make && make install`

## libsigc++

> 从官方文档中可以看到，从压缩包build比git出来的项目更容易，git出来在make的时候会遇到错误
> 
1. `sudo apt-get install mm-common`下载mm-common
2. 下载源码 `wget https://download.gnome.org/sources/libsigc++/3.0/libsigc%2B%2B-3.0.0.tar.xz`  并解压缩，然后进入解压出来的目录
    
    > 这边选择3.0.0版本，3.2版本利用autotools编译会出现问题，暂时未找到解决方法
    > 
3. `mkdir __install`
4. 之后执行`./configure  --prefix=$PWD/__install`,如果是交叉编译，需要加上`--host=xxxx`
5. `make && make install` 
6. 可以看见在__install 下面生成了如下目录，对应着头文件，库文件

<!-- ![Untitled](DBUS-CXX%20ea3c1120e10e4ed2bb59ae12c9b37443/Untitled.png) -->

## dbus-cxx

1. 下载源码 `git clone https://github.com/dbus-cxx/dbus-cxx.git`
2. 添加环境变量，指定libsig++安装出来的pkgconfig的位置，该句可以加入你的shell配置文件中
`export PKG_CONFIG_PATH=/home/flork/git/libsigc++-3.0.0/__install/lib/pkgconfig:$PKG_CONFIG_PATH`
3. 按照这些步骤安装即可

<!-- ![Untitled](DBUS-CXX%20ea3c1120e10e4ed2bb59ae12c9b37443/Untitled%201.png) -->

编译需要添加指令`pkg-config --cflags --libs dbus-cxx-2.0`

<!-- ![Untitled](DBUS-CXX%20ea3c1120e10e4ed2bb59ae12c9b37443/Untitled%202.png) -->

# 例程

> 例程来自官方文档例程，详细介绍去文档[dbus-cxx: Quick start example 0: A simple server and client](https://dbus-cxx.github.io/quick_start_example_0.html)
> 

## server.cpp

```cpp
#include <dbus-cxx.h>
#include <unistd.h>

double add(double param1, double param2) { return param1 + param2; }

int main()
{
    std::shared_ptr<DBus::Dispatcher> dispatcher = DBus::StandaloneDispatcher::create();

    std::shared_ptr<DBus::Connection> conn = dispatcher->create_connection(DBus::BusType::SESSION);

    if (conn->request_name("dbuscxx.quickstart_0.server", DBUSCXX_NAME_FLAG_REPLACE_EXISTING) != DBus::RequestNameResponse::PrimaryOwner)
        return 1;

    // create an object on us
    std::shared_ptr<DBus::Object> object = conn->create_object("/dbuscxx/quickstart_0", DBus::ThreadForCalling::DispatcherThread);

    // add a method that can be called over the dbus
    object->create_method<double(double, double)>("dbuscxx.Quickstart", "add", sigc::ptr_fun(add));

    sleep(10);

    return 0;
}
```

## client.cpp

```cpp
#include <dbus-cxx.h>
#include <iostream>

int main()
{
    std::shared_ptr<DBus::Dispatcher> dispatcher;
    dispatcher = DBus::StandaloneDispatcher::create();

    std::shared_ptr<DBus::Connection> connection;
    connection = dispatcher->create_connection(DBus::BusType::SESSION);

    // create an object proxy, which stands in for a real object.
    // a proxy exists over the dbus
    std::shared_ptr<DBus::ObjectProxy> object;
    object = connection->create_object_proxy("dbuscxx.quickstart_0.server", "/dbuscxx/quickstart_0");

    // a method proxy acts like a real method, but will go over the dbus
    // to do its work.
    DBus::MethodProxy<double(double, double)> &add_proxy = *(object->create_method<double(double, double)>("dbuscxx.Quickstart", "add"));

    double answer;
    answer = add_proxy( 1.1, 2.2 );

    std::cout << "1.1 + 2.2 = " << answer << std::endl;

    return 0;
}
```

## Makefile

```cpp
.PHONY:clean

CXXFLAG			  = -std=c++17 -O3
PKGFLAG       = `pkg-config --cflags --libs dbus-cxx-2.0`

SERVER_SRC		= server.cpp
CLIENT_SRC		= client.cpp

SERVER_TARGET   = server
CLIENT_TARGET   = client

all:$(SERVER_TARGET) $(CLIENT_TARGET)

$(SERVER_TARGET):$(SERVER_SRC)
	g++ $(CXXFLAG)  $(SERVER_SRC) -o $(SERVER_TARGET) $(PKGFLAG) 

$(CLIENT_TARGET):$(CLIENT_SRC)
	g++ $(CXXFLAG) $(CLIENT_SRC) -o $(CLIENT_TARGET) $(PKGFLAG)

clean:
	@if [ -e "$(CLIENT_TARGET)" ] ; then \
		echo "rm $(CLIENT_TARGET)" ; \
		rm $(CLIENT_TARGET) ; \
	fi;

	@if [ -e "$(SERVER_TARGET)" ] ; then \
		echo "rm $(SERVER_TARGET)" ; \
		rm $(SERVER_TARGET) ; \
	fi;
```

## 运行结果

先运行 server ，再运行 client

<!-- ![Untitled](DBUS-CXX%20ea3c1120e10e4ed2bb59ae12c9b37443/Untitled%203.png) -->