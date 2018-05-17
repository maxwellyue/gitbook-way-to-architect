不同的传值场景

ThreadLocal是线程私有变量，每个线程均有一个私有的ThreadLocal变量，为在复杂场景下的传值提供了一种便捷的方式。

比如

```java
ThreadLocal<String> threadLocal = new ThreadLocal<>();
...some codes...
threadLocal.set("value:::init-set");
...some codes...
threadLocal.get();
```

但是ThreadLocal只能解决在某一个线程内的变量set/get，假如现在我们想在子线程中获取父线程中的ThreadLoca变量的值，怎么办？

> 什么是父子线程
>
> 在A线程中，创建并启动了线程B，那么A就是B的父线程，B就是A的子线程；
>
> 对某线程而言，至多有一个父线程，可以有多个子线程。

```java
public static void main(String[] args) {
        //latch仅仅为了控制程序运行顺序，与主题无关
        CountDownLatch latch = new CountDownLatch(1);

        ThreadLocal<String> local = new ThreadLocal<>();
        local.set("value:::value-in-prent");
        System.out.println("[1]" + local.get());

        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("[2]" + local.get());
                local.set("value:::value-in-child");
                System.out.println("[3]" + local.get());
                latch.countDown();
            }
        }).start();

        try {
            latch.await();
            System.out.println("[4]" + local.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }        
}
//输出如下：
[1]value:::value-in-parent
[2]null                   
[3]value:::value-in-child
[4]value:::value-in-parent
```

我们期望在\[2\]处输出`value:::value-in-parent`，显然`ThreadLocal`无法满足需求。JDK中提供了`InheritableThreadLocal`，将上面代码中的`ThreadLocal`替换为`InheritableThreadLocal`，其他内容保持不变：

```java
public static void main(String[] args) {
        //latch仅仅为了控制程序运行顺序，与主题无关
        CountDownLatch latch = new CountDownLatch(1);

        InheritableThreadLocal<String> local = new InheritableThreadLocal<>();
        local.set("value:::value-in-prent");
        System.out.println("[1]" + local.get());

        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("[2]" + local.get());
                local.set("value:::value-in-child");   //-------------------------------------(*)
                System.out.println("[3]" + local.get());
                latch.countDown();
            }
        }).start();

        try {
            latch.await();
            System.out.println("[4]" + local.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }        
}
//输出如下：
[1]value:::value-in-parent
[2]value:::value-in-parent               
[3]value:::value-in-child
[4]value:::value-in-parent
```

可以看到，主线程中的local变量的值被传递到了子线程中，这就解决了无法在子线程中获取父线程ThreadLocal值的问题。此外，上面的代码中标出的\(\*\)位置处，改变了子线程中local值，但父线程中的local值没有改变，这就是说，子线程仅仅是获取了父线程中local值作为自己的初始值，但子线程改变自己的local值，并不会影响父线程的local值。

但是，在实际项目中，我们一般不会直接通过new Thread这种方式使用线程，而是使用线程池，这样就会对线程重复利用。此时，假如向线程池提交多个任务，这些任务会改变所在线程的threadlocal值，但是新提交的任务又必须使用父线程中的值，怎么办？我们还是使用Inheritable来试一下，看它能不能满足需求：

```java
public static void main(String[] args) {
        //latch仅仅为了控制程序运行顺序，与主题无关
        CountDownLatch latch = new CountDownLatch(2);

        InheritableThreadLocal<String> local = new InheritableThreadLocal<>();
        local.set("value:::value-in-parent");
        System.out.println("[1]" + local.get());

        ExecutorService executor = Executors.newFixedThreadPool(1);
        Runnable runnable1 = new Runnable() {
            @Override
            public void run() {
                System.out.println("[2]" + local.get());
                local.set("value:::value-in-runnable1");
                System.out.println("[3]" + local.get());
                latch.countDown();
            }
        };
        Runnable runnable2 = new Runnable() {
            @Override
            public void run() {
                System.out.println("[4]" + local.get());
                local.set("value:::value-in-runnable2");
                System.out.println("[5]" + local.get());
                latch.countDown();
            }
        };
        executor.submit(runnable1);
        try {
            Thread.sleep(1000);//只是为了让两个任务的先后顺序符合测试需求
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        executor.submit(runnable2);

        try {
            latch.await();
            System.out.println("[6]" + local.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
}
//输出如下：
[1]value:::value-in-parent
[2]value:::value-in-parent
[3]value:::value-in-runnable1
[4]value:::value-in-runnable1
[5]value:::value-in-runnable2
[6]value:::value-in-parent
```

我们期望\[4\]处获取的值为`value:::value-in-parent`，但实际却是`value:::value-in-runnable1`，这是因为，我们的线程池中只有一个线程（假设为A），runnable1执行完毕之后，将A线程的local值设为了`value:::value-in-runnable1`，runnable2仍然是在A线程执行，那么\[4\]处就是runnable1改变之后的local值。

该问题的解决方案为阿里开源的[Transmittable ThreadLocal\(TTL\)](https://github.com/alibaba/transmittable-thread-local)，上述问题的解决代码如下：

```java
public static void main(String[] args) {
        //latch仅仅为了控制程序运行顺序，与主题无关
        CountDownLatch latch = new CountDownLatch(2);
        //使用TTL提供的TransmittableThreadLocal
        TransmittableThreadLocal<String> local = new TransmittableThreadLocal<>();
        local.set("value:::value-in-parent");
        System.out.println("[1]" + local.get());

        ExecutorService executor = Executors.newFixedThreadPool(1);

        Runnable runnable1 = new Runnable() {
            @Override
            public void run() {
                System.out.println("[2]" + local.get());
                local.set("value:::value-in-runnable1");
                System.out.println("[3]" + local.get());
                latch.countDown();
            }
        };


        Runnable runnable2 = new Runnable() {
            @Override
            public void run() {
                System.out.println("[4]" + local.get());
                local.set("value:::value-in-runnable2");
                System.out.println("[5]" + local.get());
                latch.countDown();
            }
        };
        //使用TTL中的TtlRunnable对JDK原生Runnable进行包装
        Runnable t1 = TtlRunnable.get(runnable1);
        Runnable t2 = TtlRunnable.get(runnable2);

        executor.submit(t1);

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        executor.submit(t2);

        try {
            latch.await();
            System.out.println("[6]" + local.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }}
//输出如下：
[1]value:::value-in-parent
[2]value:::value-in-parent
[3]value:::value-in-runnable1
[4]value:::value-in-parent
[5]value:::value-in-runnable2
[6]value:::value-in-parent
```

可见，上面的\[4\]处输出的值为父线程中的local值，满足了我们的需求。

InheritableThreadLocal







