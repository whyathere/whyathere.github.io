---
title: FastDFS 分布式文件系统
date: 2018-05-31 10:40:45
tags: Spring
categories: 框架
---

### FastDfs简介

1. FastDfs是一个轻量级的开源分布式文件系统
2. FastDfs主要解决了大容量的文件存储和高并发访问的问题，文件存取时实现了负载均衡
3. FastDfs实现了软件方式的RAID，可以使用廉价的IDE硬盘进行存储
4. 支持存储服务器在线扩容
5. 支持相同内容的文件只保存一份，节约磁盘空间
6. FastDFS只能通过Client API访问，不支持POSIX访问方式
7. FastDFS特别适合大中型网站使用，用来存储资源文件(如：图片、文档、音频、视频等等)

     FastDFS是一个开源的轻量级分布式文件系统，她对文件进行管理，功能包括：文件存储、文件同步、文件访问（文件上传、文件下载）等，解决了大容量存储和负载均衡的问题。特别适合以文件为载体的在线服务，如相册网站、视频网站等等。  

FastDFS服务端有两个角色：**跟踪器(tracker)**和**存储节点(storage)**。跟踪器主要做调度工作，在访问上起负载均衡 的作用。

#### Tracker

![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/FastDFS/%E5%BC%95%E5%85%A5Tracker.jpg?raw=true)

Group之间是相互独立的，Group内是相互备份的；Tracker之间也是相互独立的。

- Group之间 相互独立

- 同一Group内的Storage Server 之间需要互相备份

  文件存放到一个Storage以后，需要备份到别的服务器

- Tracker之间是不交互的

  - 每个storage server都需要向所有Tracker去主动报告信息

**要点：**

1. group内的server内容都是一致的
2. 一个负责跟踪一个负责存储
3. group会向每个tracker都汇报，tracker存储的信息很少



![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/FastDFS/%E4%B8%8A%E4%BC%A0%E6%96%87%E4%BB%B6%E6%97%B6%E5%BA%8F%E5%9B%BE.jpg?raw=true)

#### 选定Tracker Server

- Tracker Server不止一个，客户端选择哪一个做上传文件？

Client是如何知道上传到那个Tracker Server

Client可以维护一个Tracker列表

- Tracker如何选择Group？(三种策略)
  - round robin(轮询)
  - load balance(选择最大剩余空间的组上传文件)
  - specify group(指定group上传)

#### 选定Storage Server

- 一个组内有多个Storage Server ，选择哪一个？
  - 1. Round robin，所有server轮询使用(默认)
  - 2. 根据IP地址进行排序选择第一个服务器(IP地址最小者)
  - 3. 根据优先级进行排序(上传优先级由storage server来设置，参数为upload_priority)

![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/FastDFS/storage%20Server%20%E7%9A%84%E5%AD%98%E5%82%A8%E7%BB%93%E6%9E%84.jpg?raw=true)

#### 选择storage path

- 如何选择storage path(虚拟磁盘目录M00，M01路径)
  - 1. round robin ，轮询(默认)
  - 2. load balance，选择使用剩余空间最大的存储路径



目录和文件

- 选定存放目录？
  - storage会生成一个file_id(wKjGgVgbV2-ABdo-AAAAHw.jpg)，采用Base64编码，file_id包含字段包括：storage server_ip(文件的源服务器)、文件创建时间、文件大小、文件CRC32校验码和随机数、；(这个时候文件名已经和最早的文件名是两回事儿了)
  - 每个存储目录下有两个256*256个子目录，storage 会按文件file_id进行两次hash，路由到其中一个子目录，然后将文件以file_id为文件名存储到该子目录下

Group1/M00/00/OC/wKjGgVgbV2-ABdo-AAAAHw.jpg   

tracker可以通过上面的东西迅速的找到文件



**要点：**

- server之间不分主从，每个目录可以不止放一个文件
- 存储服务器启动的时候就会把两个256*256个目录一次创建出来
- file_id必须由client来保存。

怎么确定存放在那个目录

storage会按照文件的file_id(wKjGgVgbV2-ABdo-AAAAHw.jpg   )做两次hash算法，路由到其中一个子目录中。

#### Storage Server之间的文件同步

- 同一组内的storage server之间是对等的，文件上传、删除等操作可以在任意一台storage server上进行；
- 文件同步只在组内的storage server之间进行，采用push方式，即源服务器同步给目标服务器；
- 源头数据才需要同步，备份数据不需要再次同步，否则就构成环路了；
- 上述第二条规则有个例外，就是新增加一台storage server时，由已有的一套storage server将已有的所有数据(包括源头数据和备份数据)同步给新增服务器

client是FastDFS提供的客户端

FastDFS有覆盖方法，可以修改某个已经上传的文件，甚至是追加操作。



例子：

A，B，C三个服务器在同一个组中

- 9:30用户向服务器A上传了一个文件X(文件创建时间9:30)
- 9:31用户向服务器B上传了一个文件Y(文件创建时间9:31)
- 9:32用户向服务器A上传了一个文件Z(文件创建时间9:32)



A向B，C同步文件X，并且向Tracker Server汇报

- 我向B同步了X(文件创建时间9:30)
- 我向C同步了X(文件创建时间9:30)

![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/FastDFS/storage%20Server%E4%B9%8B%E9%97%B4%E7%9A%84%E6%96%87%E4%BB%B6%E5%90%8C%E6%AD%A5.jpg?raw=true)

![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/FastDFS/storage%20Server%E4%B9%8B%E9%97%B4%E7%9A%84%E6%96%87%E4%BB%B6%E5%90%8C%E6%AD%A52.jpg?raw=true)

通过文件名都可以取到创建时间和源服务器地址

这个时候下载文件Y，tracker会去那个服务器上面寻找呢?

这个时候可以直接从B中下载，如果这个时候B宕机了，只有和A或C中找，查tracker中的表时，B-->A在9：31前的都同步了，就去A中找。如果A也宕机了，就去C中找，看这个时间前的是否同步。

tracker不需要保存左侧的信息，根据文件名就可以了。

![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/FastDFS/storage%20Server%E4%B9%8B%E9%97%B4%E7%9A%84%E6%96%87%E4%BB%B6%E5%90%8C%E6%AD%A53.jpg?raw=true)

这个时候假设用户想下载文件Z，就去源服务器下载，如果源服务器宕机，就去其他服务器下载。根据时间判断是否在这个时间前的同步了，如果同步了就下载，如果没有就下载。

![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/FastDFS/storage%20Server%E4%B9%8B%E9%97%B4%E7%9A%84%E6%96%87%E4%BB%B6%E5%90%8C%E6%AD%A54.jpg?raw=true)

当试图下载文件的时候，根据文件名拿到创建时间和服务器，根据时间来判断是否可以下载。



FastDFS文件同步方式

![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/FastDFS/FastDFS%E6%96%87%E4%BB%B6%E5%90%8C%E6%AD%A5%E6%96%B9%E5%BC%8F.jpg?raw=true)

最后最早同步时间，

对于服务器A来讲，服务器B向他同步的时间是9:31,服务器C是9:33.计算最后的时间，那么根据上图。

对服务器A，它的最早同步时间是9:31

对服务器B，它的最早同步时间是9:32

**取最小值，然后直接和最小值进行比对。**

 ![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/FastDFS/%E6%9C%80%E5%90%8E%E6%9C%80%E6%97%A9%E5%90%8C%E6%AD%A5%E6%97%B6%E9%97%B4.jpg?raw=true)

X可以从三个服务器下载

Y可以从A和B服务器下载，C不行。

Z可以从源服务器A下载，或者服务器B下载,C不行

W只能从服务器C这个源服务器下载，虽然根据时间判断C已经将9:33前的同步到了A和B这两个服务器，但是根据规则，它不能去A和B下载。

虽然简单，但是会牺牲一下。



选择一个可供下载的Storage Server策略

- 该文件上传到源Storage server(文件直接上传到该服务器上的)
- 文件创建时间戳<Storage server被同步到的文件时间戳，这意味着当前文件已经被同步过来了；
- 文件创建时间戳=Storage server被同步到的文件时间戳，且(当前时间-文件创建时间戳)>一个文件同步完成需要的最大时间(如5分钟);
- (当前时间-文件创建时间戳)>文件同步延迟阈值，比如我们把阈值设置为1天，表示文件同步在一天内肯定可以完成



当走到第四个的时候可能已经出问题了。



tracker定位一个组中的server



### FastDFS的使用

![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/FastDFS/FastDFS%E7%9A%84%E4%BD%BF%E7%94%A8.jpg?raw=true)

这里的FastDFSAPI可以当做client

![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/FastDFS/FastDFS%E7%9A%84%E4%BD%BF%E7%94%A8%E5%92%8Cnagix%E9%9B%86%E6%88%90.jpg?raw=true)



上图这种就是

如果上传的话还是走Nginx，到应用服务器

如果下载的时候，直接通过Nginx，直接到FastDFS，模块和Nginx集成。绕过应用服务器





要点：

- Tracker server内部没有数据库，要么是内存要么是纯文件，文件格式自己定义



### 防止盗链

- 辛苦上传的文件不想被人盗取
- 给URL增加token
  - Token只有自己的网站才能生成
  - Token会过期

refer

#### 防止盗链的配置



是否做token检查，缺省为false

http.anti_steal.check_token=true

即生成token的有效时长 秒

http.anti_steal.token_ttl=900

生成token的密钥，尽量设置得长一些

http.anti_steal.secret_key=@#%*&$)87)_+$%!~





Token = md5(文件名，密钥，时间戳)

![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/FastDFS/%E9%98%B2%E6%AD%A2%E7%9B%97%E9%93%BE%E7%9A%84%E9%85%8D%E7%BD%AE.jpg?raw=true)

将文件名、密钥和时间戳的组合通过md5加密传给token

token是在服务器端生成

客户端下载文件的时候，除了传fileid以外还需要token

客户端发送的是fileid，服务器端返回的是一个token





#### 合并存储

- 海量小文件的缺点
  - 元数据管理低效，磁盘文件系统中，目录项(dentry)、索引节点(inode)和数据(data)保存在存储介质的不同位置上
  - 数据存储分散
  - 磁盘的大量随机访问降低效率
- FastDFS提供的合并存储功能
  - 默认大文件64M
  - 每个文件空间称为slot(256bytes<=slot<=16MB)

![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/FastDFS/%E6%B2%A1%E6%9C%89%E5%90%88%E5%B9%B6%E6%97%B6%E7%9A%84%E6%96%87%E4%BB%B6ID.jpg?raw=true)

![](https://github.com/whyathere/tuchuang/blob/master/%E6%A1%86%E6%9E%B6/FastDFS/%E5%90%88%E5%B9%B6%E6%97%B6%E7%9A%84%E6%96%87%E4%BB%B6.jpg?raw=true)



### 总结

- FastDFS是穷人的解决方案
- FastDFS把简洁和高效做到了极致，非常节约资源，中小网站完全用的起

