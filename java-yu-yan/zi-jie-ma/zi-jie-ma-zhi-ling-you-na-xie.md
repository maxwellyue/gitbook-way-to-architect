# 字节码指令有哪些

字节码是JVM的机器语言。JVM加载类文件时，对类中的每个方法，都会得到一个字节码流。这些字节码流保存在JVM的方法区中。在程序运行过程中，当一个方法被调用时，它的字节码流就会被执行。

**基础简介**

1. 方法的字节码流就是JVM的指令（instruction）序列。
2. Java虚拟机的指令：由一个字节长度、代表着某种特定操作含义的数字（`操作码Opcode`）以及跟随其后的0-n个代表此操作所需参数（`操作数Operaands`）而构成。即**指令=操作码+操作数**，虚拟机中许多指令并不包含操作数，只有一个操作码。
3. JVM中，所有的计算都是围绕栈。因为JVM没有存储任意数值的寄存器，**所有的操作数在计算开始之前，都必须先压入栈中**。

**字节码和数据类型**

在java虚拟机中，**大多数的指令都包含了操作数所对应的数据类型**。这一点很重要，理解了这一点很多指令都能联想记忆，例如：_iload指令用于从局部变量表中加载int型的数据到操作数栈中，而fload就是加载float类型的数据_。下面列出：

1. i–**int**
2. l–**long**
3. s–**short**
4. b–**byte**
5. c–**char**
6. f–**float**
7. d–**double**
8. a–**reference**

## 指令详解

### 1、加载和存储指令

每个操作如果需要从操作栈中读参数，则总是将这些**参数出栈**，如果操作有结果，总是会将结果**入栈**。

加载：将数据从**栈帧的局部变量表**加载到**操作数栈**;  
存储：将**操作数栈**存储到**栈帧的局部变量表**;

* 将一个局部变量加载到操作栈：`load`。例如： `iload、iload_n、lload、lload_n、fload、fload_n、dload、dload_n、aload、aload_n`
* 将一个数值从操作数栈存储到局部变量表：`store`。例如： `istore、istore_n、lstore、lstore_n、fstore、fstore_n、dstore、dstore_n、astore、astore_n`
* 将一个常量加载到操作数栈，例如： `bipush、sipush、ldc、ldc_w、ldc2_w、aconst_null、iconst_m1、iconst_i、lconst_l、fconst_f、dconst_d`

### 2、运算指令

运算指令用于对两个操作数栈上的值进行某种特定运算，并把结果重新存入到操作栈顶。  
运算指令分为两种：整型（int/long）、浮点型（float/double）。对于没有直接支持 byte、short、char和boolean的算术指令使用int。

所有的算术指令如下：

* 加法指令（add）：`iadd、ladd、fadd、dadd`
* 减法指令\(sub\)：`isub、lsub、fsub、dsub`
* 乘法指令\(mul\)：`imul、lmul、fmul、dmul`
* 除法指令\(div\)：`idiv、ldiv、fdiv、ddiv`
* 求余指令\(rem\)：`irem、lrem、frem、drem`
* 取反指令\(neg\)：`ineg、lneg、fneg、dneg`
* 位移指令：`ishl、ishr、iushr、lshl、lshr、lushr`
* 按位或指令\(or\)：`ior、lor`
* 按位与指令\(and\)：`iand、land`
* 按位异或指令\(xor\)：`ixor、lxor`
* 局部变量自增指令：`iinc`\(是少见的直接更新一个局部变量而无需在操作数栈中进行读写的指令\)
* 比较指令\(cmp\)：`dcmpg、dcmpl、fcmpg、fcmpl、lcmp`

### 3、类型转换指令 {#类型转换指令}

类型转换指令一般用于实现用户代码的显式类型转换操作，或者用来处理 Java 虚拟机字节码指令集中指令非完全独立的问题。

**宽化类型转换**

* int 类型到 long、float 或者 double 类型
* long 类型到 float、double 类型
* float 类型到 double 类型

**窄化类型转换**

i2b、i2c、i2s、l2i、f2i、f2l、d2i、d2l 和 d2f。窄化类型转换可能会导致转换结果产生不同的正负号、不同的数量级，转换过程很可能会导致数值丢失精度。

### 4、对象创建与操作 {#对象创建与操作}

虽然类实例和数组都是对象，但 Java 虚拟机对类实例和数组的创建与操作使用了不同的字节码指令：

* 创建类实例的指令：`new`
* 检查类实例类型的指令：`instanceof`、`checkcas`
* 访问实例变量（无statice修饰）和类变量（有static修饰）的指令：`getfield`、`putfield`、`getstatic`、`putstatic`
* 创建数组的指令：`newarray`、`anewarray`、`multianewarray`
* 把一个数组元素加载到操作数栈的指令：`baload`、`caload`、`saload`、`iaload`、`laload`、`faload`、`daload`、`aaload`
* 将一个操作数栈的值储存到数组元素中的指令：`bastore`、`castore`、`sastore`、`iastore`、`fastore`、`dastore`、`aastore`
* 取数组长度的指令：`arraylength`

### 5、操作数栈管理指令 {#操作数栈管理指令}

虚拟机提供了一些用于直接操作操作数栈的指令，包括：pop、pop2、dup、dup2、dup\_x1、dup2\_x1、dup\_x2、dup2\_x2 和 swap。

### 6、控制转移指令 {#控制转移指令}

控制转移指令的作用是让JVM有条件或无条件地从指定指令转移到指令的下一条指令继续执行程序。

控制转移指令包括有：

* 条件分支：`ifeq、iflt、ifle、ifne、ifgt、ifge、ifnull、ifnonnull、if_icmpeq、if_icmpne、if_icmplt, if_icmpgt、if_icmple、if_icmpge、if_acmpeq` 和 `if_acmpne`
* 复合条件分支：`tableswitch`、`lookupswitch`
* 无条件分支：`goto`、`goto_w`、`jsr`、`jsr_w`、`ret`

### 7、方法调用和返回指令 {#方法调用和返回指令}

**方法调用指令**

`invokevirtual`：**调用对象的实例方法**  
`invokeinterface`：**调用接口方法**，它会在运行时搜索一个实现了这个接口方法的对象，找出适合的方法进行调用。  
`invokespecial`：**调用一些需要特殊处理的实例方法**：实例初始化方法、私有方法和父类方法。  
`invokestatic`：**调用static方法**。

**方法返回指令**

`ireturn`：当返回值是 boolean、byte、char、short 和 int 类型时使用  
`lreturn`：返回long类型  
`freturn`：返回floalt类型  
`dreturn`：返回double类型  
`areturn`：返回对象  
`return` ：void方法、实例初始化方法、类和接口的类初始化方法的返回指令

### 8、抛出异常 {#抛出异常}

在程序中显式抛出异常的操作会由 `athrow` 指令实现，除了这种情况，还有别的异常会在其它 Java 虚拟机指令检测到异常状况时由虚拟机自动抛出。

### 9、同步 {#同步}

JVM可以支持**方法级的同步**和**方法内部一段指令序列的同步**，这两种同步结构都是使用管程（`Monitor`）来支持的。

**方法级的同步**：是隐式的，即无需通过字节码指令来控制的，它实现在方法调用和返回操作之中。虚拟机可以从方法常量池中的方法表结构（method\_info Structure）中的 ACC\_SYNCHRONIZED 访问标志区分一个方法是否同步方法。当方法调用时，调用指令将会检查方法的 ACC\_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先持有管程，然后再执行方法，最后再方法完成（无论是正常完成还是非正常完成）时释放管程。在方法执行期间，执行线程持有了管程，其他任何线程都无法再获得同一个管程。如果一个同步方法执行期间抛出了异常，并且在方法内部无法处理此异常，那这个同步方法所持有的管程将在异常抛到同步方法之外时自动释放。

**同步一段指令集序列**：通常是由Java语言中的**synchronized块**来表示的。JVM的指令集中有`monitorenter`和`monitorexit`两条指令来支持`synchronized`关键字的语义，正确实现 synchronized 关键字需要编译器与 Java 虚拟机两者协作支持。

**结构化锁定（Structured Locking）**：是指在方法调用期间每一个管程退出都与前面的管程进入相匹配的情形。因为无法保证所有提交给 Java 虚拟机执行的代码都满足结构化锁定，所以 Java 虚拟机允许（但不强制要求）通过以下两条规则来保证结构化锁定成立。假设 T 代表一条线程，M 代表一个管程的话：T在方法执行时持有管程 M 的次数必须与 T 在方法完成（包括正常和非正常完成）时释放管程 M 的次数相等。在方法调用过程中，任何时刻都不会出现线程 T 释放管程 M 的次数比 T 持有管程 M 次数多的情况。  
请注意，在同步方法调用时自动持有和释放管程的过程也被认为是在方法调用期间发生。



来源：

[Java-字节码指令](https://blog.xiaoxiaomo.com/2016/04/01/Java-%E5%AD%97%E8%8A%82%E7%A0%81%E6%8C%87%E4%BB%A4/)：有字节码解释示例



