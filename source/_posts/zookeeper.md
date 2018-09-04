---
title: zookeeper客户端命令的使用
date: 2018-05-04 15:35:00
tags: java
---

当安装了zookeeper服务器，想快速上手使用zookeeper，可以选用命令行客户端链接zookeeper服务器。通过命令行，可以快速使用zookeeper的相关功能，已达到快读体验的目的。

1. 连接zookeeper服务器

```bash
$ bin/zkCli.sh -server 127.0.0.1:2181

//执行命令后的结果
Connecting to localhost:2181
log4j:WARN No appenders could be found for logger (org.apache.zookeeper.ZooKeeper).
log4j:WARN Please initialize the log4j system properly.
Welcome to ZooKeeper!
JLine support is enabled
[zkshell: 0]
        

```

2. help命令

```bash

[zkshell: 0] help
ZooKeeper host:port cmd args
        get path [watch]
        ls path [watch]
        set path data [version]
        delquota [-n|-b] path
        quit
        printwatches on|off
        createpath data acl
        stat path [watch]
        listquota path
        history
        setAcl path acl
        getAcl path
        sync path
        redo cmdno
        addauth scheme auth
        delete path [version]
        setquota -n|-b val path


```

3. 查看节点目录

```bash
[zkshell: 8] ls /
[zookeeper]
        
```

4. 创建节点 存入数据

```bash
[zkshell: 9] create /zk_test my_data
Created /zk_test

```
查看节点目录

```bash
[zkshell: 11] ls /
[zookeeper, zk_test]

```

5. 获取节点目录数据

```bash
[zkshell: 12] get /zk_test
my_data
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 5
mtime = Fri Jun 05 13:57:06 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 0
aclVersion = 0
ephemeralOwner = 0
dataLength = 7
numChildren = 0

```

6. 更新节点目录数据

```bash
[zkshell: 14] set /zk_test junk
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 6
mtime = Fri Jun 05 14:01:52 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0
dataLength = 4
numChildren = 0
[zkshell: 15] get /zk_test
junk
cZxid = 5
ctime = Fri Jun 05 13:57:06 PDT 2009
mZxid = 6
mtime = Fri Jun 05 14:01:52 PDT 2009
pZxid = 5
cversion = 0
dataVersion = 1
aclVersion = 0
ephemeralOwner = 0
dataLength = 4
numChildren = 0
```

7. 删除节点目录

```bash
[zkshell: 16] delete /zk_test
[zkshell: 17] ls /
[zookeeper]
[zkshell: 18]
```