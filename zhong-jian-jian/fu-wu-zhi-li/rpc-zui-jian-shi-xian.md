# RPC最简实现

现在，假如有一个服务：HelloService

```java
public interface HelloService {
    String sayHello(String name);
}
```

 其实现为HelloServiceImpl：

```java
public class HelloServiceImpl  implements HelloService {
    @Override
    public String sayHello(String name) {
        return name + " say hello";
    }
}
```

 现在，编写一个Rpc工具类，提供暴露服务的方法以及调用远程服务的方法：

```java
public class RpcManager {

    /**
     *
     * 提供服务
     *
     *
     * @param service 向外暴露的服务
     * @param port 端口
     * @throws Exception
     */
    public static void exportService(final Object service, int port) throws Exception {
        //创建服务端socket
        ServerSocket serverSocket = new ServerSocket(port);
        while (true) {
            //监听端口
            final Socket socket = serverSocket.accept();
            new Thread(new Runnable() {
                @Override
                public void run() {
                    ObjectInputStream reader = null;
                    ObjectOutputStream writer = null;
                    try {
                        reader = new ObjectInputStream(socket.getInputStream());
                        String methodName = reader.readUTF();
                        Class[] argumentsType = (Class[]) reader.readObject();
                        Object[] arguments = (Object[]) reader.readObject();
                        Method method = service.getClass().getMethod(methodName, argumentsType);
                        //调用方法
                        Object result = method.invoke(service, arguments);
                        writer = new ObjectOutputStream(socket.getOutputStream());
                        //返回结果
                        writer.writeObject(result);
                    } catch (Exception e) {
                        if (null != writer) {
                            try {
                                writer.writeObject(e);
                            } catch (IOException e1) {
                                e1.printStackTrace();
                            }
                        }
                    } finally {
                        if (null != writer) {
                            try {
                                writer.close();
                            } catch (IOException e) {
                                e.printStackTrace();
                            }
                        }
                        if (null != reader) {
                            try {
                                reader.close();
                            } catch (IOException e) {
                                e.printStackTrace();
                            }
                        }
                    }
                }
            }).start();
        }

    }


    /**
     * 调用远程服务
     *
     * @param interfaceClass 远程服务的接口
     * @param host 远程服务的host
     * @param port 远程服务的port
     * @param <T>
     * @return
     */
    public static <T> T referenceService(Class<T> interfaceClass, final String host, final int port) {

        return (T) Proxy.newProxyInstance(interfaceClass.getClassLoader(), new Class[]{interfaceClass}, new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                Socket socket = new Socket(host, port);
                ObjectOutputStream writer = null;
                ObjectInputStream reader = null;
                try {
                    writer = new ObjectOutputStream(socket.getOutputStream());
                    writer.writeUTF(method.getName());
                    writer.writeObject(method.getParameterTypes());
                    writer.writeObject(args);
                    reader = new ObjectInputStream(socket.getInputStream());
                    return reader.readObject();
                } finally {
                    if (null != writer) {
                        writer.close();
                    }
                    if (null != reader) {
                        reader.close();
                    }
                }
            }
        });
    }
}
```

 现在，将HelloService暴露出去：

```java
public class Provider {

    public static void main(String[] args) throws Exception {
        final HelloService helloService = new HelloServiceImpl();
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    RpcManager.exportService(helloService, 20880);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
```

 最后，调用远程服务：

```java
public class Consumer {
    public static void main(String[] args) {
        //调用远程服务
        HelloService helloService = RpcManager.referenceService(HelloService.class, "127.0.0.1", 20880);
        System.out.println(helloService.sayHello("maxwell"));
    }
}
//输出如下
maxwell say hello
```

## 参考

[RPC框架几行代码就够了](http://javatar.iteye.com/blog/1123915)：代码来源

