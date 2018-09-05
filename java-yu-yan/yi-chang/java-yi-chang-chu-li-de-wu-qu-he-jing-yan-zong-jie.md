**误区1：将异常直接显示在页面或客户端。**

将异常直接打印在客户端的例子屡见不鲜，以 JSP 为例，一旦代码运行出现异常，默认情况下容器将异常堆栈信息直接打印在页面上。其实从客户角度来说，任何异常都没有实际意义，绝大多数的客户也根本看不懂异常信息，软件开发也要尽量避免将异常直接呈现给用户。

**误区2：忽略异常**

如下异常处理只是将异常输出到控制台，没有任何意义。而且这里出现了异常并没有中断程序，进而调用代码继续执行，导致更多的异常。

```java
public void retrieveObjectById(Long id) {
    try {
        //..some code that throws SQLException
    } catch (SQLException ex) {
        //这里的异常打印仅仅是将错误堆栈输出到控制台，而在 Production 环境中，需要将错误堆栈输出到日志。
        ex.printStackTrace();
    }
}
```

解决方案：可以打印日志或者选择向上层抛出异常。

**误区3、将异常包含在循环语句块中**

如下代码所示，异常包含在 for 循环语句块中。

```java
for(int i=0; i<100; i++){
    try{
    
    }catch(XXXException e){
 
    }
}
```

异常处理是非常占用系统资源的。一般情况下都不会犯这样的错误，但是可能有这种情况：类 A 中执行了一段循环，循环中调用了 B 类的方法，B 类中被调用的方法包含 try-catch 这样的语句块。

**误区4、利用 Exception 捕捉所有潜在的异常**

一段方法执行过程中抛出了几个不同类型的异常，为了代码简洁，利用基类 Exception 捕捉所有潜在的异常，如下例所示：

```java
public void retrieveObjectById(Long id){
    try{
        //…抛出 IOException 的代码调用
        //…抛出 SQLException 的代码调用
    }catch(Exception e){
        //这里利用基类 Exception 捕捉的所有潜在的异常，如果多个层次这样捕捉，会丢失原始异常的有效信息
        throw new RuntimeException(“Exception in retieveObjectById”, e);
    }
}
```

可以重构为：

```java
public void retrieveObjectById(Long id){
    try{
        //..some code that throws RuntimeException, IOException, SQLException
    }catch(IOException e){
        //仅仅捕捉 IOException
        throw new RuntimeException(/*指定这里 IOException 对应的错误代码*/code,“Exception in retieveObjectById”, e);
    }catch(SQLException e){
        //仅仅捕捉 SQLException
        throw new RuntimeException(/*指定这里 SQLException 对应的错误代码*/code,“Exception in retieveObjectById”, e);
    }
}
```

**误区5、多层次打印异常**

我们先看一下下面的例子，定义了 2 个类 A 和 B。其中 A 类中调用了 B 类的代码，并且 A 类和 B 类中都捕捉打印了异常。

```java
public class A {
    private static Logger logger = LoggerFactory.getLogger(A.class);

    public void process() {
        try {
            B b = new B();
            b.process();
            //other code might cause exception
        } catch (XXXException e) {
            //如果 B 类 process 方法抛出异常，异常会在 B 类中被打印，在这里也会被打印，从而会打印 2 次
            logger.error(e);
            throw new RuntimeException(/* 错误代码 */ errorCode, /*异常信息*/msg, e);
        }
    }
}

public class B {
    private static Logger logger = LoggerFactory.getLogger(B.class);

    public void process() {
        try {
            //可能抛出异常的代码
        } catch (XXXException e) {
            logger.error(e);
            throw new RuntimeException(/* 错误代码 */ errorCode, /*异常信息*/msg, e);
        }
    }
}
```

同一段异常会被打印 2 次。如果层次再复杂一点，不去考虑打印日志消耗的系统性能，仅仅在异常日志中去定位异常具体的问题已经够头疼的了。

其实打印日志只需要在代码的最外层捕捉打印就可以了，异常打印也可以写成 AOP，织入到框架的最外层。

**误区6、异常包含的信息不能充分定位问题**

异常不仅要能够让开发人员知道哪里出了问题，更多时候开发人员还需要知道是什么原因导致的问题，我们知道 java .lang.Exception 有字符串类型参数的构造方法，这个字符串可以自定义成通俗易懂的提示信息。

简单的自定义信息开发人员只能知道哪里出现了异常，但是很多的情况下，开发人员更需要知道是什么参数导致了这样的异常。这个时候我们就需要将方法调用的参数信息追加到自定义信息中。下例只列举了一个参数的情况，多个参数的情况下，可以单独写一个工具类组织这样的字符串。

```java
public void retieveObjectById(Long id){
    try{
        //..some code that throws SQLException
   }catch(SQLException ex){
        //将参数信息添加到异常信息中
        throw new RuntimeException(“Exception in retieveObjectById with Object Id :”+ id, ex);
   }
}
```





### 内容来源

[Java 异常处理的误区和经验总结](https://www.ibm.com/developerworks/cn/java/j-lo-exception-misdirection/)

  






