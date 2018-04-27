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



