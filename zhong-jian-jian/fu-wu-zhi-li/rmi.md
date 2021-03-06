# Java RMI

### Java RMI 是什么

Java RMI 用于不同虚拟机之间的通信，这些虚拟机可以在不同的主机上、也可以在同一个主机上；一个虚拟机中的对象调用另一个虚拟上中的对象的方法，只不过是允许被远程调用的对象要通过一些标志加以标识。

在`RMI`中的核心是远程对象（remote object），除了对象本身所在的虚拟机，其他虚拟机也可以调用此对象的方法，而且这些虚拟机可以不在同一个主机上。每个远程对象都要实现一个或者多个远程接口来标识自己，声明了可以被外部系统或者应用调用的方法（当然也有一些方法是不想让人访问的）。

### Java RMI 的通信模型 {#11-rmi的通信模型}

从方法调用角度来看，`RMI`要解决的问题，是让客户端对远程方法的调用可以相当于对本地方法的调用而屏蔽其中关于远程通信的内容，即使在远程上，也和在本地上是一样的。

从客户端-服务器模型来看，客户端程序直接调用服务端，两者之间是通过`JRMP`（ [Java Remote Method Protocol](https://en.wikipedia.org/wiki/Java_Remote_Method_Protocol)）协议通信，这个协议类似于HTTP协议，规定了客户端和服务端通信要满足的规范。

但是实际上，客户端只与代表远程主机中对象的`Stub`对象进行通信，丝毫不知道`Server`的存在。客户端只是调用`Stub`对象中的本地方法，`Stub`对象是一个本地对象，它实现了远程对象向外暴露的接口，也就是说它的方法和远程对象暴露的方法的签名是相同的。客户端认为它是调用远程对象的方法，实际上是调用`Stub`对象中的方法。**可以理解为`Stub`对象是远程对象在本地的一个代理**，当客户端调用方法的时候，`Stub`对象会将调用通过网络传递给远程对象。

在`java 1.2`之前，与`Stub`对象直接对话的是`Skeleton`对象，在`Stub`对象将调用传递给`Skeleton`的过程中，其实这个过程是通过`JRMP`协议实现转化的，通过这个协议将调用从一个虚拟机转到另一个虚拟机。在`Java 1.2`之后，与`Stub`对象直接对话的是`Server`程序，不再是`Skeleton`对象了。

所以从逻辑上来看，数据是在`Client`和`Server`之间横向流动的，但是实际上是从`Client`到`Stub`，然后从`Skeleton`到`Server`这样纵向流动的。

![](https://img-blog.csdn.net/20170521103734214?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbG15ODYyNjM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 数据的传输 {#12-重要的问题}

我们都知道在`Java`程序中引用类型（不包括基本类型）的参数传递是按引用传递的，对于在同一个虚拟机中的传递时是没有问题的，因为的参数的引用对应的是同一个内存空间，但是对于分布式系统中，由于对象不再存在于同一个内存空间，虚拟机A的对象引用对于虚拟机B没有任何意义。所以，需要通过序列化手段解决对象传输的问题。

### 远程对象的发现

在调用远程对象的方法之前需要一个远程对象的引用，如何获得这个远程对象的引用在是一个关键的问题。

服务端维护一个**注册表**（Registry），注册表中是&lt;远程对象标识符， 远程对象&gt;的映射关系，客户端通过提供远程对象的标识符访问注册表，来得到远程对象的引用。这个标识符是类似`URL`地址格式的，它要满足的规范如下：

* 格式：`rmi://host:port/name`，rmi是schema，`host`指明注册表运行的主机，`port`表明接收调用的端口，`name`是一个标识该对象的简单名称。
* 主机和端口都是可选的，如果省略主机，则默认运行在本地；如果端口也省略，则默认端口是**1099**；

**示例代码**

编写一个RMI的步骤：

```text
1. 定义一个远程接口，此接口需要继承Remote
2. 开发远程接口的实现类
3. 创建一个server并把远程对象注册到端口
4. 创建一个client查找远程对象，调用远程方法
```

假如应用A有一个UserService，需要将其提供给其他应用调用：

UserSevice接口：需要继承Remote接口，表明是可供远程调用的

```java
public interface UserService extends Remote {

    String getUsername(int userId) throws RemoteException;

    User getById(int userId) throws RemoteException;
}
```

 其中的User对象代码如下：因为要进行网络传输，需要实现Serializable接口对其进行序列化。

```java
public class User implements Serializable {

    private int userId;
    private String username;
    private int age;
    
    ... 省略getter和setter方法 ...
    
    @Override
    public String toString() {
        return "User{" +
                "userId=" + userId +
                ", username='" + username + '\'' +
                ", age=" + age +
                '}';
    }
}
```

编写UserService实现类，让其继承自UnicastRemoteObject，且

```java
public class UserServiceImpl extends UnicastRemoteObject implements UserService{
    //因为父类UnicastRemoteObject的构造器均抛出了RemoteException异常
    //所以，UserServiceImpl必须显示声明一个抛出该异常的构造器
    public UserServiceImpl() throws RemoteException {}

    @Override
    public String getUsername(int userId) throws RemoteException {
        return "username:::" + userId;
    }

    @Override
    public User getById(int userId) throws RemoteException {
        User user = new User();
        user.setUserId(userId);
        user.setUsername("username:::" + userId);
        user.setAge(userId*10);
        return user;
    }
}
```

 将远程对象（UserServiceImpl）绑定到RMI Registry中：

```java
public class Registry {

    public static void main(String[] args) {
        try {
            //创建Registry实例，并在端口1099监听远程请求
            LocateRegistry.createRegistry(1099);
            //把远程对象（service）注册到Registry
            UserService stub = new UserServiceImpl();
            //将存根（stub）与一个名字（RemoteUserService）绑定，以便调用
            Naming.rebind("RemoteUserService", stub);
        } catch(Exception ex) {
            ex.printStackTrace();
        }
    }
}
```

> 上面的 UserServiceImpl也可以不继承UnicastRemoteObject，但是在将其绑定到RMI Registry中的时候，需要使用UnicastRemoteObject将其暴露出来，如下：
>
> `UserService stub =(UserService)UnicastRemoteObject.exportObject(new UserServiceImpl(), 9999);`

其他应用调用

```java
public class Client {

    public static void main(String[] args) {
        try{
            //客户端到RMI registry中寻找
            UserService service = (UserService) Naming.lookup("rmi://127.0.0.1:/RemoteUserService");
            int userId = 10;
            String username = service.getUsername(userId);
            User user = service.getById(userId);

            System.out.println(username);
            System.out.println(user.toString());
        } catch(Exception ex) {
            ex.printStackTrace();
        }
    }
}
//输出如下
username:::10
User{userId=10, username='username:::10', age=100}
```

> 为了使client能够调用远程对象的方法，client必须持有远程对象的存根，为此，Java RMI 提供了registry API 可以允许应用程序把一个名称和远程对象的存根绑定在一起，这样client就可以通过这个绑定的名称很方便的查找到需要调用的远程对象了

**RMI的优缺点**

优势：面向对象的远程服务模型；基于TCP协议上的服务，执行速度快。  
劣势：①不能跨语言；②每个远程对象都要绑定端口，不易维护；③不支持分布式事务JTA，RMI框架对于安全性、事务、可扩展性的支持非常有限。

## 参考

维基百科：[Java远程方法调用](https://zh.wikipedia.org/wiki/Java%E8%BF%9C%E7%A8%8B%E6%96%B9%E6%B3%95%E8%B0%83%E7%94%A8)

[Java RMI初探](https://crowhawk.github.io/2017/07/17/RMI/)

[Java RMI之HelloWorld篇](http://blog.51cto.com/lavasoft/91679)

[浅谈JAVA常用分布式实现方式及优缺点](http://www.aboutyun.com/thread-7070-1-1.html)

[Java RMI原理与使用](https://blog.csdn.net/suifeng3051/article/details/48469523)：过程描述的很详细

[从懵逼到恍然大悟之Java中RMI的使用](https://blog.csdn.net/lmy86263/article/details/72594760)：文字部分来源

