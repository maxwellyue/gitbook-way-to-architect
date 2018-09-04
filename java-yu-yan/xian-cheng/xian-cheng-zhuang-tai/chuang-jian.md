# 创建线程

线程的创建有以下3种方式：

* **继承Thread类** ①定义Thread类的子类，并重写该类的run方法，该run方法的方法体就代表了线程要完成的任务。因此把run\(\)方法称为执行体。 ②创建Thread子类的实例，即创建了线程对象。 ③调用线程对象的start\(\)方法来启动该线程。
* **实现Runnable接口** ①定义runnable接口的实现类，并重写该接口的run\(\)方法，该run\(\)方法的方法体同样是该线程的线程执行体。 ②创建 Runnable实现类的实例，并依此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象。 ③调用线程对象的start\(\)方法来启动该线程。
* **实现Callable接口**（jdk1.5新增，在java.util.concurrent包） ①创建Callable接口的实现类，并实现call\(\)方法，该call\(\)方法将作为线程执行体，并且有返回值。 ②创建Callable实现类的实例，使用FutureTask类来包装Callable对象，该FutureTask对象封装了该Callable对象的call\(\)方法的返回值。 ③使用FutureTask对象作为Thread对象的target创建并启动新线程。 ④调用FutureTask对象的get\(\)方法来获得子线程执行结束后的返回值。

**示例**

```java
public class ThreadLearning {

//------------------------继承Thread类-----------------------
    //匿名内部类的形式
    @org.junit.Test
    public void generateThread_1() {
        Thread thread = new Thread() {
            @Override
            public void run() {
                //do something
                System.out.println("1");
            }
        };
        thread.start();
    }

    //正常形式
    @org.junit.Test
    public void generateThread_2() {
        Thread1 thread1 = new Thread1();
        thread1.start();
    }

    class Thread1 extends Thread{
        @Override
        public void run() {
            //do something
            System.out.println("2");
        }
    }


//------------------------实现Runnable接口-----------------------
    //匿名内部类的形式
    @org.junit.Test
    public void generateThread_3() {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                //do something
                System.out.println("1");
            }
        });
        thread.start();
    }

    //正常形式
    @org.junit.Test
    public void generateThread_4() {
        Thread2 thread2 = new Thread2();
        Thread thread = new Thread(thread2);
        thread.start();
    }

    class Thread2 implements Runnable{
        @Override
        public void run() {
            //do something
            System.out.println("2");
        }
    }


//------------------------实现Callable接口-----------------------
    @org.junit.Test
    public void generateThread_5() {
        Thread3 thread3 = new Thread3();
        FutureTask<String> futureTask = new FutureTask<String>(thread3);
        new Thread(futureTask).start();
        try {
            String s = futureTask.get();
            System.out.println(s);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

    class Thread3 implements Callable<String>{
        @Override
        public String call() throws Exception {
            return "我是实现Callable接口的线程类";
        }
    }
}
```

