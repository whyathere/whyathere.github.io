---
title: Redis
date: 2018-09-10 18:27:21
tags: Redis
categories: Redis
---

## Redis

### 大概

redis是单线程的

io多路复用，所以不需要线程池

### 问题

#### 如何保证缓存与数据库双写时数据一致性？

只要用缓存，就可能涉及到缓存与数据库双存储双写，只要是双写，就一定有数据一致性的问题。

- 先更新数据库，再更新缓存
- 先删除缓存，再更新数据库
- 先更新数据库，再删除缓存

##### 先更新数据库，再更新缓存**

**原因一：线程安全角度**

同时有请求A和请求B进行更新操作，那么会出现
（1）线程A更新了数据库
（2）线程B更新了数据库
（3）线程B更新了缓存
（4）线程A更新了缓存
这就出现请求A更新缓存应该比请求B更新缓存早才对，但是因为网络等原因，B却比A更早更新了缓存。这就导致了脏数据，因此不考虑。
**原因二（业务场景角度）**
有如下两点：
（1）如果你是一个写数据库场景比较多，而读数据场景比较少的业务需求，采用这种方案就会导致，数据压根还没读到，缓存就被频繁的更新，浪费性能。
（2）如果你写入数据库的值，并不是直接写入缓存的，而是要经过一系列复杂的计算再写入缓存。那么，每次写入数据库后，都再次计算写入缓存的值，无疑是浪费性能的。显然，删除缓存更为适合。

##### 先删缓存，再更新数据库

A更新操作，B查询操作

### 本地安装启动redis

```shell
sudo make test
```

在执行redis 编译测试的时候提示

```shel
xcrun: error: invalid active developer path (/Library/Developer/CommandLineTools), missing xcrun at: /Library/Developer/CommandLineTools/usr/bin/xcrun
```

解决办法是打开 Terminal，执行如下命令：

```shell
xcode-select --install
```

这将下载并安装 xcode 开发者工具，之后就会修复这个问题。这个问题是因为需要你明确同意许可协议。如果你有多个版本的 Xcode 或者只需要命令行工具而非 Xcode，你可能需要重置 Xcode 的路径：

```shell
# 重置 Xcode 路径
xcode-select --switch /Applications/Xcode.app
# 使用命令行工具
xcode-select --switch /Library/Developer/CommandLineTool
```

### 使用

cd /usr/local/bin 	./redis-server		启动redis服务端

redis-cli			  启动客户端

get  'key'			通过key来取值

redis-cli shutdown	  关闭redis客户端，包括server