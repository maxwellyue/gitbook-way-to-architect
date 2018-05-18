# 单例模式

①静态内部类的方式

```java
public class Singleton { 
    private static class SingletonHolder { 
        private static final Singleton INSTANCE = new Singleton(); 
    } 
    private Singleton (){} 
    public static final Singleton getInstance() { 
        return SingletonHolder.INSTANCE;
    } 
}
```

这种写法，线程安全的保证机制是：JVM只会初始化类一次。

②饿汉式

```java
public class Singleton{
    //类加载时就初始化
    private static final Singleton instance = new Singleton();

    private Singleton(){}

    public static Singleton getInstance(){
        return instance;
    }
}
```

③枚举

```java
public enum EasySingleton{
    INSTANCE;
}
```

