# 多catch的执行情况

以下程序的输出结果是什么：

```text
public class ExceptionA extends Exception {

}

public class ExceptionB extends ExceptionA {

}

public class ExceptionTest {

    public static void main(String[] args){
        try{
            throw new ExceptionB();
        }catch (ExceptionA e){
            System.out.println("ExceptionA");
        }catch (Exception e){
            System.out.println("Exception");
        }
    }
}
```

输出结果为：

```text
ExceptionA
```

**对于try里面发生的异常，它会根据发生的异常和catch里面的进行匹配：按照catch块从上往下匹配，当它匹配某一个catch块的时候，就直接进入到这个catch块里面去了，而忽略后面所有的catch块。**

**另外：在写异常处理的时候，一定要把异常范围小的放在前面，范围大的放在后面，即如果多个catch块中的异常出现继承关系，父类异常catch块放在下面（否则，连编译都无法通过）。**

也就是说，下面的代码，无法通过编译：

```text
public class ExceptionA extends Exception {

}

public class ExceptionB extends ExceptionA {

}

public class ExceptionTest {

    public static void main(String[] args){
        try{
            throw new ExceptionB();
        }catch (ExceptionA e){
            System.out.println("ExceptionA");
        }catch (ExceptionB e){
            System.out.println("ExceptionB");
        }catch(Exception e){
            System.out.println("Exception");
        }
    }
}
```

