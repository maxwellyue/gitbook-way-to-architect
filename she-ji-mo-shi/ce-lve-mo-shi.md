
### 概念及定义
---
* 概念
在完成某一功能时，有时需要根据不同环境采取不同的策略或行为。将这些不同的策略或行为（称为算法）一一封装起来，而不是使用if--else，从而在使用的时候，可以将这些算法任意替换。这就是策略模式。

* 使用场景
  * 如果在一个系统里面有许多类，它们之间的区别仅在于它们的行为，那么使用策略模式可以动态地让一个对象在许多行为中选择一种行为。
  * 一个系统需要动态地在几种算法中选择一种。 
  * 如果一个对象有很多的行为，如果不用恰当的模式，这些行为就只好使用多重的条件选择语句来实现。


* 策略模式的结构

![策略模式结构图](http://upload-images.jianshu.io/upload_images/1932449-ccff26a9003c80c8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  * Strategy：策略接口，用来约束一系列具体的策略算法，Context使用这个接口来调用具体的策略实现定义的算法。
  * ConcreteStrategy：具体的策略实现，也就是具体的算法实现。
  * Context：上下文，负责和具体的策略类交互。通常上下文会持有一个真正地策略实现，上下文还可以让具体的策略类来获取上下文的数据，甚至让具体的策略类来回调上下文的方法。



### 实际场景1：报价管理
---
假如现在要对不同客户进行报价，则不同客户会有不同的报价，假设①普通客户或新客户报全价；②老客户统一折扣5%；③大客户统一折扣10%。
使用策略模式，首先创建策略接口，里面仅定义一个用于计算报价的抽象方法`calcPrice()`：
```
public interface Strategy {

    /**
     * 计算应报的价格，即计算报价
     * @param goodsPrice 商品销售原价
     * @return 报价
     */
    double calcPrice(double goodsPrice);
}
```
接下来，为三种具体策略分别创建实现类：
```
//新客户或普通客户
public class NormalCustomerStrategy implements Strategy {
    @Override
    public double calcPrice(double goodsPrice) {
        return goodsPrice;
    }
}
//老客户
public class OldCustomerStrategy implements Strategy {
    @Override
    public double calcPrice(double goodsPrice) {
        return goodsPrice*(1-0.05);
    }
}
//大客户
public class BigCustomerStrategy implements Strategy {
    @Override
    public double calcPrice(double goodsPrice) {
        return goodsPrice*(1-0.1);
    }
}
```
最后，创建上下文类，取名为Price：
```
public class Price {

    //虽然是一个接口，但实际是持有具体的策略实现类对象
    private Strategy strategy;

    /**
     * 构造方法，传入一个具体的策略对象
     * @param strategy 具体的策略对象
     */
    public Price(Strategy strategy){
        this.strategy = strategy;
    }

    /**
     * 计算报价
     * @param goodsPrice
     * @return
     */
    public double quote(double goodsPrice){
        return this.strategy.calcPrice(goodsPrice);
    }
}
```
至此，所有策略模式相关类都已创建完成。
客户端使用的时候：
```
public class Client {

    public static void main(String[] args){
        //1:选择并创建需要使用的策略对象
        Strategy strategy = new BigCustomerStrategy();
        //2:创建上下文
        Price price = new Price(strategy);
        //3:计算报价
        double quote = price.quote(1000);
        System.out.println("应该向客户报价：" + quote);
    }
}
```
在这个例子中，策略算法即报价的计算只需要一个参数：商品原价。这种情况是最简单的，很好处理。
假如不同的策略算法需要的参数是不确定的，又该怎么办？



### 实际场景2：工资支付
---
上面提到的问题的解决方案有两种：①将不同策略算法所需要的参数放在上下文中，然后将上下文传给策略算法；②将不同策略算法所需要的特定的数据放在各自的实现类中。

实际场景：不同员工的工资支付方案不同，比如现金、银行卡、现金加股票、现金加期权、美元等。此时，现金支付不需要银行账号，而银行卡则需要银行账号。
现在，先假设工资支付只支持人民币支付和美元支付。按照策略模式，我们来创建相关的类。
首先，策略接口，注意这里支付方法的参数为上下文（为了之后的扩展）：
```
public interface PaymentStrategy {
    /**
     * 支付
     * @param context 支付上下文
     */
    void pay(PaymentContext context);
}
```
对应的支付上下文类为PaymentContext：
```
public class PaymentContext {

    private String username;
    private double money;
    private PaymentStrategy strategy;

    public PaymentContext(String username, double money, PaymentStrategy strategy){
        this.username = username;
        this.money = money;
        this.strategy = strategy;
    }

    //get方法，以便策略算法在计算时从上下文获取所需数据
    public String getUsername() {
        return username;
    }

    //get方法，以便策略算法在计算时从上下文获取所需数据
    public double getMoney() {
        return money;
    }

    /**
     * 支付工资
     */
    public void payNow(){
        this.strategy.pay(this);
    }
```
具体的两个实现类：人民币现金支付`RMBCash`和美元现金`DollarCash`支付：
```
//人民币现金支付
public class RMBCash implements PaymentStrategy {
    @Override
    public void pay(PaymentContext context) {
        System.out.println("给" + context.getUsername() + "人民币现金支付：" + context.getMoney());
    }
}

//美元现金支付
public class DollarCash implements PaymentStrategy {
    @Override
    public void pay(PaymentContext context) {
        System.out.println("给" + context.getUsername() + "美元现金支付：" + context.getMoney());
    }
}
```
至此，所有策略模式相关类都已创建完成。
客户端使用的时候：
```
public class Client {

    public static void main(String[] args){
        PaymentStrategy strategyRMB = new RMBCash();
        PaymentStrategy strategyDollar = new DollarCash();

        PaymentContext context1 = new PaymentContext("小王", 8000, strategyRMB);
        context1.payNow();

        PaymentContext context2 = new PaymentContext("Peter", 6000, strategyDollar);
        context2.payNow();
    }
}

```

现在，进行扩展：要支持银行卡支付，此时需要将银行卡号这个信息传入具体的策略实现类。根据上面提到的两种解决方案，这个银行卡信息可以放在上下文中，也可以直接放在策略实现类中。现在先看第一种：将银行卡号信息放在上下文中。
此时，创建一个新的上下文类`PaymentContext2`，使其继承原来的上下文`PaymentContext`，并增加一个成员变量`account`：
```
public class PaymentContext2 extends PaymentContext{

    //银行卡号
    private String account;
    
    public PaymentContext2(String username, double money, String account, PaymentStrategy strategy) {
        super(username, money, strategy);
        this.account = account;
    }

    //get方法，以便策略算法在计算时从上下文获取所需数据
    public String getAccount() {
        return account;
    }
}
```
然后，增加一个`PaymentStrategy `实现类
```
public class Card implements PaymentStrategy {
    @Override
    public void pay(PaymentContext context) {
        //进行强转
        PaymentContext2 context2 = (PaymentContext2)context;
        //转账
        System.out.println("给" + context2.getUsername() + "的银行卡号" + context2.getAccount() + "支付：" + context2.getMoney());
    }
}
```
好了，可以调用了：
```
public class Client {

    public static void main(String[] args){
        PaymentStrategy strategyRMB = new RMBCash();
        PaymentStrategy strategyDollar = new DollarCash();

        PaymentContext context1 = new PaymentContext("小王", 8000, strategyRMB);
        context1.payNow();

        PaymentContext context2 = new PaymentContext("Peter", 6000, strategyDollar);
        context2.payNow();

        //新增银行卡支付
        PaymentStrategy strategyCard = new Card();
        PaymentContext2 context3 = new PaymentContext2("小李", 5000, "6223987999987788", strategyCard);
        context3.payNow();
    }
}
```
现在，看另外一个解决方法：将银行号账号信息放在具体的策略实现类中。此时，就不必扩展上下文，只需要策略实现类即可，只是这个策略实现类除了支付算法，还要支付算法用到的特定数据，也就是说该实现类在进行策略计算时，所需数据不能完全从上下文中获取，因为自身持有所需要的必要信息。该实现类具体如下：
```
public class Card2 implements PaymentStrategy {

    private String account;

    public Card2(String account){
        this.account = account;
    }
    
    @Override
    public void pay(PaymentContext context) {
        System.out.println("给" + context.getUsername() + "的银行卡号" + this.account + "支付：" + context.getMoney());

    }
}
```
好了，调用时：
```
public class Client {

    public static void main(String[] args){
        PaymentStrategy strategyRMB = new RMBCash();
        PaymentStrategy strategyDollar = new DollarCash();

        PaymentContext context1 = new PaymentContext("小王", 8000, strategyRMB);
        context1.payNow();

        PaymentContext context2 = new PaymentContext("Peter", 6000, strategyDollar);
        context2.payNow();
        
        //新增银行卡支付
        PaymentStrategy strategyCard2 = new Card2("6223987999987788");
        PaymentContext context4 = new PaymentContext("小张", 7000, strategyCard2);
        context4.payNow();
    }
}
```
这两种方式的比较：
**扩展上下文的方式**：所有的策略的实现风格统一，策略所需要的数据都统一从上下文中获取；在上下文中添加的新数据，别的算法也可以用的上，可以视为公共的数据；但缺点是，如果这些数据都只有一个特定的策略算法适用，那么这些数据就有些浪费；另外每次添加新的算法都去扩展上下文，容易形成复杂的上下文对象层次。
**在具体的策略算法上添加自己需要的数据**：这样实现比较简单。但缺点是，跟其他策略实现的风格不一致，其他策略实现都是通过上下文统一获取数据，但这个策略一部分数据来自上下文，一部分数据来自自己，有些不统一；另外，外部使用这些策略方法的时候也不一样了，难以以一个统一的方法来动态切换策略算法。

两种实现各有优劣，要具体情况具体分析。

### 实际场景3：日志记录
---
上面的两个场景中，均是由客户端来选择具体的算法，现在这个例子是根据上下文来选择具体的策略算法。
实际场景：在一个系统中，把日志记录到数据库和日志文件中（可以看做两种策略），然后再运行期间根据需要进行动态地切换。
首先，定义策略接口：
```
public interface LogStrategy {

    void log(String message);
}
```
然后，两个实现类：
```
//数据库记录
public class DbLog implements LogStrategy {
    @Override
    public void log(String message) {
        System.out.println("将" + message + "记录到数据库");
    }
}

//文件记录
public class FileLog implements LogStrategy {
    @Override
    public void log(String message) {
        System.out.println("将" + message + "记录到文件");
    }
}
```
最后，上下文：
```
public class LogContext {

    /**
     * 记录日志的方法，提供给客户端使用
     *
     * 在上下文中，自行实现对具体策略的选择
     *
     * @param message
     */
    public void log(String message){

        //优先选用策略：记录到数据库
        LogStrategy strategy = new DbLog();
        try {
            strategy.log(message);
        }catch (Exception e){
            //数据库记录日志出错，就记录到为日志文件中
            strategy = new FileLog();
            strategy.log(message);
        }
    }
}
```
好了，客户端可以调用了，相比之前的代码比较简单，因为策略选择的代码在策略上下文中实现，而不是让客户端来选择：
```
public class Client {
    public static void main(String[] args){
        LogContext context = new LogContext();
        context.log("记录日志");
    }
}
```


### 优缺点
---
* 优点
  * 策略模式的功能就是定义一系列算法，实现让这些算法可以相互替换；
  * 避免使用多重条件判断；
  * 扩展性良好，只需要增加新的策略实现类，然后在使用策略的地方选择使用这个新的策略就可以了。
* 缺点
  * 增加了对象数目，由于策略模式把每个具体的策略实现都单独封装为类，如果备选的策略很多，那么对象的数目就会很客观；
  * 客户必须了解每种策略的不同，因为有时候需要让客户端来选择具体使用哪一个策略。
  * 只适合扁平的算法结构。策略模式的一系列算法地位是平等的，而且在运行时刻只有一个算法被使用，这就限制了算法使用的层级，使用的时候不能嵌套使用。对于出现需要嵌套的使用多个算法的情况，比如折上折、折后券等业务的实现，需要组合或者嵌套多个算法的情况，可以考虑使用装饰模式，或者变形的职责链，或者AOP等方式来实现。

### 策略模式的本质
---
如果没有上下文，策略模式就回到了最基本的接口和实现了。上下文的意义在于，客户端不必直接与具体的策略交互，上下文还可以提供一些公共功能或是存储相关状态，减小客户端使用的难度。
策略模式的本质在于：分离算法，选择实现。
策略模式很好地体现了设计原则中的开闭原则：通过把一系列可变的算法进行封装，并定义出合理的结构，使得在系统出现新的算法的时候，很容易把新的算法加入到已有的系统中，而已有的实现不需要做任何修改。

----
内容绝大部分摘抄自《研磨设计模式》


