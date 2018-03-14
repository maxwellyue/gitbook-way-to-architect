

调度器本质上还是通过juc的ScheduledExecutorService进行的

调度器启动后你无法通过修改系统时间达到让它马上执行的效果

被@Schedule注解的方法如果有任何Throwable出现, 不会中断后续Task, 默认只会打印Error日志，定时任务不会同时被触发。



http://hstrust.iteye.com/blog/2316429



