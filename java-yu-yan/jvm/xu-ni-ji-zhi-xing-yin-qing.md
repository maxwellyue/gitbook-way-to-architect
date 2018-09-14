# Java虚拟机的执行引擎

输入的是字节码文件，处理过程是字节码解析的等效过程，输出的是执行结果。

## 帧栈

---

### 概念

帧栈是用于支持虚拟机进行方法调用和方法执行的数据结构，它是虚拟机运行时数据区中的虚拟机栈的栈元素。

每一个帧栈中都包括以下信息：局部变量表（`Local Varable Table`）、操作数栈（`Operand Stack`）、动态连接（`Dynamic Linking`）、方法返回地址（`Return Address`）和一些额外的附加信息。

一个帧栈需要分配多大内存，不会受到程序运行期间变量数据的影响，而是在程序代码编译时就确定了的（在方法表的Code属性中，详见类文件结构中的内容）。

一个线程中的方法调用链可能会很长，对于执行引擎来说，在活动线程中，只有位于栈顶的帧栈才是有效的，称为当前帧栈，与这个帧栈相关联的方法称为当前方法。执行引擎运行的字节码指令都只针对当前帧栈进行操作，在概念模型上，典型的帧栈结构如下（栈是线程私有的，也就是每个线程都会有自己的栈）。

![典型的帧栈结构](http://upload-images.jianshu.io/upload_images/1932449-a249e30ebf3c5408.png?imageMogr2/auto-orient/strip|imageView2/2/w/1240)

**局部变量表**

存放**方法参数和方法内部定义的局部变量**。在编译阶段，就在Class文件的Code属性的max\_locals数据项中确定了该方法所需要分配的局部变量表的最大容量。（仅仅是变量，不包括具体的对象）。

局部变量表内部以变量槽（Variable Slot）为最小单位。对于`byte`、`char`、`float`、`int`、`short`、`boolean`、`reference`、`returnAddress`等长度不超过32位的数据类型，每个局部变量占用一个Slot，double和long这两种64位的数据类型则需要两个Slot。

在方法执行时，虚拟机是使用局部变量表完成参数值到参数变量列表的传递的，如果执行的是实例方法（非`static`方法），则局部变量表中第0位索引的Slot默认是用于传递方法所属对象实例的引用，在方法中通过`this`来访问这个隐含的参数。

局部变量表中的Slot是可重用的，如果当前字节码PC计数器的值已经超出了某变量的作用域，则这个变量对应的Slot可以交给其它变量重用。重用可以节省栈空间，但也会带来副作用：（为虚拟机设置运行参数加上`-verbose:gc`即可输出`gc`日志信息）

```java
//--------------------------测试1---------------------------//
public static void main(String[] args){
        byte[] placeholder = new byte[64*1000*1000];
        System.gc();
}
//查看日志，并未回收
[GC (System.gc())  69437K->63438K(251392K), 0.0012879 secs]
[Full GC (System.gc())  63438K->63277K(251392K), 0.0058505 secs]
//------------------------测试2-----------------------------//
public static void main(String[] args) {
        {
            byte[] placeholder = new byte[64 * 1000 * 1000];
        }
        System.gc();
}
//查看日志，并未回收
[GC (System.gc())  69437K->63420K(251392K), 0.0011785 secs]
[Full GC (System.gc())  63420K->63277K(251392K), 0.0058676 secs]
//------------------------测试3-----------------------------//
public static void main(String[] args) {
        {
            byte[] placeholder = new byte[64 * 1000 * 1000];
        }
        int a = 0;
        System.gc();
}
//查看日志，回收了
[GC (System.gc())  69437K->63454K(251392K), 0.0011921 secs]
[Full GC (System.gc())  63454K->777K(251392K), 0.0056915 secs]
```

测试1中在`System.gc()`时，变量`placeholder`还处在作用于之内，不会回收；测试2在`System.gc()`时，变量`placeholder`虽然已经不在作用域，但是`placeholder`原本所占用的Slot还没有被复用，所以作为GC Root一部分的局部变量表仍然保持着对它的关联，所以也没有回收。这种关联没有被及时打破的影响在绝大部分  
下都很轻微，但假如有一个方法，后面的代码有一些耗时很长的操作，而前面又定义了占用大量内存、实际已经不会再使用的变量，则手动将其设为null是有意义的。&lt;/br&gt;  
还有一点就是，局部变量不像类变量（仅指被static修饰的变量，不包括实例变量）一样存在准备阶段，它不存在系统默认值。所以**必须为局部变量定义初始值**。（不指定，编译也会报错）。

**操作数栈**  
操作数栈，也叫操作栈，是先入后出的栈。其中的元素是任意的Java数据类型，包括`long`和`double`。32位数据类型所占容量为1，64位为2。在方法执行的任何时刻，操作数栈的最大深度都不会超过Code属性中`max_stacks`数据项所设定的最大值。&lt;/br&gt;  
当一个方法开始执行的时候，操作栈是空的，在方法的执行过程中，会有各种字节码指令出栈/入栈。例如，在做算术运算的时候，是通过操作栈来进行的，调用其他方法的时候是通过操作栈来进行参数传递的。

例如，整数加法的字节码指令`iadd`在运行的时候，操作栈中最接近栈顶的两个元素已经存入了两个int型数值，当执行这个指令时，会将这两个int值出栈并相加，然后将相加的结果入栈。

在概念模型中，两个帧栈作为虚拟机栈的元素是完全独立的，但是在大多数虚拟机实现中都会做优化，将两个帧栈出现一部分重叠：让下面帧栈的部分操作栈与上面帧栈的部分局部变量表重叠，以便在方法调用时共用一部分数据，避免不必要的参数复制传递。

**动态连接**  
每个帧栈都包含一个指向运行时常量池中该帧栈所属方法的引用。持有这个引用是为了支持方法调用过程中的动态连接。字节码中的方法调用指令（这里，“方法调用”是指令的修饰词，不要理解错了）以常量池中指向方法的符号引用作为参数。这些符号引用一部分会在类加载阶段或者第一次使用的时候就转化为直接引用，这种转化称为**静态解析**。另外一部分将在每一次运行期间转化为直接引用，这部分称为**动态连接**。

**方法返回地址**  
当一个方法开始执行后，有两种方式退出这个方法：

* 正常完成出口
  执行引擎遇到方法返回的字节码指令，此时将返回值传递给上层的方法调用者（是否有返回值以及返回值的类型由方法返回指令来决定）
* 异常完成出口
  方法执行过程中出现异常，并且该异常没有在方法体中处理（可能是Java虚拟机内部产生的异常，也可能代码中使用`athrow`字节码指令产生的异常）。
  方法退出实际就是当前帧出栈，因此退出时可能执行的操作有：恢复上层方法的局部变量表和操作数栈，把返回值（如果有）压入调用者帧栈的操作数栈中，调整PC计数器的值以指向方法调用指令后面的一条指令。

**附加信息**

规范之外的，取决于具体虚拟机实现。

## 方法调用

---

方法调用并不等同于方法执行，**方法调用阶段的唯一目的就是确定被调用的方法的版本（即调用哪个方法）**。一切方法调用在Class文件里存储的都是符号引用，而不是方法的直接引用（方法在实际运行时内存布局中的入口地址）。

在虚拟机中，有5条方法调用字节码指令：  
①`invokestatic`：调用静态方法；  
②`invokespecial`：调用实例构造器`<init>`方法、私有方法和父类方法；  
③`invokevirtual`：调用所有的虚方法；  
④`invokeinterface`：调用接口方法，在运行时再确定一个实现此接口的对象；  
⑤`invokedynamic`：先在运行时动态解析出调用点限定符所引用的方法，然后再执行该方法。

方法的调用可以分为解析调用和分派调用。

* **解析调用**  
  在类加载中的解析阶段，会将方法调用中的目标方法的一部分符号引用转化为直接引用，这部分可以转化的前提是：方法在程序运行之前就有一个可确定的调用版本，并且这个方法的调用版本在运行期间是不可改变的。这类方法的调用称为解析（或解析调用）。  
  在Java中，符合上述特点（编译器可知，运行期不可变）的方法：静态方法和私有方法。与对应方法调用指令，只要能被`invokestatic`和`invokespecial`指令调用的方法，都是可以在解析阶段确定唯一调用版本的。这类方法称为非虚方法，其他方法都称为虚方法。  
  在Java中，非虚方法除了`invokestatic`和`invokespecial`指令能调用的方法外，还包括`final`方法。  
  解析调用一定是静态的过程，在编译期间就可以完全确定，在类加载的解析阶段就可以把方法的符号引用转变为直接引用，不会延迟到运行期间再去完成。

* **分派调用**  
  分派调用是理解继承、封装和多态（尤其是重载和重写）的关键。对它的理解，可以更清楚虚拟机是如何确定正确的目标方法的。

  * **静态分派**  
    首先来理解两个概念：静态类型和实际类型。

    ```java
    Human human = new Man();//这里假设Man是Human的子类
    ```

    上述代码中：`Human`为变量的静态类型，`Man`为变量的实际类型。  
    虚拟机（准确地说是编译器）在重载时，是通过参数的静态类型而不是实际类型作为判定依据的。

    ```java
    public class StaticDispatch {

    static abstract class Human{}

    static class Man extends Human{}

    static class Woman extends Human{}

    public void sayHello(Human human){
        System.out.println("hello, human");
    }

    public void sayHello(Man man){
        System.out.println("hello, man");
    }

    public void sayHello(Woman woman){
        System.out.println("hello, woman");
    }

    @Test
    public void test(){
        Human man = new Man();
        Human woman = new Woman();
        StaticDispatch dispatch = new StaticDispatch();
        dispatch.sayHello(man);
        dispatch.sayHello(woman);
    }
    }
    //最终的打印结果如下：（重载时是以静态类型判断的）
    hello, human
    hello, human
    ```

    编译器虽然可以确定方法的重载版本，但在很多情况下这个重载版本并不是“唯一的”，往往只能确定一个“更加合适的”版本。这种情况的产生的主要原因是字面量不需要定义，所以字面量没有显示的静态类型，它的静态类型只能通过语言上的规则去理解和推测。

    ```
    public class OverLoad {

        public static void sayHello(Object object){
            System.out.println("hello Object");
        }
        public static void sayHello(int a){
            System.out.println("hello int");
        }
        public static void sayHello(long a){
            System.out.println("hello long");
        }
        public static void sayHello(Character character){
            System.out.println("hello Character");
        }
        public static void sayHello(char c){
            System.out.println("hello char");
        }
        public static void sayHello(char... c){
            System.out.println("hello char...");
        }
        public static void sayHello(Serializable serializable){
            System.out.println("hello Serializable");
        }

        @Test
        public void test(){
            sayHello('a');
        }
    }
    ```

    以上代码，将打印出`hello char`；  
    ①将`sayHello(char c)`注释掉，将打印出：`hello int`，  
    ②继续将`sayHello(int a)`注释掉，将打印出：`hello long`，  
    这两步的原因是字符`a`发生自动类型转换（char-&gt;int-&gt;long-&gt;float-&gt;double）  
    ③继续将`sayHello(long a)`注释掉，将打印出：`hello Character`  
    原因是字符`a`被自动装箱为`Character`类型  
    ④继续将`sayHello(Character character)`注释掉，将打印出：`hello Serializable`  
    原因是`a`被自动装箱为`Character`类型后仍然找不到方法，继续自动转型，`Character`实现了`Serializable`接口。  
    ⑤继续将`sayHello(Serializable serializable)`注释掉，将打印出：`hello Object`  
    原因是char装箱后转型为父类了，如果有多个父类，将在继承关系中从下往上搜索，约接近上层优先级越低。  
    ⑥继续将`sayHello(Object object)`注释掉，将打印出：`hello char...`  
    可见：可变长参数的重载优先级是最低的。

  * **动态分派**  
    在运行期间，根据参数实际类型来确定方法执行版本的过程。主要对应Java中的重写。

    ```
    public class DynamicDispatch {

        static abstract class Human {
            abstract void sayHello();
        }

        static class Man extends Human {
            @Override
            void sayHello() {
                System.out.println("man say hello");
            }
        }

        static class Woman extends Human {
            @Override
            void sayHello() {
                System.out.println("woman say hello");
            }
        }

        @Test
        public void test() {
            Human man = new Man();
            Human woman = new Woman();
            man.sayHello();
            woman.sayHello();
        }
    }
    //将会打印
    man say hello
    woman say hello
    ```

    通过以上可知，静态分派与动态分派是不同情况下方法调用所采取的不同的分派方式，两者并不是非此即彼的，还可能出现一个方法调用在确定直接引用时，既用到静态分派，又用到动态分派。确定重载方法的时候用到的是静态分派，确定重写方法的时候用到的是动态分派。即**重载看参数静态类型，重写看参数实际类型**。这里的参数，重载时是指方法的参数列表中那个参数，重写时是指该方法的调用者。

  * 单分派与多分派  
    方法的接受者和方法的参数统称为方法的宗量，根据分派基于多少种宗量，可以将分派划分为单分派和多分派两种。  
    Java语言属于**静态多分派、动态单分派**的语言。

  * 虚拟机动态分派的实现

动态类型语言支持：TODO

---

### 基于栈的字节码解释执行引擎

//TODO  
主要探讨虚拟机如何执行方法中的字节码指令。许多Java虚拟机的执行引擎在执行Java代码的时候都有解释执行（通过解释器执行）和编译执行（通过即时编译器生成本地代码执行）两种选择，这里进讨论解释执行。

###### 解释执行

###### 基于栈的指令集和基于寄存器的指令集

###### 基于栈的解释器执行过程



内容来自《深入理解Java虚拟机》

