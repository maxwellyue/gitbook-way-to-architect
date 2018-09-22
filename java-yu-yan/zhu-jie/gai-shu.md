# Annotation

## 什么是注解

注解（也被称为元数据）为我们在代码中添加信息提供了一种形式化的方法，使我们可以在稍后某个时刻非常方便地使用这些元数据。也就是说，**我们可以为某个类/某个方法/某个字段/某个构造器等通过注解的方式，增加一些额外信息，然后利用Annotations API来处理这些信息。**

Java中内置了3个标准注解：

* **@Override**：表示当前的方法定义将覆盖父类中的方法。如果你不小心拼写错误，或者方法签名与父类中的方法不对应，编译器就会发出错误提示。该注解是可选的，并不强制。
* **@Deprecated**：如果在程序中使用该注解标注的类/方法等，那么编译器会发出警告信息。
* **@SuppressWarnings**：用来关闭不当的编译器警告信息。

如果我们要自定义自己的注解，一般有两个步骤：**定义注解，然后编写注解处理器**。

## 定义注解

#### 元注解

Java提供了5种元注解，专门负责新注解的创建工作，其中@Repeatable是Java 1.8新增的元注解，如下表所示：

| 元注解 | 说明 |
| :--- | :--- |
| @Target | 表示该注解可以用于什么地方，可选值为：<br>CONSTRUCTOR(构造器)<br>FIELD(成员变量)<br>LOCAL_VARIABLE(局部变量)<br>METHOD(方法)<br>PACKAGE(包)<br>PARAMETER(参数)<br>TYPE(类或接口或enum或其他注解) |
|@Retention  | 表示需要在什么级别保存该注解信息，可选值为：<br>SOURCE：注解仅保留在源代码（.java文件）中，将会被编译器丢弃<br>CLASS：注解在class文件（.class文件）中可用，但会被VM丢弃<br>RUNTIME：VM将在运行期也保留注解，因此可以通过反射机制读取注解的信息 |
|@Documented  | 将此注解包含在javadoc中 |
|@Inherited |允许子类继承父类中的注解<br>即假如类A添加了某注解，且该注解为@Inherited的，则类A的子类也会自动添加该注解|
|@Repeatable|表示该注解在同一元素上，可以重复使用<br>假设某个注解的含义为标识某个类为|

#### 注解的属性

注解的属性也叫做成员变量。注解只有成员变量，没有方法。注解的成员变量在注解的定义中以“无形参的方法”形式来声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型。

使用时，采用“_@注解名(属性1=value,属性2=value2)_”的形式对属性进行赋值，如果某注解只有一个属性，在使用时，可以直接使用“_@注解名(value)_”的形式，而不必显式指明属性名。

此外，可以使用default关键字来为属性设置默认值。没有默认值的属性，在使用时，必须指定值，有默认值的属性，则可以不设置，使用默认值。


```java
//定义MyAnnotation注解，它包含id和msg两个属性
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyAnnotation {
    int id() default 0;
    String msg();
}

//使用：“属性=属性值”的形式
@MyAnnotation(id=5,msg="hello")
public class Xxxxx {}

//使用：“属性=属性值”的形式，因为id有默认值，如果不关心，可以省略
@MyAnnotation(msg="hello")
public class Xxxxx {}

```

#### 注解的继承
TODO


## 编写注解处理器
当我们为类/方法/变量等添加了一个注解后，相当于给它们增加了一个注释。但是增加注释往往不是我们的目的，我们还要根据这个注释来对他们做一些处理。这就是注解处理器。

要编写注解处理器，就必须知道哪些元素（类还是方法还是变量等）上添加了什么注解，这些信息需要使用反射API来获取。获取之后，还要根据注解信息，为元素做一些处理，这个处理过程，也需要使用反射API来完成。

与注解相关的反射API如下，只要注解可以出现的元素（Class/Method/Constructor等），都可以通过对应的下述方法来获取：

```java
//
public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {}

 public <A extends Annotation> A getAnnotation(Class<A> annotationClass) {}

public Annotation[] getAnnotations() {}

```















## 参考

《Java编程思想》
[秒懂，Java 注解 （Annotation）你可以这样学
](https://blog.csdn.net/briblue/article/details/73824058)




