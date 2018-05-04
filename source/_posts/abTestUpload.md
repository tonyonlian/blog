---
title: 如何使用ab测试上传文件 
tags: 测试
---

#### 1. 简单了解ab测试

ab是Apache超文本传输协议(HTTP)的性能测试工具。可以使用工具对网络接口进行压力测试，以判断网络接口的性能。

一般对网络接口进行压力测试，需要关注几个重要的指标，吞吐率，响应时间，并发数。通过这些指标的测试，可以反映出服务器的并发能力。

最常用的的工具有 ab 、siege、http_load等，本文主要说如何使用ab测试上传文件接口，在进入主题之前写大体了解下ab测试报告。

```
[root@iZ2325lrssqZ ~]# ab -n 1000 -c 10  http://localhost:8089/getAllApp
This is ApacheBench, Version 2.3 <$Revision: 655654 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 10.139.97.238 (be patient)
Completed 100 requests
Completed 200 requests
Completed 300 requests
Completed 400 requests
Completed 500 requests
Completed 600 requests
Completed 700 requests
Completed 800 requests
Completed 900 requests
Completed 1000 requests
Finished 1000 requests


Server Software:        
Server Hostname:        10.139.97.238
Server Port:            8089

Document Path:          /getAllApp
Document Length:        663124 bytes

Concurrency Level:      10
Time taken for tests:   115.019 seconds
Complete requests:      1000
Failed requests:        0
Write errors:           0
Total transferred:      663340000 bytes
HTML transferred:       663124000 bytes
Requests per second:    8.69 [#/sec] (mean)
Time per request:       1150.193 [ms] (mean)
Time per request:       115.019 [ms] (mean, across all concurrent requests)
Transfer rate:          5632.04 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        1    1   0.7      1      10
Processing:   371 1139 382.2   1093    2782
Waiting:      320  903 314.1    867    2249
Total:        372 1140 382.2   1095    2783

Percentage of the requests served within a certain time (ms)
  50%   1095
  66%   1242
  75%   1360
  80%   1427
  90%   1690
  95%   1863
  98%   2107
  99%   2255
 100%   2783 (longest request)
```
<!--more-->
上述命令表示10个并发用户向服务器请求1000次，下面简单说明下报告中的各项指标含义：

- Document Path：表示请求路径， 这里为/getAllapp；
- Document Length: 表示报文的大小， 这里为663124bytes；
- Concurrency Level: 表示并发级别 ，就是命令中传入的-c， 此处为10，即10个并发；
- Time taken for tests: 表示完成所有测试用时；
- Complete requests: 表示测试中一共完成了多少个请求，就是命令中-n 的参数1000；
- Failed requests: 表示失败的请求个数；
- Write errors: 表示写入过程中出现的错误次数；
- Total transferred: 表示响应的所有报文大小；
- HTML transferred: 表示仅HTTp报文的正文大小，它会比上一个值小；
- Requests per second: 表示服务器没秒能处理多少请求，是我们重点关注的指标，能反映服务器的并发能力，这个值又称RPS或QPS；
- Time per request: 两个这样的指标，第一个表示用户平均等待时间，第二个表示服务器平均请求处理时间；
```
  Time per request(第二个) = Time per request(第-个)/c(并发数） 
```
- Transfer rate: 表示传输率，等于传输大小除以传输时间，这个值受网卡带宽限制；
- Connection Times: 连接时间，它包括客户端向服务端建立连接，服务器端处理请求，等待报文响应的整个过程。


#### 2. ab 上传文件

ab提交文件是以multipart/form-data 形式提交的，ab中上传文件一般命令为：

```
ab -v 2 -T 'multipart/form-data; boundary=----WebKitFormBoundaryM1tLdAWapR8WCJSe' -p ./abpost.txt http://localhost:7080/file/upload
```

指定T参数 （T参数表示POST数据所使用的Content-type头信息），
指定p参数（p包含了需要POST的数据的文件），文件数据是包含在abpost.txt文件中，这个文件内部结构符合rfc1867协议规定。

下面我们就一步步说明怎么构造abpost.txt文件上传数据

1. 定义边界线

我们在ab的T参数中定义http协议的Content-type 同时也约定了边界线。如上边命令中定义的边界线:

```
boundary=----WebKitFormBoundaryM1tLdAWapR8WCJSe

```
注意“=” 后面是紧跟着的四条短线（-）

2. 2.构造abpost.txt文件内容

这个文件可放在ab命令使用的当前目录，这样就按照上面的写法，作为p参数。先贴出abpost.txt内容，后面再做解释。

```
------WebKitFormBoundaryM1tLdAWapR8WCJSe
Content-Disposition: form-data; name="file"; filename="file.jpg"
Content-Type: image/jpeg

iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAYAAADgdz34AAAABHNCSVQICAgIfAhkiAAAAAlwSFlzAAAFMQAABTEBt+0oUgAAABl0RVh0U29mdHdhcmUAd3d3Lmlua3NjYXBlLm9yZ5vuPBoAAAVUSURBVEiJpZRZbFzVHcZ/5965c2ffV9sz8RjHdpyxweNEoSRN2YJLwxJBqYTahz6URSC1UqtWbSOlIPHQVuoDFeKpaaWqUh+iShRUElUCTBZRm8iBAnHiNfEyZuzxzHjuzHj224fUI4whQeJI5+Ecne/7/b9zjv7ous5XmQ6L/B3Aous6gBHwfBWdxC2G32F8Sgihhm3q8MEu19mQy9zlMCuJvSHbq7fSArcGRJymBx4fDI6ntNrGQ/3+4XjA8reoS33owT3e+zrctju+NqBYbXz0cNw/6DDJz9pVA30h62A8ZPt+PGz3+S3Sk18LYBciYDJIxwC+e3uoz6bK7A3arCN7fJ2VehNZkr4d8qjxm3mI/z8aAG4hXJ0d9uPVejNVrbNyYJfj1/ujzn6X2bBDuFlrIBB8mNQWzs1mXpFkyeU2GzqvrVX+cC1XnNgB2OO3DveErH9+NB4YLFQaldVCpdETsFliu3sJtkcQAj5dWqRer+P1B5ENMqsrSVLJJa6uZOtuk6K7LIpyaSmffHc6c2JiOX8SoFVau8d8/Kn7BweHDhzk3Oio5DRn1L1D+4h2dbeqdnl821IEwu3s7h9AeuffBiEp+uEjI0Q/nGibSZ//AXBy2xu4zIYeh9PF8nKyXtEyisPlJtrVzaamsTB6lvT0TMu4mM2yeP4CxUwG1WQiGuumpOXE5OXLNbvDScCm9gkhrNsSrBWqq1NXr/YaFYMBwOF0AbD+3jjdb15gsTdCPbYLg0Eh+845bjv7AbPfXMP6+DFkWQZgYeoTpdwQxWvZ0qyu68VtCS7NZp94/ePUit6oA2Cx2QGQvG5WQy6KATcGgwJA0+Ug7Xeiu5wA5DLrAOjNJuNz6cql5cJzW76tBBooMa/F9vnf0jacYLOvh5jF2tqL3ns3xf0JonYHpUKBXGadaFc32fQafcGq2zqVbgf+uw0QdZmO7PaZ3VvrxWuzRGJdqCYzZquN02OXSfREqNbqLKU3+EZ/JwCKauTuBx9uJai++U8RshkPAae3ATw2ZcCu3lj6giHaIrsoaHlUkxmASr3BP85+QLFcZX9vtJVGUYykshpehxWDLOH1B/Fa59t2XlG5MVuqNtgVjbD/0Lc+f1McOzhAuVpDMcjIkkQqq3F+Jk3d7OXS5Ay/fSIBQKPRIFuqr+4AzKZLZ+bWN7WhhM8OMH51mcFYAJNRaUHypTLnZ9apmb2UJAs/+c0rqKrKn35/onVmOZUmla+O7QDoun79e0OhRW1jox/AYDTy/MlRjt5zF0I2MDW/yOH7R/jhiWNIksSpU6dQVfWGuJwHTORzWSbmU+nruc13t3y3NbvkRmUyuXidbHqNRMzPzx/dh9UX5ulfvMihkUc4fN8IqqoihGCrxUxdmSSoNgCYn7rC7NrmFV3X178QMLdS/tm5udz8RxPjNJtN+trc7NGX+N2vfoyW30DTtBsiSaLZbJLL5Xj9L3/kzp4w66sp3r74SWY6XXjhs57builAvN3xy6cPtL3Uc1tMjif2tX7Ra2PTlCwBfB1d+IJhzvzrDbqdEo8NRygXNS6Mvs3Lb02ffH8h96ObAoQQ4s5Ox2tPJsJHvXaL3Be/nUhXN0IIACq1OlmtRMjjQNd1UsklLo79h7+OLbw3Op05stUivhSwBRkI247f2+N5Zqjd0WF3OHG6PZitNoQQ1KoVKpUK6dSnTC5nMqcn03+/uJj/qa7r1R1eXwTYGh1u2x0DYfMLQZuxN+Yxd0bcJpMkBMl8pTa3VlpIl2ozHy8XXp7JFE9/mcdNAZ9JJFkVZSDiUY7KQjItbJTPaJv193Vdr91K+z+BklGf3fugXAAAAABJRU5ErkJggg==
------WebKitFormBoundaryM1tLdAWapR8WCJSe--
```
注意：abpost.txt文件最好使sublime之类的文本编辑编辑，不要使用Windows下的记事本编辑。


```
------WebKitFormBoundaryM1tLdAWapR8WCJSe
```
开始边界。这是第一行，与边界线T参数里面定义的一样，但必须注意字母前面的短线要变成六条，而不是之前T参数中的四条。

```
Content-Disposition: form-data; name="file"; filename="file.jpg"
Content-Type: image/jpeg
```
头部信息。紧接着的两行是头部信息，包含“Content-Disposition”，
“Content-Type”。其中“Content-Disposition”要指明数据格式，服务端接收文件的名字（file），以及上传文件的名字
而在 Content-Type 中，一般就是指明MIME类型

```
iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAYAAADgdz34AAAABHNCSVQICAgIfAhkiAAAAAlwSFlzAAAFMQAABTEBt+0oUgAAABl0RVh0U29mdHdhcmUAd3d3Lmlua3NjYXBlLm9yZ5vuPBoAAAVUSURBVEiJpZRZbFzVHcZ/5965c2ffV9sz8RjHdpyxweNEoSRN2YJLwxJBqYTahz6URSC1UqtWbSOlIPHQVuoDFeKpaaWqUh+iShRUElUCTBZRm8iBAnHiNfEyZuzxzHjuzHj224fUI4whQeJI5+Ecne/7/b9zjv7ous5XmQ6L/B3Aous6gBHwfBWdxC2G32F8Sgihhm3q8MEu19mQy9zlMCuJvSHbq7fSArcGRJymBx4fDI6ntNrGQ/3+4XjA8reoS33owT3e+zrctju+NqBYbXz0cNw/6DDJz9pVA30h62A8ZPt+PGz3+S3Sk18LYBciYDJIxwC+e3uoz6bK7A3arCN7fJ2VehNZkr4d8qjxm3mI/z8aAG4hXJ0d9uPVejNVrbNyYJfj1/ujzn6X2bBDuFlrIBB8mNQWzs1mXpFkyeU2GzqvrVX+cC1XnNgB2OO3DveErH9+NB4YLFQaldVCpdETsFliu3sJtkcQAj5dWqRer+P1B5ENMqsrSVLJJa6uZOtuk6K7LIpyaSmffHc6c2JiOX8SoFVau8d8/Kn7BweHDhzk3Oio5DRn1L1D+4h2dbeqdnl821IEwu3s7h9AeuffBiEp+uEjI0Q/nGibSZ//AXBy2xu4zIYeh9PF8nKyXtEyisPlJtrVzaamsTB6lvT0TMu4mM2yeP4CxUwG1WQiGuumpOXE5OXLNbvDScCm9gkhrNsSrBWqq1NXr/YaFYMBwOF0AbD+3jjdb15gsTdCPbYLg0Eh+845bjv7AbPfXMP6+DFkWQZgYeoTpdwQxWvZ0qyu68VtCS7NZp94/ePUit6oA2Cx2QGQvG5WQy6KATcGgwJA0+Ug7Xeiu5wA5DLrAOjNJuNz6cql5cJzW76tBBooMa/F9vnf0jacYLOvh5jF2tqL3ns3xf0JonYHpUKBXGadaFc32fQafcGq2zqVbgf+uw0QdZmO7PaZ3VvrxWuzRGJdqCYzZquN02OXSfREqNbqLKU3+EZ/JwCKauTuBx9uJai++U8RshkPAae3ATw2ZcCu3lj6giHaIrsoaHlUkxmASr3BP85+QLFcZX9vtJVGUYykshpehxWDLOH1B/Fa59t2XlG5MVuqNtgVjbD/0Lc+f1McOzhAuVpDMcjIkkQqq3F+Jk3d7OXS5Ay/fSIBQKPRIFuqr+4AzKZLZ+bWN7WhhM8OMH51mcFYAJNRaUHypTLnZ9apmb2UJAs/+c0rqKrKn35/onVmOZUmla+O7QDoun79e0OhRW1jox/AYDTy/MlRjt5zF0I2MDW/yOH7R/jhiWNIksSpU6dQVfWGuJwHTORzWSbmU+nruc13t3y3NbvkRmUyuXidbHqNRMzPzx/dh9UX5ulfvMihkUc4fN8IqqoihGCrxUxdmSSoNgCYn7rC7NrmFV3X178QMLdS/tm5udz8RxPjNJtN+trc7NGX+N2vfoyW30DTtBsiSaLZbJLL5Xj9L3/kzp4w66sp3r74SWY6XXjhs57builAvN3xy6cPtL3Uc1tMjif2tX7Ra2PTlCwBfB1d+IJhzvzrDbqdEo8NRygXNS6Mvs3Lb02ffH8h96ObAoQQ4s5Ox2tPJsJHvXaL3Be/nUhXN0IIACq1OlmtRMjjQNd1UsklLo79h7+OLbw3Op05stUivhSwBRkI247f2+N5Zqjd0WF3OHG6PZitNoQQ1KoVKpUK6dSnTC5nMqcn03+/uJj/qa7r1R1eXwTYGh1u2x0DYfMLQZuxN+Yxd0bcJpMkBMl8pTa3VlpIl2ozHy8XXp7JFE9/mcdNAZ9JJFkVZSDiUY7KQjItbJTPaJv193Vdr91K+z+BklGf3fugXAAAAABJRU5ErkJggg==
```
上传的文件数据。数据就是真实的文件数据的 base64 编码后的数据（可以使用在线base64编码文件数据），与之前的文件头信息空格一行。

```
------WebKitFormBoundaryM1tLdAWapR8WCJSe--
```

文件数据最后是结尾边界。一定要注意： 前面有6条线，字母尾部有2条短线，短线的数量严格遵守，否则文件无法上传成功。主要是原因是，如果不严格按照规范写，会导致http提交报文无法正确解析的。

到此abpost.txt文件构造完毕。将构造好的abpost.txt文件放在执行ab命令的当前目录，换上自己的文件上传接口地址，执行命令就可以。



