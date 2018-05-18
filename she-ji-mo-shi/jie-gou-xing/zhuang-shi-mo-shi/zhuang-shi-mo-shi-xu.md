# 装饰模式续

上面的实例代码中，我们新增的功能（增加滚动条和黑色边框）都是加入到被装饰的原始类即Component的方法中，客户端并不会调用这些方法：

```java
//滚动条装饰类
class ScrollBarDecorator extends  ComponentDecorator{

       public ScrollBarDecorator(Component  component){
              super(component);
       }

       public void display(){
              this.setScrollBar();
              super.display();
       }

       public void setScrollBar(){
              System.out.println("为构件增加滚动条！");
       }
}
```

即经过装饰后，被装饰类只是在原有方法的内部去增强了原有方法，其实这类似于AOP的功能增强。但假如要在被装饰类中添加一个新的方法，且该方法与被装饰类中的原有方法没有任何关系，比如OA系统中，采购单\(PurchaseRequest\)和请假条都有display\(\)方法，现在要对他们添加审批和删除功能，那么使用装饰模式的代码结构如下：![](../../../.gitbook/assets/ping-mu-kuai-zhao-20180508-xia-wu-9.48.27.png)

其中，审批装饰类代码具体如下：

```java
//审批装饰类
public class Approver extends Decorator{

      public Approver(Document document){
             super(document);
      }

      public void approve(){
             System.out.println("审批！");
      }
}
```

这个审批功能与原始类的显示方法（`display()`）没有关系，那么客户在使用`Component c = new PurchaseRequest()`定义一个采购单时，由于`Component`中只有`display()`方法，新增的审批功能无法被调用。所以，客户端就不能使用这种面向抽象的编程方式，而必须使用`PurchaseRequest request = new PurchaseRequet()`这种方式。

**装饰模式的透明与半透明**

构建基础构件例子中演示的装饰模式，称之为透明装饰模式。这种方式可以让客户端透明地使用装饰之前的对象和装饰之后的对象，无须关心它们的区别，此外，还可以对一个已装饰过的对象进行多次装饰，得到更为复杂、功能更为强大的对象。

OA系统例子中演示的装饰模式，称之为半透明装饰模式。半透明装饰模式可以给系统带来更多的灵活性，设计相对简单，使用起来也非常方便；但是其最大的缺点在于不能实现对同一个对象的多次装饰，而且客户端需要有区别地对待装饰之前的对象和装饰之后的对象。

> 为什么半透明装饰模式不能实现对同一个对象的多次装饰
>
> 如果进行多次装饰，最终功能只会增加一个，因为客户端至多能调用到最外层的那个装饰功能。

**JDK中的装饰模式**

JDK中最典型的装饰模式的应用就是I/O流，看如下代码：

```java
try (DataInputStream inputStream = new DataInputStream(new BufferedInputStream(new FileInputStream("test.txt")))){

    //do something with inputStream
    ......

} catch (FileNotFoundException e) {
    e.printStackTrace();
}catch (IOException e){
    e.printStackTrace();
}
```

这里，`FileInputSream`相当于原始的被装饰的类，它经过了`BufferedInputSream`装饰（拥有了buffer功能），又经过`DataInputStream`装饰。

这几个类之间的关系如下图：![](../../../.gitbook/assets/ping-mu-kuai-zhao-20180508-xia-wu-10.14.38.png)

这里的`InputStream`相当于`Component`，`FileInputStream`相当于`ConcreteComponent`，而`FilterInputStream`相当于`Decorator`，`BufferedInputStream`和`DataInputStream`相当于`ConcreteDecorator`。

输出流OutputStream与之类似。

## 参考

[扩展系统功能--装饰模式](https://quanke.gitbooks.io/design-pattern-java/content/装饰模式-Decorator%20Pattern.html)：文字和代码大多来自于此

《研磨设计模式 第22章 装饰模式》：JDK中装饰模式

