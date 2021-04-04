---
title: Linux相关知识
date: 2018-09-26 19:09:01
tags: 服务器
categories: 服务器
---

### 从纯净系统启动为止

#### 挂在iso镜像

1. 找到光盘的完整路径名，在命令行输入： ls -l /dev | grep cdrom
2. 可以看到光盘的名字叫做： cdrom
3.  mount /dev/cdrom /mnt 将镜像挂在到/mnt目录下
4. 进入到Packages目录，在命令行输入：cd Packages。然后输入：ls -l | grep mysql。找到我们要拷贝出来的rpm包
5. 卸载光盘 umount /mnt
6. 安装rpm插件， rpm -ivh 软件包名

#### 开关机和重启

1. 关机命令： 

   - halt 立刻关机 
   - power off 立刻关机
   - shutdown -h now 立刻关机(root使用)
   - Shutdown -h 10 过十分钟关机。可以使用shutdown -c命令取消重启

2. 重启命令

   - reboot

   - shutdown -r now 立即重启(root使用)

   - shutdown -r 10.   shutdown -c可以取消


### 日志相关

#### less命令

**启动时携带的参数**

-b <缓冲区大小> 设置缓冲区的大小，注意这里不是每页显示的行数
-i 忽略搜索时的大小写
-m 显示类似more命令的百分比
-N 显示每行的行号
-o <文件名> 将less 输出的内容在指定文件中保存起来
-s 显示连续空行为一行
-S 行过长时间将超出部分舍弃



**启动后可以使用的参数**

/字符串：向下搜索"字符串"的功能
?字符串：向上搜索"字符串"的功能

n：重复前一个搜索（与 / 或 ? 有关）
N：反向重复前一个搜索（与 / 或 ? 有关）

b：向前翻动一页
空格：向后翻动一页

上箭头：向前翻动一行
下箭头：向下翻动一行

g：到文件开头
G：到文件结尾



```sql
\[resetPassword\] account : 
```



[]需要转译，所以加斜杠，否则查不出来。

q：退出less命令

### 常用命令

lsof -i:端口号

kil -9 进程号

#### 查看当前服务器外网ip命令

curl https://ip.cn