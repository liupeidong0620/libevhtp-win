# windows 编译evhtp 说明

# 目录说明

* ev-thr （vs 测试例子）
* libevhtp-1.tar.gz（libevhtp修改后的源码包，可以直接查看修改内容）

# 依赖库

evhtp依赖库
* openssl（主要为了https）
* libevent
* pthread （多线程模式）

# 编译工具

* vs 2017
* perl
* cmake

```sh
# 判断系统是否安装cmake
where cmake
D:\vs2008\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin\cmake.exe

# 判断系统是否安装 perl
where perl
D:\Strawberry\perl\bin\perl.exe

```

## 编译opensll

### 工具

* x86 Native Tools Command Prompt for VS 2017(vs2017 命令行 编译32位库)
* perl 工具
* cmake
* nmake (安装 vs 2017 c++ 模块会自带)


### 下载源码

```sh
# 这里什么方式下载都行
# 下载完解压
curl -O 'https://www.openssl.org/source/old/1.0.1/openssl-1.0.1u.tar.gz'
tar -xzvf openssl-1.0.1u.tar.gz
cd openssl-1.0.1u
```

### 编译

```sh
# 在vs 命令行中进行操作
$ perl Configure VC-WIN32 --prefix="D:\vs2008\code\openssl\openssl" --openssldir="D:\vs2008\code\openssl\ssl"
$ ms\do_ms
$ nmake -f ms\nt.mak 
$ nmake -f ms\nt.mak test
# 安装库和openssl工具
$ nmake -f ms\nt.mak install
```

## 编译libevent

### 下载源码

```sh
# 下载源码
libevent-2.0.22-stable.tar.gz
# 解压源码
tar -xzvf libevent-2.0.22-stable.tar.gz
```

### 修改配置文件

* 需要修改Makefile.nmake 修改内容当作参考
* 主要为了生成 libevent_openssl.lib， libevent_pthreads.lib 库

```sh
# 查看文件
$ cat Makefile.nmake


# WATCH OUT!  This makefile is a work in progress.  It is probably missing
# tons of important things.  DO NOT RELY ON IT TO BUILD A GOOD LIBEVENT.

# Needed for correctness

OPENSSL_DIR="D:\vs2008\code\openssl\openssl"

!IFDEF OPENSSL_DIR
SSL_CFLAGS=/I$(OPENSSL_DIR)\include /DEVENT__HAVE_OPENSSL
!ELSE
SSL_CFLAGS=
!ENDIF

# CFLAGS=/IWIN32-Code /Iinclude /Icompat /DWIN32 /DHAVE_CONFIG_H /I.
CFLAGS=/IWIN32-Code /Iinclude /Icompat /DWIN32 /DHAVE_CONFIG_H /I. $(SSL_CFLAGS)

# For optimization and warnings
CFLAGS=$(CFLAGS) /Ox /W3 /wd4996 /nologo

# XXXX have a debug mode

LIBFLAGS=/nologo

CORE_OBJS=event.obj buffer.obj bufferevent.obj bufferevent_sock.obj \
        bufferevent_pair.obj listener.obj evmap.obj log.obj evutil.obj \
        strlcpy.obj signal.obj bufferevent_filter.obj evthread.obj \
        bufferevent_ratelim.obj evutil_rand.obj
WIN_OBJS=win32select.obj evthread_win32.obj buffer_iocp.obj \
        event_iocp.obj bufferevent_async.obj
EXTRA_OBJS=event_tagging.obj http.obj evdns.obj evrpc.obj

# 添加输出文件*_openssl.lib *_openssl.obj
!IFDEF OPENSSL_DIR
SSL_OBJS=bufferevent_openssl.obj
SSL_LIBS=libevent_openssl.lib
!ELSE
SSL_OBJS=
SSL_LIBS=
!ENDIF

# 添加pthreads输出文件
THR_OBJS=evthread.obj
THR_LIBS=libevent_pthreads.lib

ALL_OBJS=$(CORE_OBJS) $(WIN_OBJS) $(EXTRA_OBJS) $(SSL_OBJS) $(THR_OBJS)
STATIC_LIBS=libevent_core.lib libevent_extras.lib libevent.lib $(SSL_LIBS) $(THR_LIBS)


all: static_libs tests

static_libs: $(STATIC_LIBS)

libevent_core.lib: $(CORE_OBJS) $(WIN_OBJS)
        lib $(LIBFLAGS) $(CORE_OBJS) $(WIN_OBJS) /out:libevent_core.lib

libevent_extras.lib: $(EXTRA_OBJS)
        lib $(LIBFLAGS) $(EXTRA_OBJS) /out:libevent_extras.lib

libevent.lib: $(CORE_OBJS) $(WIN_OBJS) $(EXTRA_OBJS)
        lib $(LIBFLAGS) $(CORE_OBJS) $(EXTRA_OBJS) $(WIN_OBJS) /out:libevent.lib

# how to make "libevent_openssl"
libevent_openssl.lib: $(SSL_OBJS)
        lib $(LIBFLAGS) $(SSL_OBJS) /out:libevent_openssl.lib

# how to make "libevent_pthreads"
libevent_pthreads.lib: $(THR_OBJS)
        lib $(LIBFLAGS) $(THR_OBJS) /out:libevent_pthreads.lib

clean:
        del $(ALL_OBJS)
        del $(STATIC_LIBS)
        cd test
        $(MAKE) /F Makefile.nmake clean
tests:
        cd test
        $(MAKE) /F Makefile.nmake

```

### 编译

```sh
$ cd <libevent source dir>
$ nmake -f makefile.nmake
```

### 复制文件

```sh
# 头文件(复制所有)
include/
WIN32-Code/
# lib库
$ ls libevent
libevent.lib    libevent_core.lib        libevent_openssl.lib     libevent_pthreads.lib          libevent_extras.lib  

```

## 下载pthread库

### 下载

``` sh
# 下载地址
ftp://sourceware.org/pub/pthreads-win32/pthreads-w32-2-8-0-release.exe

# 下载运行exe安装
```

## 编译evhtp

### 下载

```sh
git clone https://github.com/criticalstack/libevhtp.git

# 目前使用的分支
git checkout 1.2.13

cd libevhtp/build # 目录中
```

### 修改部分源码

* 为了编译源码，源码有一些改动。

### 编译工具

* x86 Native Tools Command Prompt for VS 2017(vs2017 命令行 编译32位库)
* cmake
* vs 2017

### 编译

```sh
# 第一步
# 进入 vs 命令行（x86 编译32位库）
# 使用cmake 生成vs 工程
# 这个地方 主要是 开启几个宏定义，代码中开启相应功能
# 只是一个参照，需要根据不同版本 进行调整

cmake -DLIBEVENT_INCLUDE_DIR:PATH="D:/vs2008/code/libevent-2.0.22-stable/include" -DLIBEVENT_CORE_LIBRARY:FILEPATH="D:/vs2008/code/libevent-2.0.22-stable/libevent_core.lib" -DLIBEVENT_LIBRARY:FILEPATH="D:/vs2008/code/libevent-2.0.22-stable/libevent.lib" -DLIBEVENT_EXTRA_LIBRARY:FILEPATH="D:/vs2008/code/libevent-2.0.22-stable/libevent_extras.lib" -DLIBEVENT_PTHREADS_LIBRARY:FILEPATH="D:/vs2008/code/libevent-2.0.22-stable/libevent_pthreads.lib" -DHAS_SYS_ONIG:FILEPATH="HAS_SYS_ONIG-NOTFOUND" -DLIBEVENT_include_DIR="D:/vs2008/code/libevent-2.0.22-stable/include" -DEVHTP_DISABLE_EVTHR:BOOL="0" -DEVHTP_DISABLE_REGEX:BOOL="1"  -DLIBEVENT_OPENSSL_LIBRARY:FILEPATH="D:/vs2008/code/libevent-2.0.22-stable/libevent_openssl.lib" -DLIB_EAY:FILEPATH="D:/vs2008/code/openssl/openssl/lib/libeay32.lib" -DSSL_EAY:FILEPATH="D:/vs2008/code/openssl/openssl/lib/ssleay32.lib" -DOPENSSL_INCLUDE_DIR:PATH="D:/vs2008/code/openssl/openssl/include" ..

```

### cmake生成参数

```sh

-- EVHTP_VERSION            :  1.2.13
-- EVHTP_DISABLE_SSL        :  OFF   # 开启ssl
-- EVHTP_DISABLE_EVTHR      :  0     # 开启pthread
-- EVHTP_DISABLE_REGEX      :  1     # 关闭regex
-- EVHTP_BUILD_SHARED       :  OFF
-- EVHTP_USE_JEMALLOC       :  OFF
-- EVHTP_USE_TCMALLOC       :  OFF

-- CMAKE_BUILD_TYPE         : Release
-- CMAKE_INSTALL_PREFIX     : C:/Program Files (x86)/libevhtp
-- CMAKE_BINARY_DIR         : D:/vs2008/code/libevhtp-1/build
-- CMAKE_CURRENT_BINARY_DIR : D:/vs2008/code/libevhtp-1/build
-- CMAKE_CURRENT_SOURCE_DIR : D:/vs2008/code/libevhtp-1
-- PROJECT_BINARY_DIR       : D:/vs2008/code/libevhtp-1/build
-- PROJECT_SOURCE_DIR       : D:/vs2008/code/libevhtp-1
-- CMAKE_MODULE_PATH        : D:/vs2008/code/libevhtp-1/cmake
-- CMAKE_SYSTEM_NAME        : Windows
-- CMAKE_SYSTEM_VERSION     : 10.0.18362
-- CMAKE_C_COMPILER         : D:/vs2008/VC/Tools/MSVC/14.16.27023/bin/Hostx86/x86/cl.exe
-- CMAKE_AR                 :
-- CMAKE_RANLIB             :
-- CFLAGS                   : -DNO_STRNDUP -DNO_SYS_UN -DEVHTP_SYS_ARCH=32 -DWIN32 /DWIN32 /D_WINDOWS /W3 -DPROJECT_VERSION=1.2.13 -Wall
                              /MD /O2 /Ob2 /DNDEBUG

-- Configuring done
-- Generating done

```

### vs 编译

```sh
# 第二步
# cmake 之后 vs就可以打开工程

# 在vs 中添加
# pthread include
# opensll include

# 点击生成，可以生成对应的库

# 复制生成的头文件(所有)

build/include
build/compat
include

# lib库位置

build/Debug/

```

### 编译中遇到的问题

```sh
# cmake 修改
cmake 查看系统中是否有openssl库，查找不到cmake不过。需要注释一些配置，使其可以通过

# 会报strndup 奇怪错误
需要把定义放在调用前面

# 开启pthread时，各种奇怪错误
需要包含<sys/queue.h> 头文件

# 具体细节看，evhtp中的改动
```

# 例子

```c++
#include <stdio.h>

#include <WinSock2.h>      // windows socket

#include <iostream>
#include <string.h>
#include "evhtp.h"


static void DefaultHandler(evhtp_request_t* req, void* arg)
{
	evhtp_send_reply(req, EVHTP_RES_NOTFOUND);
}

static void helloHandler(evhtp_request_t* req, void* arg)
{
	//1. deal req
	//2. doReq
	//3. return
	//evhtp_headers_add_header(req->headers_out, evhtp_header_new("Connection", "keep-alive", 0, 0));
	std::cout << "request: " << req->uri->path->full << std::endl;
	evhtp_headers_add_header(req->headers_out, NULL);
	std::string str = "hello world\n";
	evbuffer_add(req->buffer_out, str.c_str(), strlen(str.c_str()));
	evhtp_send_reply(req, EVHTP_RES_OK);
}

int main()
{
	WSADATA wsaData;
	WSAStartup(MAKEWORD(1, 1), &wsaData);

	std::cout << "ev-thr start ..." << std::endl;
	evbase_t *base = event_base_new();
	evhtp_t *htp = evhtp_new(base, NULL);

	/* ssl support */
	evhtp_ssl_cfg_t ssl_cfg;
	memset(&ssl_cfg, 0, sizeof(evhtp_ssl_cfg_t));
	evhtp_set_gencb(htp, DefaultHandler, NULL);
	/* 设置回调函数 */
	evhtp_set_cb(htp, "/hello", helloHandler, NULL);

	/* ssl support */
	char pemfile[] = "D:/quic/QUIC/QUIC/certs/y.play.360kan.com.crt";
	char privfile[] = "D:/quic/QUIC/QUIC/certs/y.play.360kan.com.key";

	ssl_cfg.pemfile = pemfile;//"F:/ThirdParty/openssl-OpenSSL_1_0_1u/certs/demo/ca-cert.pem";    // "/etc/wpdconfig/server.crt";
	ssl_cfg.privfile = privfile;//"F:/ThirdParty/openssl-OpenSSL_1_0_1u/certs/demo/ca-cert.pem"; // "/etc/wpdconfig/server.key";
	ssl_cfg.cafile = NULL;        // "/etc/wpdconfig/ca.crt";
	ssl_cfg.capath = NULL;        // "/etc/wpdconfig/";
	ssl_cfg.ciphers = NULL;        // ciphers
	ssl_cfg.ssl_opts = SSL_OP_NO_SSLv2;
	ssl_cfg.ssl_ctx_timeout = 60 * 60 * 48;
	ssl_cfg.verify_peer = SSL_VERIFY_NONE;
	ssl_cfg.verify_depth = 42;
	ssl_cfg.x509_verify_cb = NULL;
	ssl_cfg.x509_chk_issued_cb = NULL;
	ssl_cfg.scache_type = evhtp_ssl_scache_type_internal;
	ssl_cfg.scache_size = 1024;
	ssl_cfg.scache_timeout = 1024;
	ssl_cfg.scache_init = NULL;
	ssl_cfg.scache_add = NULL;
	ssl_cfg.scache_get = NULL;
	ssl_cfg.scache_del = NULL;

	if (evhtp_ssl_init(htp, &ssl_cfg) == -1)
	{
		std::cout << "evhtp_ssl_init failed" << std::endl;
		return 0;
	}

	//int threadNum = 10;
	evhtp_use_threads(htp, NULL, 5, NULL);
	std::cout << "ev-thr init ..." << std::endl;
	std::cout << "ev-thr listen addr: 0.0.0.0:18090" << std::endl;
	evhtp_bind_socket(htp, "0.0.0.0", 18090, 1024);
	event_base_loop(base, 0);

	std::cout << "ev-thr stop." << std::endl;
	WSACleanup();
	return 0;
}
```

### 依赖

* openssl
* libevenet
* pthread
* libevhtp

### 验证

```sh

# 简单展示，工程目录肯定有差别
# 添加头文件
# 简单展示
../inc;../openssl/include;../pthread/Pre-built.2/include;
# 添加静态库
# 简单展示
../pthread/Pre-built.2/lib/pthreadVC2.lib;../openssl/lib/libeay32.lib;../lib/libevent_openssl.lib;../openssl/lib/ssleay32.lib;../lib/evhtp.lib;../lib/libevent.lib;../lib/libevent_pthreads.lib;

# 使用vs编译
# 编译完成后，需要设置动态库位置

# pthread 动态库
# 动态库放入这个位置 运行程序
C:\Windows\SysWOW64

# 验证
# curl -k https://xxx.xxx.xxx/hello

# cmd命令行中查看线程数
>pslist -dmx 5556

PsList v1.4 - Process information lister
Copyright (C) 2000-2016 Mark Russinovich
Sysinternals - www.sysinternals.com

Process and thread information for LIUPEIDONG-L1:

Name                Pid      VM      WS    Priv Priv Pk   Faults   NonP Page
ev-thr             5556   60756    1264    2564    3264    19738     33  121
 Tid Pri    Cswtch            State     User Time   Kernel Time   Elapsed Time
14144  10        48     Wait:UserReq  0:00:00.140   0:00:00.062    1:11:32.912
12148   8         2     Wait:UserReq  0:00:00.000   0:00:00.000    1:11:32.691
15404  10         3     Wait:UserReq  0:00:00.000   0:00:00.000    1:11:32.685
9752   8         2     Wait:UserReq  0:00:00.000   0:00:00.000    1:11:32.679
13172  10         3     Wait:UserReq  0:00:00.000   0:00:00.000    1:11:32.673
 256   8         2     Wait:UserReq  0:00:00.000   0:00:00.000    1:11:32.667

```