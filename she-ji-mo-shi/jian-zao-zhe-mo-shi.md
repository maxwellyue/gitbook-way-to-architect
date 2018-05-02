# Builder模式

---

**一般在什么场景下使用建造者模式？**

①需要的这个对象非常复杂，即它包含非常多的属性

②对象构建出来就保持不变或很少改变

符合此类要求的我能想到的就是一些配置类或者其他一些工具库。如[Spring-Data-Redis](https://github.com/spring-projects/spring-data-redis)中[JedisClientConfiguration](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/connection/jedis/JedisClientConfiguration.java)或者[Picasso](http://square.github.io/picasso/)中的[Picasso](https://github.com/square/picasso/blob/master/picasso/src/main/java/com/squareup/picasso3/Picasso.java)中就使用了建造者模式，可以通过以下代码感受一些使用建造者模式创建对象的方式：

```java
//获取JedisClientConfiguration的方式
public JedisClientConfiguration jedisClientConfiguration() {
        
        JedisClientConfiguration.JedisPoolingClientConfigurationBuilder configurationBuilder = (
                JedisClientConfiguration.JedisPoolingClientConfigurationBuilder) JedisClientConfiguration.builder();
        
        GenericObjectPoolConfig GenericObjectPoolConfig = new GenericObjectPoolConfig();
        GenericObjectPoolConfig.setMaxIdle(1000);
        GenericObjectPoolConfig.setMaxTotal(100);
        GenericObjectPoolConfig.setMinIdle(100);
        
        return configurationBuilder
                .poolConfig(GenericObjectPoolConfig)
                .and()
                .usePooling()//显式指明使用pool
                .build();
}

//Picasso的使用方式
Picasso.get().load("http://i.imgur.com/DvpvklR.png").into(imageView);
```

使用建造者模式的一般步骤或思想

①最终对象一般都比较复杂，构建过程中可能需要其他对象；

②一般将必要参数设置在构建builder的方法中，再由build为最终对象设置非必须参数

③最后由build\(\)方法返回最终对象



**实例1**





