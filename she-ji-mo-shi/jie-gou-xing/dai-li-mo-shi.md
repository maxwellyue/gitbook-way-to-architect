# 代理模式

---

给某个对象提供一个代理对象，并由代理对象控制对于原对象的访问，即客户不直接操控原对象，而是通过代理对象间接地操控原对象。

代理模式的本质是控制对象访问，可以在具体的目标对象前后，附加很多操作，从而进行功能扩展。

代理模式的的实现方式有两种：

* 静态代理  
  代理类是在编译时就实现好的。也就是说`Java`编译完成后代理类是一个实际的`class`文件。

* 动态代理  
  代理类是在运行时生成的。也就是说`Java`编译完之后并没有实际的`class`文件，而是在运行时动态生成的类字节码，并加载到`JVM`中。

**静态代理代码示例
**假如有接口`UserService`和该接口的实现类`UserServiceImpl`
```java
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
现在，假如要对`UserService`中的所有方法进行安全检查(即增加一个方法，假设为`checkSecurity()`)，则实现类`UserServiceImpl`的代码需要修该为这样：
```java
public class UserServiceImpl implements UserService {
    @Override
    public User getUser(String userId) {
        checkSecurity();
        System.out.println("---getUser-----");
        return null;
    }

    @Override
    public void updateUser(User user) {
        checkSecurity();
        System.out.println("---updateUser-----");
    }

    @Override
    public void deleteUser(String userId) {
        checkSecurity();
        System.out.println("---deleteUser-----");
    }

    @Override
    public void addUser(User user) {
        checkSecurity();
        System.out.println("---addUser-----");
    }

    /**
     * 校验安全性的方法
     */
    private void checkSecurity(){
        System.out.println("----checkSecurity-----");
    }
}
```
假如以后还要对每个方法进行日志记录、缓存等，都需要对其中的每个方法添加相应的逻辑代码，给代码维护带来很大麻烦。

现在使用动态代理来解决该问题：为`UserServiceImpl`添加一个代理类`UserServiceImplProxy`；同时让这个代理类`UserServiceImplProxy`实现`UserService`的接口：
```java
public class UserServiceImplProxy implements UserService {
    
    private UserService userService;

    //构造器
    public UserServiceImplProxy(UserService userService){
        this.userService = userService;
    }

    @Override
    public User getUser(String userId) {
        checkSecurity();
        return userService.getUser(userId);
    }

    @Override
    public void updateUser(User user) {
        checkSecurity();
        userService.updateUser(user);
    }

    @Override
    public void deleteUser(String userId) {
        checkSecurity();
        userService.deleteUser(userId);
    }

    @Override
    public void addUser(User user) {
        checkSecurity();
        userService.addUser(user);
    }

    /**
     * 校验安全性的方法
     */
    private void checkSecurity(){
        System.out.println("----checkSecurity-----");
    }
}
```
`UserServiceImpl`中还是原来的方法：
```java
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
从上面的代码可以看出，使用了静态代理模式之后，与`User`紧密相关的增删改查方法都在`UserServiceImpl`中，类似检查安全或日志记录等非`User`相关操作都在代理类中实现。
当要实现`User`相关的操作时，不使用`UserServiceImpl`，而是通过代理`UserServiceImpProxy`进行：
```java
public static void main(String[] args) {
        UserService userService = new UserServiceImplProxy(new UserServiceImpl());
        userService.deleteUser("user的id");
}
```
静态代理虽然让`User`相关操作在一个类中（`UserServiceImpl`），非`User`相关操作在另一个类中的（即一定程度提高代码内聚性），但需要为每个目标类（上面例子中的`UserServiceImpl`）写一个代理类。假如我们有很多目标类都要实现同样的安全检查这类的操作，就要创建多个代理类，并且写很多重复的代码，显然我们需要更简便的方式。

**动态代理示例（使用`JDK`的方式）**
还是以上面的静态代理为基础，继续看动态代理是怎么解决上述问题的（动态代理的实现有多种：`JDK` 自带的动态处理、`CGLIB`、`Javassist `或者 `ASM `库，下文中演示的是`JDK` 自带的动态代理实现）。

首先，接口`UserService`和该接口的实现类`UserServiceImpl`与演示静态代理时一致：
```java
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

之后，创建一个代理类（`UserServiceImpl `）的调用处理器，起个名字叫：`SecurityHandler`，并让它实现`InvocationHandler`接口（该接口是`JDK1.3`之后自带的）
```java
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
好了，现在就可以调用了，调用时与静态代理类似：
```java
public static void main(String[] args) {
		SecurityHandler handler = new SecurityHandler();
		UserService userService = (UserService)handler.createProxyInstance(new UserServiceImpl());
		userService.deleteUser("user的id");
}
```

#### 参考
----
[理解Spring AOP](https://www.jianshu.com/p/58d6901e2cbd)
