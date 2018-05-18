# 享元模式

享元模式，英文为Flyweight Pattern，fly weight翻译过来即轻量级（可以飞起来的重量，就是轻量级啊）

享元模式的本质：对象复用（节省内存空间，提升系统性能）

解释：假如你需要1000个对象A，而这1000个对象有非常多的属性或状态都是一样的（不会改变），只有一部分属性或状态不一样（随外界需求而变）。我们将这些一致的不变的属性称之为内部状态，将随外界环境（其实就是客户需求）的改变而改变的属性称之为外部状态。

举个例子，假如在设计游戏的时候，在一些场景下，需要1000个小兵，这1000个小兵只是在地图上的位置不同，但战斗力都相同。这里小兵的战斗力就是小兵的内部状态，而小兵的位置就是小兵的外部状态。

再举个例子，还有五子棋、围棋等游戏，假如需要100个棋子，这些棋子的位置是随时改变的，而棋子的归属（哪个玩家的）、棋子的颜色（一般同一玩家的棋子颜色一致）都是不会变化的，这里位置就是棋子的外部状态，而棋子的归属、颜色就是棋子内部状态。

再再举一个例子，假如系统中有10本书，然后每本书有1000个订单，现在这10\*1000个订单的信息（每个订单中要包含对应书的信息）。对这10\*1000个订单而言，订单自身信息（如下单人、下单时间、金额等）是不一样的，就是外部信息，而对于订单中包含的书的信息则属于内部状态，只有10种，系统中，只要有10个书的实例即可。享元，共享的是这10个书的实例。

**示例：上面书与订单的例子**

书

```java
public class Book {

    private String bookName;
    private String author;
    private int price;

    public String getBookName() {
        return bookName;
    }

    public void setBookName(String bookName) {
        this.bookName = bookName;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }
}
```

订单

```java
public class Order {

    private String username;
    private Book book;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Book getBook() {
        return book;
    }

    public void setBook(Book book) {
        this.book = book;
    }
}
```

生成订单

```java
public class OrderFactory {

    private static final Map<String, Book> books = new HashMap<>();

    public static Order generateOrder(String username, String bookName){
        Order order = new Order();
        order.setUsername(username);
        if(books.containsKey(bookName)){
            order.setBook(books.get(bookName));
        }else {
            Book book = createNewBookInstance(bookName);
            order.setBook(book);
            books.put(bookName,book );
        }
        return order;
    }

    private static Book createNewBookInstance(String bookName){
        //演示，实际可能是去数据库等获取具体信息
        Book book = new Book();
        book.setBookName(bookName);
        book.setAuthor("maxwell");
        book.setPrice(new Random().nextInt(100));
        return book;
    }
}
```

客户端

```java
public class Client {

    public static void main(String[] args) {

        //假如只有10种书，每本书有1000个订单，而在某处我们需要将这些订单信息都加载到内存中，
        // 这10*1000个订单实例则会共享这10个书的实例
        // 这样就可以节省[（10*1000 - 10 ）* 每个书的实例的大小]的内存空间
        List<Order> orders = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            String bookName = "book" + i;
            for (int j = 0; j < 1000; j++) {
                Order order = OrderFactory.generateOrder("xiaoming" + j, bookName);
                orders.add(order);
            }
        }
    }
}
```

## 参考

[JAVA设计模式-享元模式（Flyweight）](https://www.jianshu.com/p/f88b903a166a)

[一起学设计模式 - 享元模式](http://blog.battcn.com/2017/11/17/java/design-pattern/flyweight-pattern/)

