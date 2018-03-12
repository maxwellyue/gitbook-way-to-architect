以下程序的输出结果是什么：

```
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

```
ExceptionA
```



