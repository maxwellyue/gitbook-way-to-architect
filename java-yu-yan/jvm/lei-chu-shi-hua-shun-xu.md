# 类初始化顺序

* new关键字实例化对象、读取或设置一个类的静态字段（被final修饰、已在编译期把结果放入常量池的静态字段除外）、调用一个类的静态方法
* 使用`java.lang.reflect`包中的方法对类进行**反射调用**的时候，如果类没有被初始化过，则需要先触发初始化。
* 当初始化一个类的时候，如果发现其父类还没有进行过初始化，则需要**先触发父类**的初始化。
* 当虚拟机启动时，用户需要指定一个要执行的主类（包含main方法的那个类），虚拟机会先初始化这个主类。

## 初始化的顺序

* 无继承类初始化顺序 **静态变量/静态代码块 ---&gt; 成员变量/代码块 ---&gt; 构造函数** 其中，静态变量与静态代码块，成员变量与代码块的初始化顺序只取决于定义顺序。
* 子类初始化顺序 **父类静态变量/父类静态方法块 ---&gt; 子类静态变量/子类静态方法块 ---&gt; 父类成员变量/方法块 ---&gt; 父类构造函数 ---&gt; 子类成员变量/方法块 ---&gt; 子类构造函数**

