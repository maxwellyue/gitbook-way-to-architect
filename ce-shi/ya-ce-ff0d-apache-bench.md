# 压测:apache bench

## 压力测试

什么是压力测试

> 模拟实际应用的软硬件环境及用户使用过程的系统负荷，长时间或超大负荷地运行测试软件，来测试被测系统的性能、可靠性、稳定性等，主要是得到系统的处理峰值。

下面是压力测试相关的一些概念。

1. **吞吐率（Requests per second）** 概念：服务器并发处理能力的量化描述，单位是reqs/s，指的是某个并发用户数下单位时间内处理的请求数。 计算公式：总请求数 / 处理完成这些请求数所花费的时间。
2. **并发连接数（The number of concurrent connections）** 概念：某个时刻服务器所接受的请求数目。
3. **并发用户数（The number of concurrent users，Concurrency Level）** 概念：要注意区分这个概念和并发连接数之间的区别，一个用户可能同时会产生多个会话，也即连接数。
4. **用户平均请求等待时间（Time per request）** 计算公式：处理完成所有请求数所花费的时间/ （总请求数 / 并发用户数）
5. **服务器平均请求等待时间（Time per request: across all concurrent requests）** 计算公式：处理完成所有请求数所花费的时间 / 总请求数，是吞吐率的倒数。

### apache bench

apache bench简称为ab，是Apache出品的HTTP的性能测试工具，安装简单，使用方便。

比如，我们想要对某个接口进行一次“并发数为10，总请求数为100”的一次测试，只需要使用如下命令：

```text
ab -n 100 -c 10 http://test.com/
```

常用参数如下：

| 参数 | 说明 |
| :--- | :--- |
| -n | 在一次会话中所执行的请求个数。默认为1。 |
| -c | 并发数，一次产生的请求个数。默认是一次一个。 |
| -t | 测试进行的最大秒数 |
| -s | 响应等待的最大秒数，默认30s |
| -p | 执行post请求时，需要提交的数据，需要同时使用-T来指定content-type |
| -u | 执行put请求时，需要提交的数据，需要同时使用-T来指定content-type |
| -T | 执行POST/PUT请求时，指定content-type |
| -C | 添加Cookie，可以重复使用该参数 |
| -H | 添加HTTP头部属性 |
| -m | 指定执行请求时的方法： |
| -w | 以HTML表格的方式输出结果 |
| -k | 使用KeepAlive长连接，默认不启用 |
| -i | 使用HEAD请求（默认为GET请求） |

#### 更多示例

## 参考

[超实用压力测试工具－ab工具](https://www.jianshu.com/p/43d04d8baaf7)

[ApacheBench 参数讲解与基础使用](https://github.com/luisedware/Archives/issues/2)

[烂泥：apache性能测试工具ab的应用](https://www.ilanni.com/?p=8348)

