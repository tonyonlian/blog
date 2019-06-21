---
title: 解决issue make： *** [Release/obj.target/fibers/src/fibers.0] Error 1 
date: 2019-06-21 22:31:00
tags: node 
---


1. 问题：在centos6.5的ecs上执行node install保存如下:

make: *** [Release/obj.target/fibers/src/fibers.0] Error 1
make: leaving directory
gyp ERR! build error
gyp ERR! Error:`make` failed with exit code:2

![gCLXdR.png](https://t1.picb.cc/uploads/2019/06/21/gCLXdR.png)

2. 解决办法：

- (1) 安装python2.7
- (2) gcc升级到4.8

```bash
wget http://people.centos.org/tru/devtools-2/devtools-2.repo -O /etc/yum.repos.d/devtoolset-2.repo
yum -y install devtoolset-2-gcc devtoolset-2-gcc-c++ devtoolset-2-binutils
scl enable devtoolset-2 bash
echo "source /opt/rh/devtoolset-2/enable" >>/etc/profile

```
3. 如何升级gcc参考下方：

[《为CentOS 6、7升级gcc至4.8、4.9、5.2、6.3、7.3等高版本》](https://www.vpser.net/manage/centos-6-upgrade-gcc.html)