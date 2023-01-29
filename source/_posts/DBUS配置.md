---
title: DBUS配置
date: 2023-01-23 13:29:59
tags:
- DBUS
categories: 
- 学习
- Linux
excerpt: DBUS协议相关配置和介绍
---

# DBUS 配置

## 资料

建议看这几个官方文档

[dbus (www.freedesktop.org)](https://www.freedesktop.org/wiki/Software/dbus/)

[DBus Overview (pythonhosted.org)](https://pythonhosted.org/txdbus/dbus_overview.html)

[D-Bus Tutorial (dbus.freedesktop.org)](https://dbus.freedesktop.org/doc/dbus-tutorial.html)

[D-Bus Specification (dbus.freedesktop.org)](https://dbus.freedesktop.org/doc/dbus-specification.html)

[DBus介绍 - 北落不吉 - 博客园 (cnblogs.com)](https://www.cnblogs.com/hzl6255/p/4096260.html)

[Dbus组成和原理 - konglingbin - 博客园 (cnblogs.com)](https://www.cnblogs.com/klb561/p/9058282.html)

[DBUS基础知识 - 简书 (jianshu.com)](https://www.jianshu.com/p/c073daaf427f)

[DBUS基础知识（非常全面）_土戈的博客-CSDN博客_dbus](https://blog.csdn.net/f110300641/article/details/106823611)

[D-Bus实例介绍 | Just For Coding (just4coding.com)](http://just4coding.com/2018/07/31/dbus/)

# 介绍

`D-Bus`是`Linux`及其他类`UNIX`系统上的一种`IPC(Interprocess communication`)机制。相较于传统的`管道(PIPE)`、`Socket`等原生基于字节流的IPC方式，`D-Bus`提供了基于独立`Message`的传输方式，应用程序使用起来更加简单。`D-Bus`的设计初衷是为`Linux`桌面环境上的一系列应用程序提供通信方式，它的设计里保留了许多对上层框架设计考虑的元素。

`D-Bus`的常用架构与传统的`Socket`一对一通信模式不同，它基于中间消息路由转发的模式来实现, 如下图:

![DBUS消息转发图](https://raw.githubusercontent.com/FreddieGeorge/blogImg/main/img/DBUS%E6%B6%88%E6%81%AF%E8%BD%AC%E5%8F%91%E5%9B%BE.png)

`D-Bus`默认提供两种BUS，`系统BUS(system)`和`会话BUS(session)`。系统BUS在每台机器上是惟一的，用于后台服务及操作系统之间的通信。会话BUS用于每个登录用户会话的应用程序之间的通信。每个BUS实例由一个`bus-daemon`进程来管理，由其负责消息路由转发。应用程序需要收发消息，需要连接到BUS实例上。BUS实例使用基于XML的配置文件来控制安全策略，如用户能否注册服务，能给哪些服务接口发送消息等等。

### 建立服务流程

- 建立 dbus 连接 `dbus_bus_get()`
- 为该连接起名 `dbus_bus_request_name()` ，该名字将作为在后续进行远程调用的时候的服务名
- 进入监听循环 `dbus_connection_read_write()`
- 从总线上取出消息 `dbus_connection_pop_message()`
- 对比消息中的方法接口名和方法名 `dbus_message_is_method_call()`
- 如果一致，则跳转到对应的处理。在处理中，我们会从消息中取出远程调用的参数，并且建立回传结果的通路，`reply_to_method_call()`
    
    > 回传动作本身等同于一次不需要等待结果的远程调用。
    > 

### 发送信号流程

- 在建立好 dbus 连接和起名后，可以建立一个发送信号的通道，`dbus_message_new_signal()` ，在函数中填入该信号的接口名和信号名
- 把信号对应的相关参数压进去，`dbus_message_iter_init_append()`   ，`dbus_message_iter_append_basic()`
- 启动发送，`dbus_connection_send()` ，`dbus_connection_flush()`

### 进行远程调用流程

- 在建立好 dbus 连接和起名后，可以申请一个远程调用通道 `dbus_message_new_method_call()`
- 压入本次调用的参数 `dbus_message_iter_init_append()`，`dbus_message_iter_append_basic()`
    
    > 实际上我们是申请了一个首地址，把真正要传的参数，往这个首地址里面送(送完之后一般都会判断是否内存越界了)
    > 
- 启动发送调用并释放发送相关的消息结构， `dbus_connection_send_with_reply()`。这个启动函数中带有一个句柄。我们马上会阻塞等待这个句柄给我们带回总线上回传的消息。
- 当这个句柄回传消息之后，我们从消息结构中分离出参数。用dbus提供的函数提取参数的类型和参数 ， `dbus_message_iter_init()`; `dbus_message_iter_next()`; `dbus_message_iter_get_arg_type()`; `dbus_message_iter_get_basic()`。

### 信号接收流程

- 在建立好 dbus 连接和起名后，为我们将要进行的消息循环添加匹配条件，`dbus_bus_add_match()`
- 我们进入等待循环后，只需要对信号名，信号接口名进行判断就可以分别处理各种信号了。在各个处理分支上。我们可以分离出消息中的参数。对参数类型进行判断和其他的处理。

# 环境搭建

- `sudo apt-get install libdbus-glib-1-dev`
- `sudo apt-get install libgtk2.0-dev`

### 常见错误

1. 在程序中 `#include <dbus/dbus.h>` 的时候会报错: `dbus/dbus.h ： No such file or directory` ，这其实是是 dbus-1.0 的安装位置有问题，在编译时添加 `pkg-config --libs --cflags dbus-1`
2.  `dbus/dbus-arch-deps.h ： No such file or directory`,在编译时添加 `pkg-config --libs --cflags dbus-1` 对于1应该也可以通过添加此编译条件解决, 详 [Makefile](https://www.notion.so/Makefile-8ea3d347262043d6852db7233d0d0aa5) 

# 例程

### message.h

```c
const char *const SERVER_BUS_NAME = "com.flork.dbus";
const char *const OBJECT_PATH_NAME_YES = "/com/flork/dbus/yes";
const char *const OBJECT_PATH_NAME_NO = "/com/flork/dbus/no";
const char *const INTERFACE_NAME = "com.flork.dbus_demo";
const char *const METHOD_NAME = "hello";
```

### server.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>
#include <unistd.h>
#include <ctype.h>

#include <dbus/dbus.h>

#include "message.h"

DBusError dbus_error;

void print_dbus_error(char *str);

int main(int argc, char **argv)
{
    
    DBusConnection *conn;
    int ret;

    // 初始化错误信息
    dbus_error_init(&dbus_error);

    // 建立连接
    conn = dbus_bus_get(DBUS_BUS_SYSTEM, &dbus_error);

    if(dbus_error_is_set(&dbus_error))
    {
        print_dbus_error("dbus_bus_get");
    }

    if(!conn)
    {
        exit(1);
    }

    // 为连接起名
    
    ret = dbus_bus_request_name(conn, SERVER_BUS_NAME, 
																DBUS_NAME_FLAG_DO_NOT_QUEUE, &dbus_error);

    if (dbus_error_is_set(&dbus_error))
    {
        print_dbus_error("dbus_bus_request_name");
    }

    if (ret != DBUS_REQUEST_NAME_REPLY_PRIMARY_OWNER)
    {
        fprintf(stderr, "not primary owner, ret = %d\n", ret);
    }

    while (1)
    {
        if(!dbus_connection_read_write_dispatch(conn,-1))
        {
            fprintf(stderr, "Not connected now.\v");
            exit(1);
        }

        DBusMessage *msg;

        // 从总线上取出消息
        if((msg = dbus_connection_pop_message(conn)) == NULL)
        {
            fprintf(stderr, "Did not get message\n");
            continue;
        }

        // 对比消息中的方法接口名和方法名
        if(dbus_message_is_method_call(msg,INTERFACE_NAME,METHOD_NAME))
        {
            char *s;

            // 获取参数
            if(dbus_message_get_args(msg,&dbus_error,DBUS_TYPE_STRING,&s,DBUS_TYPE_INVALID))
            {
                printf("Received: %s\n", s);

                // 回复
                DBusMessage *reply;
                char answer[1024];

                assert(reply = dbus_message_new_method_return(msg));

                DBusMessageIter iter;
                // 把信号对应的相关参数压进去
                dbus_message_iter_init_append(reply, &iter);

                if (dbus_message_has_path(msg, OBJECT_PATH_NAME_YES))
                {
                    sprintf(answer, "Yes, %s", s);
                }
                else if (dbus_message_has_path(msg, OBJECT_PATH_NAME_NO))
                {
                    sprintf(answer, "No, %s", s);
                }
                else
                {
                    sprintf(answer, "No object found");
                }

                char *ptr = answer;
                assert(dbus_message_iter_append_basic(&iter, DBUS_TYPE_STRING, &ptr));

                assert(dbus_connection_send(conn, reply, NULL));
                dbus_connection_flush(conn);
                dbus_message_unref(reply);
            }
            else
            {
                print_dbus_error("Error getting message");
            }
        }
    }

    return 0;
}

void print_dbus_error(char *str)
{
    fprintf(stderr, "%s: %s\n", str, dbus_error.message);
    dbus_error_free(&dbus_error);
}
```

### client.c

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>
#include <unistd.h>
#include <ctype.h>

#include <dbus/dbus.h>

#include "message.h"

DBusError dbus_error;

void print_dbus_error(char *str);

int main(int argc, char **argv)
{
    DBusConnection *conn;
    char input[80];

    dbus_error_init(&dbus_error);

    assert(conn = dbus_bus_get(DBUS_BUS_SYSTEM, &dbus_error));

    if (dbus_error_is_set(&dbus_error))
    {
        print_dbus_error("dbus_bus_get");
    }

    const char *objects[] = {OBJECT_PATH_NAME_YES, OBJECT_PATH_NAME_NO, NULL};
    int i = 0;

    for (; objects[i]; i++)
    {
        DBusMessage *request;
        assert((request = dbus_message_new_method_call(SERVER_BUS_NAME, objects[i], INTERFACE_NAME, METHOD_NAME)));

        DBusMessageIter iter;
        dbus_message_iter_init_append(request, &iter);
        snprintf(input, sizeof(input), "alice");

        char *ptr = input;
        assert(dbus_message_iter_append_basic(&iter, DBUS_TYPE_STRING, &ptr));

        DBusPendingCall *pending_return;
        assert(dbus_connection_send_with_reply(conn, request, &pending_return, -1));

        assert(pending_return);
        dbus_connection_flush(conn);
        dbus_message_unref(request);

        dbus_pending_call_block(pending_return);

        DBusMessage *reply;
        assert((reply = dbus_pending_call_steal_reply(pending_return)));
        dbus_pending_call_unref(pending_return);

        char *s;
        if (dbus_message_get_args(reply, &dbus_error, DBUS_TYPE_STRING, &s, DBUS_TYPE_INVALID))
        {
            printf("Reply: %s\n", s);
        }
        else
        {
            fprintf(stderr, "Did not get arguments in reply\n");
            exit(1);
        }

        dbus_message_unref(reply);
    }

    return 0;
}

void print_dbus_error(char *str)
{
    fprintf(stderr, "%s: %s\n", str, dbus_error.message);
    dbus_error_free(&dbus_error);
}
```

### Makefile

```makefile
.PHONY:clean
CFLAG			     = `pkg-config --libs --cflags dbus-1`

SERVER_TARGET	 = server
CLIENT_TARGET	 = client

server:
	gcc server.c -o $(SERVER_TARGET) $(CFLAG)

client:
	gcc client.c -o $(CLIENT_TARGET) $(CFLAG)

clean:
	rm $(SERVER_TARGET) $(CLIENT_TARGET)
```

### 编译

使用 `make server` 或者 `make client` 生产可执行文件即可

直接运行 server 会发现程序报错，Ubunut的安全策略不允许注册服务

在 `/etc/dbus-1/system.d/`下创建安全策略文件`com.flork.dbus.conf`，内容如下:

```html
<busconfig>

  <policy user="flork">
    <allow own="com.flork.dbus"/>
  </policy>

  <policy context="default">
    <allow send_interface="com.flork.dbus_demo"/>
  </policy>

</busconfig>
```

再次运行`./server &`，使用 `busctl` 可以查看 server 已经创建成功

![busctl结果](https://raw.githubusercontent.com/FreddieGeorge/blogImg/main/img/busctl%E7%BB%93%E6%9E%9C.png)

之后运行 `./client` 即可

![DBUS运行结果](https://raw.githubusercontent.com/FreddieGeorge/blogImg/main/img/DBUS%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C.png)

### 遇到的错误

仔细检查创建的安全策略文件是否正确！
