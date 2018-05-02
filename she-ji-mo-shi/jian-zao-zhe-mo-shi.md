# Builder模式

---

**一般在什么场景下使用建造者模式？**

①需要的这个对象非常复杂，即它包含非常多的属性

②对象构建出来就保持不变或很少改变

符合此类要求的我能想到的就是一些配置类。如[Spring-Data-Redis](https://github.com/spring-projects/spring-data-redis)中[JedisClientConfiguration](https://github.com/spring-projects/spring-data-redis/blob/master/src/main/java/org/springframework/data/redis/connection/jedis/JedisClientConfiguration.java)或者[Picasso](http://square.github.io/picasso/)中的[Picasso](https://github.com/square/picasso/blob/master/picasso/src/main/java/com/squareup/picasso3/Picasso.java)中就使用了建造者模式，可以通过以下代码感受一些使用建造者模式创建对象的方式：

```
JedisClientConfiguration.JedisPoolingClientConfigurationBuilder
```





