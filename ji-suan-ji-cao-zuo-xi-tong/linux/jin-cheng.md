# 进程

## 如何在后台启动某程序

```bash
# 示例1：后台启动filebeat
# 原始的启动方法
./filebeat -e -c filebeat.yml
# 使其在后台执行
nohup ./filebeat -e -c filebeat.yml &


# 示例2：后台启动java程序
# 原始的启动方法
java -jar xxxxxxxx.jar
# 使其在后台执行
nohup java -jar xxxxxxxx.jar &
```

上面例子中nohup和&均是后台启动程序的方式，但是它们的功效不同：

使用**&**后台运行程序：

* 结果会输出到终端
* 使用Ctrl + C发送SIGINT信号，程序免疫
* 关闭session发送SIGHUP信号，程序关闭

使用**nohup**运行程序：

* 结果默认会输出到nohup.out
* 使用Ctrl + C发送SIGINT信号，程序关闭
* 关闭session发送SIGHUP信号，程序免疫

**平日线上经常使用nohup和&配合来启动程序**：

* 同时免疫SIGINT和SIGHUP信号





## 参考

\[一分钟了解nohup和&的功效\]\([https://mp.weixin.qq.com/s/nyT-FPdIUdJUiUCYVGEnTg](https://mp.weixin.qq.com/s/nyT-FPdIUdJUiUCYVGEnTg)\)

