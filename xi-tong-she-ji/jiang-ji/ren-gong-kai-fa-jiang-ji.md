# 人工降级开关的实现

在开启、关闭降级开关时，我们需要使用可配置的方式来动态调整。在应用时，首先要封装一套应用层API，以便业务逻辑使用。对于开关数据的存储，如果涉及的服务器/系统较少，初期可以使用配置文件的方式进行配置。如果涉及的服务器/系统较多，则可以使用配置中心（如Consule，Etcd等）进行配置。

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

##  **配置文件实现开关配置**

使用properties文件作为配置文件，借助JDK 7 WatchService实现文件变更监听，实现代码如下所示。

```text
static {     try {         filename= "application.properties";         resource= new ClassPathResource(filename);         //监听filename所在目录下的文件修改、删除事件        watchService = FileSystems.getDefault().newWatchService();        Paths.get(resource.getFile().getParent()).register(watchService,StandardWatchEventKinds.ENTRY_MODIFY, StandardWatchEventKinds.ENTRY_DELETE);         properties= PropertiesLoaderUtils.loadProperties(resource);     } catch(IOException e) {e.printStackTrace();}     //启动一个线程监听内容变化，并重新载入配置     Thread watchThread = new Thread(() -> {         while(true) {             try{                WatchKey watchKey = watchService.take();                for (WatchEvent<?> event : watchKey.pollEvents()) {                    if (Objects.equal(event.context().toString(), filename)){                         properties =PropertiesLoaderUtils.loadProperties (resource);                         break;                    }                }                watchKey.reset();             } catch (Exception e){e.printStackTrace();}         }     });    watchThread.setDaemon(true);    watchThread.start();       Runtime.getRuntime().addShutdownHook(newThread(() -> {         try{             watchService.close();         } catch(IOException e) {e.printStackTrace();}     }));   } 
```

* 使用WatchService监听“application.properties”文件所在目录内容变化，包括修改、删除事件。
* 通过后台线程实现阻塞等待内容变化事件，一旦发现有内容变更，如果是“application.properties”文件发生变更，则重新装载配置文件。
* JVM停止时记得关闭WatchService。

整体实现比较简单，然后就可以封装Properties实现自己的开关API了。通过配置文件的方式缺点是每次配置文件内容变更需要将配置文件同步到服务器上，这点算是比较麻烦的，如果自动部署系统支持动态更改配置文件并同步用这种方式，那么也并不麻烦。只是如果要维护多个项目时，则需要切换多个界面来操作。

**3. 配置中心实现开关配置**

统一配置中心，或者叫分布式配置中心，目的是实现配置开关的集中管理，要有配置后台方便开关的配置，对于一般公司来说配置中心的维护要简单，不需要投入过多的人力来做这件事情，配置中心不管是采用拉取模式还是推送模式，要考虑到连接数和网络带宽可能带来的风险和问题。目前有一些开源方案可以选择，如Zookeeper、Diamond、Disconf、Etcd3、Consul。本文选择了Consul，其支持多数据中心、服务发现、KV存储等特性，而且使用简单，提供了简单的Web UI方便管理，更多介绍可以参考Nginx负载均衡部分。我们借助Consul的KV存储特性来实现配置管理。

启动Consul Server





使用配置文件的方式

