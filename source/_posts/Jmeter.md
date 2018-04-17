---
title: Jmeter
author: Kairou Zeng
date: 2018/03/20
tags: [测试]
---

# Jmeter 测试相关

## 线程组

一个性能测试请求负载是基于一个线程组完成的。一个测试计划必须有一个线程组。

JMeter中，每个测试计划至少需要包含一个线程组，当然也可以在一个计划中创建多个线程组。

在测试计划下面，多个线程是并行执行的，也就是说这些线程组是同时被初始化并同时执行线程组下的Sampler的。

线程组主要包括三个参数：线程数、准备时长(Ramp-Up Period(in seconds))、循环次数。

`线程数`：虚拟用户数。一个虚拟用户占用一个进程或线程。设置多少虚拟用户数这里就设置多少个线程数。

`准备时长`：设置的虚拟用户数需要多长时间全部启动。如果线程数为20，准备时长为10，那么需要10秒钟启动20个线程，也就是每秒启动2个线程。

`循环次数`：每个线程发送请求的次数。如果线程数为20，循环次数为100，那么每个线程发送100次请求。总请求数为20*100=2000。如果勾选了“永远”，那么所有线程会一直发送请求，直到选择停止运行脚本。

## JmeterPlugin 说明

**Ramp-up Period(in seconds):**

1. 决定多长时间启动所有线程。如果使用10个线程，ramp-up period是100秒，那么JMeter用100秒使所有10个线程启动并运行。每个线程会在上一个线程启动后10秒(100/10)启动。

Ramp=up需要充足长以避免在启动测试时有一个太大的工作负载，并且要充足小以至于最后一个线程在第一个完成前启动。

2. 用于告知JMeter 要在多长时间内建立全部的线程。默认值是0。如果未指定ramp-up period ，也就是说ramp-up period 为零， JMeter 将立即建立所有线程。假设ramp-up period 设置成T 秒， 全部线程数设置成N个， JMeter 将每隔T/N秒建立一个线程。

3. Ramp-Up Period(in-seconds)代表隔多长时间执行，0代表同时并发

### Over Time

`Response Times Over Time`：响应时间变化曲线。X轴表示的是系统运行的时刻，Y轴表示的是响应时间，F(X,Y)表示系统随着时间的推移，系统的响应时间的变化，可以看出响应时间稳定性。

`Response Time Percentiles Over Time`: 响应时间百分比，X轴表示的是百分比，Y轴表示的是响应时间，F(X,Y)表示低于某个百分比的响应时间

`Active Threads Over Time`：随着时间推移活跃线程数， X轴表示访问的时刻，Y轴表示活动线程数，F(X,Y)表示某个时刻的活动线程数

`Bytes Throughput Over Time`：随着时间推移每秒接收和请求字节数变化趋势图，蓝色为每秒发送字节数，黄色为每秒接收字节数

`Latencies Over Time`：每秒钟的响应等待时间， 表明Jmeter测试期间，随着时间的推移系统的响应等待时间的变化，也是系统随着时间推移，系统效率的变化。

`Connect Time Over Time`：请求连接建立的时间

----
### Throughput

`Hits per second`：每秒点击率

`Codes per second`：每秒状态码数量

`Transactions per second`：每秒事务量

`Response Times vs Threads`：响应时间用户数， X轴表示的是活动线程数，也就是并发访问的用户数，Y轴表示的是响应时间，F(X,Y)表示在某种并发量的情况下，系统的响应时间是多少。

`Latency Vs Request`: 延迟时间点请求的 成功/失败 数

----
### Response Times

`Response Time Percentiles`：响应时间百分比

`Response Time Overview`：响应时间分布

`Time Vs Threads`: 测试过程中的线程数时续图

`Response Time Distribution`: 响应时间分布

## JmeterPlugin 参数说明

`Samples`：运行的线程数（也可理解为请求数）

`Average`：平均响应时间，单位：ms

`Median`：中位数，即50%用户的响应时间

`90%Line`：90%用户的响应时间

`95%Line`：95%用户的相应时间

`99%Line`：99%用户的响应时间

`Min`：最小响应时间

`Max`：最大响应时间

`Error`：本次测试中出现错误的请求的数量/请求的总数

`Throughput`：吞吐量-每秒完成的请求数

`Received KB/sec`：每秒从服务器端接收到的数据量

`Sent KB/sec`：发送的千字节每秒的吞吐量测试

`KB/sec`：每秒从服务器端接收到的数据量

`Elapsed time`：经过的时间(= Sample time = Load time = Response time ) 
这个时间是我们测试常用的时间，也是整个请求的消耗时间，从发送到接收完成全程消耗的时间

`Latency time`：延迟时间。不常用，表示请求发送到刚开始接收响应时，这个时间<Elapsed time

`Connection time`：不常用，请求连接建立的时间，这个时间 < Latency time < Elapsed time
