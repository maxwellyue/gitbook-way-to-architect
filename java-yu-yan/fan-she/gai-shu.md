## 什么是反射

反射\(Reflection\)是Java 程序开发语言的特征之一，它允许**运行中**的Java程序获取自身的信息，并且可以操作类或对象的内部属性。

Oracle官方对反射的解释是

> Reflection enables Java code to discover information about the fields, methods and constructors of loaded classes, and to use reflected fields, methods, and constructors to operate on their underlying counterparts, within security restrictions.  
> The API accommodates applications that need access to either the public members of a target object \(based on its runtime class\) or the members declared by a given class. It also allows programs to suppress default reflective access control.

通过反射，我们可以在运行时获得程序或程序集中每一个类型的成员和成员的信息。一般而言，程序中对象的类型都是在编译期就确定下来的，而Java反射机制可以动态地创建对象并调用其属性，这样的对象的类型在编译期是未知的。所以我们可以通过反射机制直接创建对象，即使这个对象的类型在编译期是未知的。  
**反射的核心是JVM在运行时才动态加载类或调用方法/访问属性，它不需要事先（写代码的时候或编译期）知道运行对象是谁。**

Java的反射框架主要提供以下功能：**关键词是运行时（而非编译期）**

* 1.在运行时判断任意一个对象所属的类；
* 2.在运行时构造任意一个类的对象；
* 3.在运行时判断任意一个类所具有的成员变量和方法
* 4.在运行时调用任意一个对象的方法；

## 应用场景

很多人都认为反射在实际的Java开发应用中并不广泛，其实不然。当我们在使用IDE\(如Eclipse，IDEA\)时，当我们输入一个对象或类并想调用它的属性或方法时，一按点号，编译器就会自动列出它的属性或方法，这里就会用到反射。

**反射最重要的用途就是开发各种通用框架。**

很多框架（比如Spring）都是配置化的（比如通过XML文件配置各种bean），为了保证框架的通用性，它们可能需要根据配置文件加载不同的对象或类，调用不同的方法，这个时候就必须用到反射——运行时动态加载需要加载的对象。

对与框架开发人员来说，反射虽小但作用非常大，它是各种容器实现的核心。而对于一般的开发者来说，不深入框架开发则用反射用的就会少一点，不过了解一下框架的底层机制有助于丰富自己的编程思想，也是很有益的。

## 基本用法

反射可以用于判断任意对象所属的类，获得Class对象，构造任意一个对象以及调用一个对象。这里我们介绍一下基本反射功能的实现\(反射相关的类一般都在java.lang.relfect包里\)。

为了操作方便，Java除抽象出Class来表示类之外，还提供了Method/Field/Constructor来分别表示方法/字段/构造器。

#### **获取Class对象**

\(1\)使用Class类的**forName\(String className\)**静态方法

```java
Class<?> userClass = Class.forName("com.maxwell.learning.common.reflectexample.User");
```

\(2\)直接获取某一个对象的class：**类名.class**

```java
Class<String> stringClass = String.class;
Class<HashMap> hashMapClass = HashMap.class;
```

\(3\)调用某个对象的**getClass\(\)**方法

```java
User user = new User();
Class<? extends User> userClass = user.getClass();

String s = new String("abc");
Class<? extends String> aClass = s.getClass();
```

#### 判断是否为某个类的实例

一般地，我们用instanceof关键字来判断是否为某个类的实例。同时我们也可以借助反射中Class对象的isInstance\(\)方法来判断是否为某个类的实例，它是一个Native方法：

```java
public native boolean isInstance(Object obj);
```

用法示例

```java
static class Animal { }
static class Cat extends Animal { }

@Test
public void testInstanceOf() {
    Cat cat = new Cat();
    boolean isInstanceOf = cat instanceof Animal;
    boolean isInstance = Animal.class.isInstance(cat);
}
```

#### 创建实例 {#3、创建实例}

（1）使用Class对象的newInstance\(\)方法来创建Class对象对应类的实例

```java
User user = User.class.newInstance();
```

2）先通过Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance\(\)方法来创建实例。这种方法可以用指定的构造器构造类的实例

```java
//假如User类中有如下构造器
public User(String name, int age) {
        this.name = name;
        this.age = age;
}

//则可以这样来
Constructor<User> constructor = User.class.getConstructor(String.class, int.class);
User user = constructor.newInstance("xiaoming", 18);
```

#### 获取类构造器

* getConstructor\(Class&lt;?&gt;... parameterTypes\) ：根据入参类型和个数，获取对应public的构造器
* getDeclaredConstructor\(Class&lt;?&gt;... parameterTypes\) ：根据入参类型和个数，获取对应构造器，不能获取到父类的
* getConstructors\(\)：获取所有public的构造器
* getDeclaredConstructors\(\)：获取所有构造，不能获取到父类的

此外，Constructor类有一个newInstance方法可以创建一个对象实例。

```java
public T newInstance(Object ... initargs)
```

此方法可以根据传入的参数（没有则不传）来调用对应的Constructor创建对象实例。

#### 获取类的变量（字段）信息

* getFiled\(String filedName\): 根据字段名获取public成员变量
* getDeclaredField\(String filedName\)：根据字段名获取已声明的变量，但不能得到其父类的变量
* getFileds\(\)：获取所有的pulbic变量
* getDeclaredFields\(\)：获取所有的已声明的变量，但不能得到其父类的public变量

如果是私有变量，必须设置field.setAccessible\(true\)之后才可以访问，否则将抛出java.lang.NoSuchFieldException异常。

```java
//假如有如下类
public class User {

    private String name;

    private int age;
}
//下面将会报错：java.lang.NoSuchFieldException
Field nameFiled = user.getClass().getField("name");

//解决方法：field.setAccessible(true)
User user = new User("xiaoming", 18);
Class<? extends User> userClass = user.getClass();
Field[] fields = userClass.getDeclaredFields();
for (Field field : fields){
    field.setAccessible(true);
    System.out.println(String.format("属性[%s], 值[%s]", field.getName(), field.get(user)));
}
//输出如下
属性[name], 值[xiaoming]
属性[age], 值[18]
```

#### 获取方法 {#4、获取方法}

获取某个Class对象的方法集合，主要有以下几个方法：

* getDeclaredMethods\(\)：返回所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法
* getMethods\(\)：返回所有public方法，包括其继承类的公用方法
* getMethod\(String name, Class&lt;?&gt;... parameterTypes\)：第一个参数为方法名称，后面的参数为方法的参数对应的Class对象
* getDeclaredMethod\(String name, Class&lt;?&gt;... parameterTypes\)：可以获取到私有的，但不能获取到父类的

#### 调用方法 {#7、调用方法}

当我们从类中获取了一个方法后，我们就可以用invoke\(\)方法来调用这个方法：

```java
public Object invoke(Object obj, Object... args)
```

使用示例

```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
}
@Test
public void testInvokeMethod() throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException, NoSuchFieldException {
    Calculator calculator = new Calculator();
    Class<? extends Calculator> calculatorClass = calculator.getClass();
    Method method = calculatorClass.getMethod("add", int.class, int.class);
    int res = (int)method.invoke(calculator, 1, 2);
    System.out.println(res);
}
//输出如下
3
```

#### 利用反射创建数组 {#8、利用反射创建数组}

数组在Java里是比较特殊的一种类型，可以使用`java.lang.reflect.Array`来进行创建。

```java
@Test
public void testArray() throws ClassNotFoundException {
    Class<?> clazz = Class.forName("java.lang.String");
    Object array = Array.newInstance(clazz,25);
    Array.set(array,0,"aaa");
    Array.set(array,1,"bbb");
    Array.set(array,2,"ccc");
    Array.set(array,3,"ddd");
    Array.set(array,4,"eee");
    String  res = (String)Array.get(array, 3);
    System.out.println(res);
}
//输出如下
ddd
```

#### 总结

我们可以发现这样的规则：

* **getDeclaredXxxx：获取到自身的方法/变量/构造器等，包括public/protected/private，但不包含父类的**
* **getXxxx：获取自己及父类的方法/变量/构造器等，但只能是public的。**

对于私有构造器/字段/方法，即使通过**getDeclaredXxxx**方法获取到，但在使用之前，也需要使用setAccessible\(true\)来设置访问权限。

**反射到底慢在哪**

反射所花费的时间大约是正常情况下的2倍（只是某种测试用例下，没办法下这种结论，只是有这么一种概念）。那么到底慢在哪里呢？

使用反射，则在运行时会停下来做类加载\(包含很多步\)，加载可以缓存，但是有可能引发雪崩，方法调用还会多步寻址，虽然寻址后还是可以缓存，但这些过程jit没法插手，你可以认为java变成了php。

> TODO：翻译
>
> Reflection is slow for a few obvious reasons:
>
> 1. The compiler can do no optimization whatsoever as it can have no real idea about what you are doing. This probably goes for the`JIT`as well
> 2. Everything being invoked/created has to be _discovered_\(i.e. classes looked up by name, methods looked at for matches etc\)
> 3. Arguments need to be dressed up via boxing/unboxing, packing into arrays,`Exceptions`wrapped in
>    `InvocationTargetException`s and re-thrown etc.
> 4. All the processing that
>    [_Jon Skeet_mentions here](https://stackoverflow.com/questions/1392351/java-reflection-why-is-it-so-bad/1392379#1392379)
>    .
>
> Just because something is 100x slower _does not mean it is too slow for you _assuming that reflection is the "right way" for you to design your program. For example, I imagine that IDEs make heavy use of reflection and my IDE is mostly OK from a performance perspective.
>
> After all, the **overhead of reflection **is likely to **pale into insignificance **when **compared with**, say, parsing XML or **accessing a database**!
>
> Another point to remember is that _micro-benchmarks are a notoriously flawed mechanism for determining how fast something is in practice_. As well as[_Tim Bender's_remarks](https://stackoverflow.com/questions/1392351/java-reflection-why-is-it-so-bad/1392412#1392412), the JVM takes time to "warm up", the JIT can re-optimize code hotspots on-the-fly etc.

#### 参考

[深入解析Java反射（1） - 基础](https://www.sczyh30.com/posts/Java/java-reflection-1/)：文字部分大多来源于此，略有改动

[Reflections中的getDeclared\*\*与get\*\*的区别](https://jishusuishouji.github.io/2017/05/02/Reflections中的getDeclared-与get-的区别.md/Reflections中的getDeclared__与get__的区别_/)

[Java 反射到底慢在哪里？](https://www.zhihu.com/question/19826278)

[Java Reflection: Why is it so slow?](https://stackoverflow.com/questions/1392351/java-reflection-why-is-it-so-slow)



