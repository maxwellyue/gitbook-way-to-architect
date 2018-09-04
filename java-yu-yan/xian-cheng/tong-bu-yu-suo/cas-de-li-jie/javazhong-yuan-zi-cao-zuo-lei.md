# Java中原子操作类

从JDK1.5开始，Java提供了`java.util.concurrent.atomic`包，该包中的原子操作类提供了一种使用简单、性能高效**（使用CAS操作，无需加锁）、线程安全**地更新一个变量的方式。  
![](https://upload-images.jianshu.io/upload_images/1932449-0b6bed1d4ef31ec1.png?imageMogr2/auto-orient/strip|imageView2/2/w/1000/format/webp)

根据变量类型的不同，Atomic包中的这12个原子操作类（都是使用Unsafe实现的包装类）可以分为4种类型：

* **原子更新基本类型**：`AtomicBoolean、AtomicInteger、AtomicLong`
* **原子更新数组**：`AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray`
* **原子更新引用**：`AtomicReference、AtomicReferenceFiledUpdater、AtomicMarkableReference`
* **原子更新字段（属性）**：`AtomicIntegerFieldUpdater、AtomicLongFieldUpdater、AtomicStampedReference` 

## 原子更新基本类型类

使用原子的方式更新基本类型，Atomic包提供了以下3个类：  
①`AtomicBoolean`  
②`AtomicInteger`  
③`AtomicLong`  
这三个类提供的方式几乎是一模一样，下面以`AtomicInteger`为例进行讲解，`AtomicInteger`的常用的方法如下：

```java
int addAndGet(int delta)
以原子方式将AtomicInteger的value设置为：delta + 原value，返回更新后的值（即delta + 原value）

boolean compareAndSet(int expect, int update)
以原子的方式，如果AtomicInteger的当前值是expect，则将AtomicInteger的值设置为update

int getAndIncrement()
以原子的方式将AtomicInteger的当前值加1，注意：返回的是加1前的值

void lazySet(int newValue)
最终将AtomicInteger设置为newValue（使用lazySet设置值后，其他线程可能在之后的一段时间内还是可以读到旧的值）

int getAndSet(int newValue)
以原子的方式将AtomicInteger设置为newValue
```

基本使用方式示例：

```java
public void test(){
    Executor executor = Executors.newFixedThreadPool(3);
    AtomicInteger atomicInteger = new AtomicInteger(0);

    for(int i = 0; i < 10; i++){
        executor.execute(()->{
            System.out.println("atomicInteger的当前值：" + atomicInteger.addAndGet(1));
        });
    }
}

//输出如下（输出顺序可能不同，但结果一定是正确的）
atomicInteger的当前值：1
atomicInteger的当前值：2
atomicInteger的当前值：4
atomicInteger的当前值：5
atomicInteger的当前值：3
atomicInteger的当前值：7
atomicInteger的当前值：6
atomicInteger的当前值：9
atomicInteger的当前值：8
atomicInteger的当前值：10
```

其内部实现原子操作的原理是通过`UnSafe`类的`CAS`操作。//TODO 具体实现  
其他`Java`的基本类型均可以使用类似的思路实现。

### 原子更新数组

通过原子的方式更新数组里的某个元素，Atomic包提供了以下3个类：  
①`AtomicIntegerArray`：原子更新整型数组里的元素  
②`AtomicLongArray`：原子更新长整型数组里的元素  
③`AtomicReferenceArray`：原子更新引用类型数组里的元素

`以AtomicIntegerArray`为例，主要是提供以原子的方式更新数组里的整型元素，其主要方法如下：

```java
int addAndGet(int i, int delta)
以原子的方式将数组中i位置处的元素值加上delta，返回：i位置处的元素的旧值+ delta

boolean compareAndSet(int i, int expect, int update)
如果当前值等于预期值（数组i位置处的元素），则以原子的方式将数组i位置处的元素值设置为update
```

使用示例：

```java
public void testAtomicIntegerArray() {

    int[] originArray = new int[]{1, 2, 3};

    AtomicIntegerArray array = new AtomicIntegerArray(originArray);

    array.getAndSet(0, 8);
    System.out.println(array.get(0));
    System.out.println(originArray[0]);
}
//输出结果：
8
1   ----注意这里，构造方法中是将原数组复制了一份，所以对AtomicIntegerArray的操作，不会影响原数组
```

### 原子更新引用类型

如果要原子更新多个变量，就需要使用原子更新引用类型，Atomic提供了3个类：  
①`AtomicReference`：原子更新引用类型  
②`AtomicReferenceFiledUpdater`：原子更新引用类型里的字段  
③`AtomicMarkableReference`：原子更新带有标记位的引用类型

以`AtomicReference`为例，演示代码如下：

```java
class AtomicReferenceExample{

    private  AtomicReference<User> userAtomicReference = new AtomicReference<>();

    @Test
    public void test(){
        User originUser = new User(18, "小岳");
        userAtomicReference.set(originUser);
        User updateUser = new User(28, "老岳");
        userAtomicReference.compareAndSet(originUser, updateUser);
        System.out.println(userAtomicReference.get().getName() + ":" + userAtomicReference.get().getAge());
    }

    class User{
        private String name;
        private int age;

        public User(int age, String name){
            this.name = name;
            this.age = age;
        }

        public int getAge() {
            return age;
        }

        public String getName() {
            return name;
        }
    }
}
//输出如下
老岳:28
```

### 原子更新字段类

如果需要原子地更新某个类里的某个字段，就需要使用原子更新字段类，Atomic包提供了一下3个类进行原子字段更新。  
①`AtomicIntegerFieldUpdater`：原子更新整型的字段  
②`AtomicLongFieldUpdater`：原子更新长整型的字段  
③`AtomicStampedReference`：原子更新带有版本号的引用类型（**可以解决CAS操作的ABA问题**）

使用更新字段类必须使用静态方法`newUpdater(Class<U> tclass, String fieldName)`创建一个更新器（同时指定要更新的类和该类中的要更新的字段名），并且**该字段必须用**`public volatile`**修饰**。

以`AtomicIntegerFieldUpdater`为例，演示代码如下：

```java
class AtomicIntegerFieldUpdaterExample{

    private AtomicIntegerFieldUpdater<User> fieldUpdater = AtomicIntegerFieldUpdater.newUpdater(User.class, "age");

    @Test
    public void test(){
        User user = new User(18, "小岳");
        fieldUpdater.addAndGet(user, 10);
        System.out.println("user现在的年龄:" + fieldUpdater.get(user));
    }

    class User{
        private String name;
        public volatile int age;

        public User(int age, String name){
            this.name = name;
            this.age = age;
        }

        public int getAge() {
            return age;
        }

        public String getName() {
            return name;
        }
    }
}
//输出如下
user现在的年龄:28
```

