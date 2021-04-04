---
title: Mac使用中遇到的问题
date: 2018-09-10 18:27:21
tags: About Mac
categories: Mac

---

## 

## 软件安装

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

