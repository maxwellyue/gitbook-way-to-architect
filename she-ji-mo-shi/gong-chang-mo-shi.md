# 工厂模式

---

工厂模式：提供了一种创建对象的最佳方式，在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。

工厂模式分三种：简单工厂模式、工厂方法模式、抽象工厂模式。

## 简单工厂

---

现在假如有接口A，接口A的实现类AImpl，在客户端创建AImpl对象的时候，我们一般是这么写：

```java
A a = new AImpl();
```

看上去没什么问题，但是仔细想想，客户端不但知道了接口，还知道了具体的实现。而接口存在的意义是让客户端无需知道具体的实现类。但现在如果不用`new AImpl()`的方式，客户端是没有办法来接口A的一个具体对象的。

简单工厂就是用来解决此类问题的：

假如现在有接口A，接口A的两个实现类A1Impl、A2Impl，要想获取接口A实现的对象，新建一个工厂类：

```java
public class Factory {

    private Factory(){}

    public static A createA(int condition) {
        A instance = null;

        if (condition == 1) {
            instance = new A1Impl();
        } else if (condition == 2) {
            instance = new A2Impl();
        }
        return instance;
    }
}
```

客户端在创建接口A对象的时候，根据需求去创建：

```
//获取A1Impl1实例
A instance = Factory.createA(1);

//获取A2Impl实例
A instance = Factory.createA(2);
```

注意点：

①工厂类通常会私有化构造器，并将创建对象的方法写为static方法，客户端使用时相当于在使用一个工具类。

②由于客户端在使用时，由于需要传入选择条件（上面代码中的condition），其实也在一定程度上暴露了实现细节。

③假如又增加了一个接口A的实现类A3Impl，就要修改工厂类了（也可以通过配置文件 + 反射的方式，规避这个问题）。

## 工厂方法模式

---

##  







