---
title: jetty-maven-plugin插件的使用
tags: java
---

1. 安装jetty插件 
打开项目的pom.xml文件，然后找到build节点，然后添加如下插件
```xml
<plugin>
  <groupId>org.mortbay.jetty</groupId>
  <artifactId>jetty-maven-plugin</artifactId>
  <version>8.1.16.v20140903</version>
<plugin>

```
<!--more-->
然后重新编译一下，就可以使用jetty插件了，在运行jetty插件需要对setting.xml做修改。
我们知道，maven默认情况下只有org.apache.maven.plugins和org.codehaus.mojo两个grounpId下的插件才支持简化命令调用，像可以运行mav help:system 这样的简介命令。但mvn jetty:run就不行了。因为maven-help-plugin 的groupId是org.apache.maven.plugins，而jetty-maven-plugin的groupId是org.mortbay.jetty。为了能在命令行直接运行mvn  jetty:run，用户需要配置setting.xml 如下：

```xml
<settings>
<pluginGroups>
	<pluginGroup>org.mortbay.jetty</pluginGroup>
</pluginGroups>
  ...
<settings>

```

现在可以使用mvn jetty:run 运行jetty了。Jetty默认监听本地端口为8080，jetty部署的项目的Context  path默认是 / , 也就是说，项目的访问入口是：http://localhost:8080

2. jetty插件的的配置
在pom.xml 文件中的jetyy的plugin节点下，添加configuration的节点就可以配置jetty了。
- Jetty服务的停止配置 
如果希望通过命令 mvn jetty:stop执行关闭jetty服务，则需要如下配置，配置特殊端口 stopPort和控制键 stopKey
```xml
<plugin>
  <groupId>org.mortbay.jetty</groupId>
  <artifactId>jetty-maven-plugin</artifactId>
  <version>8.1.16.v20140903</version>
   <configuration>
	  <stopKey>shoutdown</stopKey>
<stopPort>9900</stopPort>
 </configuration>
<plugin>

```
这下可以通过mvn jetty:stop来停止jetty服务

- 端口的配置

Jetty默认的端口是8080 ，命令行可以修改运行端口：mvn –Djetty.port=8081 jett:run。pom.xml的配置方式如下：

```xml
<plugin>
  ...
   <configuration>
	 <httpConnector>
         <port>8081</port>
		</httpConnector>
 </configuration>
<plugin>

```

- 自动热部署
Jetty的自动热部署命令行方式：mvn –Djetty.ScanIntervalSeconds=10 jetty:run。也可以在pom.xml文件中配置,配置如下:

```
<plugin>
  ...
   <configuration>
	 <scanIntervalSeconds>10</scanIntervalSeconds>
 </configuration>
<plugin>

```
默认值是0 大于0数值表示开启，0表示关闭，单位为秒，已配置数值为一个周期，自动的扫描文件检查其内容是否变化，如果发现文件变化，则自动重新部署。

- 手动重载

Jetty的自动热部署命令行方式：mvn –Djetty.reload =manual jetty:run。也可以在pom.xml文件中配置,配置如下:
默认值为 automatic，它与大于0的scanIntervalSeconds节点一起起作用，实现自动热部署工作。如果将值设置为manual的好处是，当你改变文件内容并保存时，不会马上触发自动扫描和重部署动作，你还可以继续修改，直到你在console或命令行中敲回车（Enter）时候才会触发加载动作。这样可以更加方便的调试修改。

```xml
<plugin>
  ...
   <configuration>
	 <reload >manual</reload>
 </configuration>
<plugin>

```
- Web上下文

Jetty插件的contextPath 的默认值是 /, 在pom.xml文件可以对Web应用的ContextPath 进行配置，配置如下:

```xml
<plugin>
  ...
   <configuration>
	  <webApp>
		<contextPath>/${project.artifactId}</contextPath>
	</webApp>
 </confi

```
${project.artifactId} 引用了 <artifactId> 节点的值，即项目的名称。
项目的静态资源文件目录默认是 src/main/webapp，如果静态资源目录有多个，或者不在默认的 src/main/webapp 目录下，可做如下配置：
```xml
<plugin>
  ...
   <configuration>
	  <webApp>
		<contextPath>/${project.artifactId}</contextPath>
		<resourceBases>
		<resourcBase>${project.basedir}/src/main/webapp</resourceBase>
 		<resourcBase>${project.basedir}/commons</resourceBase>
		</resourceBases>
	</webApp>
 </configuration>
<plugin>

```

引用静态资源文件时，路径不包含资源目录的名称，如 commons/main.css，引用方式为：
```
<link href="main.css" rel="stylesheet" />
```

- 访问日志的配置

在你的 pom.xml 文件添加如下配置： 
```xml
<plugin>
  ...
   <configuration>
	<requestLog implementation="org.eclipse.jetty.server.NCSARequestLog">
  		<filename>target/access-yyyy_mm_dd.log</filename>
            <filenameDateFormat>yyyy_MM_dd</filenameDateFormat>
  		<logDateFormat>yyyy-MM-dd HH:mm:ss</logDateFormat>
  		<logTimeZone>GMT+8:00</logTimeZone>
  		<append>true</append>
 		 <logServer>true</logServer>
  		<retainDays>120</retainDays>
  		<logCookies>true</logCookies>
 	</requestLog>
 </configuration>
<plugin>

```

org.eclipse.jetty.server.NCSARequestLog 是 org.eclipse.jetty.server.RequestLog 的一个实现类。
org.eclipse.jetty.server.NCSARequestLog 是一种伪标准的 NCSA 日志格式。下面是一些节点参数的解释：
filename：日志文件的名称
filenameDateFormat：日志文件的名称的日期格式，它要求日志文件名必须含有 yyyy_mm_dd 串
logDateFormat：日志内容的时间格式
logTimeZone：时区
append：追加到日志
logServer：记录访问的主机名
retainDays：日志文件保存的天数, 超过删除
logCookies：记录 cookies
启动 jetty 服务，在项目的 target 目录下会生成一个 access-2017_03_10.log 文件，该文件中的其中一条记录如下：

```
//=====log start eg======
localhost 0:0:0:0:0:0:0:1 -  -  [2017-03-10 13:44:32] "GET /operation/index.jsp HTTP/1.1" 200 138 "-" "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.87 Safari/537.36" "JSESSIONID=o35qtph2hwe1fzenoqeys1vz"
...
//=====log end eg======
```
