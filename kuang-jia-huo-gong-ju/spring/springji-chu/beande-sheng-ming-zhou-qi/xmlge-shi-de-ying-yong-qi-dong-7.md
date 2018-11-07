第十二步：finishRefresh，结束刷新

```java
protected void finishRefresh() {
    //清空context级别的资源缓存
    clearResourceCaches();

    //初始化LifecycleProcessor
    initLifecycleProcessor();

    //回调LifecycleProcessor的onRefresh方法
    getLifecycleProcessor().onRefresh();

    //发布context刷新事件
    publishEvent(new ContextRefreshedEvent(this));

    // Participate in LiveBeansView MBean, if active.
    LiveBeansView.registerApplicationContext(this);
}
```



