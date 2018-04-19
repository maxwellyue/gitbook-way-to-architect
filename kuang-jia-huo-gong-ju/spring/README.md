# Spring

## 动态代理的两种方式及区别？

**动态代理有两种方式：**

* JDK动态代理：在运行时根据类的接口生成新的实现类（通过反射），让新的实现类对已有对象进行代理。
* cglib：对指定的目标类生成一个子类，并覆盖其中方法实现增强，但因为采用的是继承，所以不能对final修饰的类进行代理。

**两种代理方式对比：**

* JDK动态代理**：**某个类必须有实现的接口，而生成的代理类也只能代理某个类接口定义的方法。
* CGLIB用继承的方式实现相关的代理类，底层是使用字节码处理框架ASM来转换字节码并生成新的类。

**什么情况下会用哪种方式实现动态代理**

1. 如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP ，但通过配置来强制使用CGLIB实现AOP
2. 如果目标对象没有实现了接口，必须采用CGLIB库，spring会自动在JDK动态代理和CGLIB之间转换

**如何强制使用CGLIB实现AOP？ **

1. 添加CGLIB库，SPRING\_HOME/cglib/\*.jar
2. 在spring配置文件中加入`<aop:aspectj-autoproxy proxy-target-class="true"/>`

## JDK动态代理编写步骤

### 步骤

①创建被代理的类和接口  
②创建一个实现InvocationHandler的类（代理类），实现invoke方法  
③调用Proxy的静态方法`createProxyInstance(Class clazz)`，创建一个代理类  
④通过代理调用方法

### 完整示例

首先，接口UserService和该接口的实现类UserServiceImpl：

```text
public interface UserService {

    User getUser(String userId);

    void updateUser(User user);

    void deleteUser(String userId);

    void addUser(User user);

}

public class UserServiceImpl implements UserService {
    @Override
    public User getUser(String userId) {

        System.out.println("---getUser-----");
        return null;
    }

    @Override
    public void updateUser(User user) {

        System.out.println("---updateUser-----");
    }

    @Override
    public void deleteUser(String userId) {

        System.out.println("---deleteUser-----");
    }

    @Override
    public void addUser(User user) {

        System.out.println("---addUser-----");
    }
}
```

之后，创建一个代理类（UserServiceImpl）的调用处理器，起个名字叫：SecurityHandler，并让它实现InvocationHandler接口（该接口是JDK1.3之后自带的）

```text
public class SecurityHandler  implements InvocationHandler {
    //使用一个通用的对象
    private Object targetObject;

    public Object createProxyInstance(Object object){
        this.targetObject = targetObject;
        //根据目标接口生成代理
        //第一个参数是代理对象的classloader，第二个是代理对象实现的接口，第三个参数可以理解为回调。
        return Proxy.newProxyInstance(targetObject.getClass().getClassLoader(),
                targetObject.getClass().getInterfaces(),
                this);

    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //要增加的额外操作：本例中就是检查安全
        checkSecurity();
        //调用目标方法
        Object ret = method.invoke(targetObject, args);
        return ret;
    }

    /**
     * 校验安全性的方法
     */
    private void checkSecurity(){
        System.out.println("----checkSecurity-----");
    }
}
```

好了，现在就可以调用了：

```text
public static void main(String[] args) {
        SecurityHandler handler = new SecurityHandler();
        UserService userService = (UserService)handler.createProxyInstance(new UserServiceImpl());
        userService.deleteUser("user的id");
}
```

内容来源：

[理解Spring AOP](https://www.jianshu.com/p/58d6901e2cbd)

[Spring AOP 之JDK动态代理和CGLIB代理的区别](http://youyu4.iteye.com/blog/2348704)

