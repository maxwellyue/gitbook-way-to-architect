# 深拷贝和浅拷贝

下面列表是Java中深拷贝和浅拷贝的区别

| Shallow Copy | Deep Copy |
| :--- | :--- |
| Cloned Object and original object are not 100% disjoint. | Cloned Object and original object are 100% disjoint. |
| Any changes made to cloned object will be reflected in original object or vice versa. | Any changes made to cloned object will not be reflected in original object or vice versa. |
| Default version of clone method creates the shallow copy of an object. | To create the deep copy of an object, you have to override clone method. |
| Shallow copy is preferred if an object has only primitive fields. | Deep copy is preferred if an object has references to other objects as fields. |
| Shallow copy is fast and also less expensive. | Deep copy is slow and very expensive. |

_表格来源_：[Difference Between Shallow Copy Vs Deep Copy In Java](https://link.jianshu.com/?t=http://javaconceptoftheday.com/difference-between-shallow-copy-vs-deep-copy-in-java/)

| 浅拷贝 | 深拷贝 |
| :--- | :--- |
| 原对象和克隆对象并不是100%无关联 | 原对象和克隆对象100%无关联 |
| 对克隆对象的任何改变都会反映在原对象中，反之亦然 | 克隆对象的改变不会反映在原对象中，反之亦然 |
| 默认的clone\(\)方法创建的是浅拷贝 | 要实现深拷贝，必须重写clone\(\)方法 |
| 如果一个对象中字段只有基本类型，推荐浅拷贝 | 如果一个对象中字段存在其他对象的引用类型，推荐深拷贝 |
| 浅拷贝速度快，代价小 | 深拷贝相对较慢，代价大 |

**浅拷贝**

如果属性是基本类型，拷贝的就是基本类型的值；

如果属性是内存地址（引用类型），拷贝的就是内存地址 ，因此如果其中一个对象改变了这个地址，就会影响到另一个对象。

**深拷贝**

拷贝所有的属性，并拷贝属性指向的实际对象（而非仅仅拷贝引用地址）。

# 实现

### 方式1、重写clone\(\)方法

要对某个对象进行浅拷贝，只需要让该类实现Cloneable接口，并重写clone\(\)方法即可：

```java
public class Product {

    private String name;

    //省略 construct/setter/getter
}

public class Order implements Cloneable {

    private String id;

    private Product product;
    
    //省略 construct/setter/getter

    @Override
    public Order clone() throws CloneNotSupportedException{
        return (Order) super.clone();
    }
}
//测试
public static void main(String[] args) throws CloneNotSupportedException{

    Product product = new Product("book");
    Order order = new Order("123", product);
    Order clone = order.clone();
    
    System.out.println(order);
    System.out.println(clone);

    System.out.println(order.getProduct());
    System.out.println(clone.getProduct());
}
//输出如下（省略包名）
Order@5ca881b5
Order@24d46ca6
Product@4517d9a3
Product@4517d9a3
```

可以看到，对Order进行clone\(\)后，生成的对象与原对象内存不一样，但其中的Product却是同一个。

这就是浅拷贝，对基本类型拷贝只，对引用类型，仅拷贝引用地址，并不实际拷贝引用的对象。

在上面的例子中，如果想要实现深拷贝：让Product类也实现cloneable接口，并重写clone\(\)方法，并改造Order的clone\(\)方法：

```java
public class Product implements Cloneable {

    private String name;
    
    //省略 construct/setter/getter

    @Override
    public Product clone() throws CloneNotSupportedException {
        return (Product) super.clone();
    }
}

public class Order implements Cloneable {

    private String id;

    private Product product;

    public Order(String id, Product product) {
        this.id = id;
        this.product = product;
    }

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public Product getProduct() {
        return product;
    }

    public void setProduct(Product product) {
        this.product = product;
    }

    @Override
    public Order clone() throws CloneNotSupportedException {
        Order clone = (Order)super.clone();
        clone.product = this.product.clone();
        return clone;
    }
}
//测试如上，输出如下（省略包名）：
Order@5ca881b5
Order@24d46ca6
Product@4517d9a3
Product@372f7a8d
```

可以发现，如果要实现深拷贝，需要对每个涉及的类重写clone\(\)方法，并对类中的每个引用类型进行clone\(\)，如果涉及的类的层次很深，那么就需要写非常多的clone\(\)方法。

> 注意将重写的clone\(\)方法的访问限制符由默认的protected改为public（如果要在包外用到clone\(\)的话）。

### 方式2、通过序列化实现深拷贝

序列化会将整个对象写入到一个持久化存储文件中，并且当需要的时候把它读取回来（反序列化）, 这意味着当你需要把它读取回来时你需要整个对象的一个拷贝。这就是当你深拷贝一个对象时真正需要的东西。请注意，当你通过序列化进行深拷贝时，必须确保对象中所有类都是可序列化的。

还是用上面的例子：

```java
public class Product implements Serializable{

    private String name;

    //省略 construct/setter/getter
}

public class Order implements Serializable {

    private String id;

    private Product product;
    
    //省略 construct/setter/getter
}

//测试
public static void main(String[] args) throws IOException, ClassNotFoundException {

    Product product = new Product("book");
    Order order = new Order("123", product);

    // 序列化
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
    ObjectOutputStream oos = new ObjectOutputStream(bos);
    oos.writeObject(order);
    oos.flush();
    //反序列化
    ByteArrayInputStream bin = new ByteArrayInputStream(bos.toByteArray());
    ObjectInputStream ois = new ObjectInputStream(bin);
    Order serialOrder = (Order) ois.readObject();

    System.out.println(order);
    System.out.println(serialOrder);

    System.out.println(order.getProduct());
    System.out.println(serialOrder.getProduct());
}
//输出如下（省略包名）
Order@1f32e575
Order@49097b5d
Product@27716f4
Product@6e2c634b
```

序列化实现起来比较简单，不用对原有的类进行大幅改造，但有以下两个问题：

* 无法序列化transient变量
* 性能较差：创建一个socket，序列化一个对象，通过socket传输，然后再反序列化，这个过程与调用已有对象的方法相比是很慢的。它比通过实现Clonable接口这种方式来进行深拷贝几乎多花100倍的时间（这个倍数待求证）。

# 参考

[细说 Java 的深拷贝和浅拷贝](https://segmentfault.com/a/1190000010648514)

[Java 深拷贝和浅拷贝](https://my.oschina.net/jackieyeah/blog/206391)

[Java如何实现深拷贝](https://www.linuxidc.com/Linux/2018-04/151898.htm)



