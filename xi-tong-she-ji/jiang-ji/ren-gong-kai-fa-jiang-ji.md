# 人工降级开关的实现

在开启、关闭降级开关时，我们需要使用可配置的方式来动态调整。在应用时，首先要封装一套应用层API，以便业务逻辑使用。对于开关数据的存储，如果涉及的服务器/系统较少，初期可以使用配置文件的方式进行配置。如果涉及的服务器/系统较多，则可以使用配置中心（如Consule，Etcd等）进行配置。

其实整体的思路就是将降级开关做成应用的配置项，然后借助配置文件或配置中心实时获取配置项的变化。配置文件和配置中心的实时更新本身与降级这个主题关系不大。

## 应用层API封装

**1. 应用层API封装**

如下是我们抽象并封装的开关API。

```java
USER(     
    "用户信息",    
    "user.not.call.backend", "是否调用后端服务",    
    "user.call.backend.rate.limit", "调用后端服务的限流",    
    "user.redis.expire.seconds", "redis缓存过期时间"), 
```

其中：

* `user.not.call.backend`：是否调用后端用户服务。如果不开启，那么只会访问缓存，不会将流量打到后端。
* `user.call.backend.rate.limit`：调用后端服务的限流大小，比如配置100，即一秒只有100个请求会打到后端服务，剩余请求如果缓存没用命中时，则直接返回空数据或错误。
* `user.redis.expire.seconds`：后端返回的用户数据在缓存中缓存多久。

通过封装后，我们可以很简单地使用这些API。

```java
//如果禁止访问后端服务，则直接返回空值
if (Switches.USER.notCall()) {     
    retur null; 
} 
//如果允许了后端用户服务，则将服务结果放到缓存中
cacheService.set(CacheKeys.getUserKey(pin), info, Switches.USER.getExpiresInSeconds()); 
```

API实现是从配置文件获取相关配置，如果没有，则返回一个默认值。

```java
public boolean notCall() {     
    return DynamicConfigurer.getBoolean(callKey, false); 
} 
public int getExpiresInSeconds() { 
    return DynamicConfigurer.getInt(expiresKey, DEFAULTEXPIRES_IN SECONDS); 
}
```

## 使用**配置文件的方式**

使用properties文件作为配置文件，借助JDK 7 WatchService实现文件变更监听，实现代码如下所示。

```java
public class Configs {
    public static Properties properties;
    static {
        final String configFilename = "app.properties";
        final ClassPathResource resource = new ClassPathResource(configFilename);
        try {
            //先加载一次配置文件
            properties = PropertiesLoaderUtils.loadProperties(resource);
            //设置监听配置文件所在目录的文件删除/新增操作
            final WatchService watcher = FileSystems.getDefault().newWatchService();
            Paths.get(resource.getFile().getParent()).register(watcher,
                    StandardWatchEventKinds.ENTRY_CREATE,
                    StandardWatchEventKinds.ENTRY_DELETE,
                    StandardWatchEventKinds.ENTRY_MODIFY);
            //开启监听线程
            Thread watchThread = new Thread(() -> {
                try {
                    WatchKey watchKey = watcher.take();
                    for (WatchEvent<?> event : watchKey.pollEvents()) {
                        //如果是配置文件发生变化，则重新加载配置文件
                        System.out.println(event.kind() + ":::" + event.context().toString());
                        if (Objects.equals(configFilename, event.context().toString())) {
                            properties = PropertiesLoaderUtils.loadProperties(resource);
                            break;
                        }
                    }
                    watchKey.reset();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            });
            watchThread.setDaemon(true);
            watchThread.start();

            //JVM停止时，关闭watcher线程
            Runtime.getRuntime().addShutdownHook(new Thread(() -> {
                try {
                    watcher.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public static boolean getBoolean(String key) {
        String property = properties.getProperty(key, "false");
        return "true".equals(property);
    }
    public static int getInt(String key) {
        String property = properties.getProperty(key, "0");
        return Integer.valueOf(property);
    }
}
```

这样，每次需要修改配置文件内容，可以用修改后的配置文件替换原配置文件。

> 如上方式，并不能监听到配置文件的直接修改操作，只能使用新配置文件替换旧配置文件的方式。

## 使用**配置中心的方式**

通过配置中心修改开关配置，应用程序去获取这些变更（推送或拉取），思路与使用配置文件的方式是一致的。

目前有一些开源方案可以选择，如Zookeeper、Diamond、Disconf、Etcd3、Consul。



## 内容来源

《亿级流量网站架构核心技术》：降级详解







