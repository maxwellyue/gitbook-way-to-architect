# 中介者模式

中介者模式\(Mediator Pattern\)：引入一个“第三者”来降低现有系统中类之间的耦合度，由这个“第三者”来封装并协调原有组件两两之间复杂的引用关系，使之成为一个松耦合的系统。

加入中介者后的系统前后对比（[图片来源](http://www.cnblogs.com/chenssy/p/3348520.html)）![](../../../.gitbook/assets/wechatimg7.jpeg)

> [迪米特法则](https://baike.baidu.com/item/迪米特法则)：又叫作最少知识原则（Least Knowledge Principle 简写LKP），
>
> 迪米特法则的初衷在于降低类之间的耦合由于每个类尽量减少对其他类的依赖，因此，很容易使得系统的功能模块功能独立，相互之间不存在（或很少有）依赖关系。
>
> 迪米特法则不希望类之间建立直接的联系。如果真的有需要建立联系，也希望能通过中介者来转达。
>
> 因此，应用迪米特法则有可能造成的一个后果就是：系统中存在大量的中介类，这些类之所以存在完全是为了传递类之间的相互调用关系——这在一定程度上增加了系统的复杂度。
>
> 中介者模式是迪米特法则的典型应用（另外，还有门面模式）。

**中介者模式的本质：解耦**

**中介者模式中的角色**

* Mediator（抽象中介者）：定义各组件（或同事对象）之间通信的方法
* ConcreteMediator（具体中介者）：实现抽象中介者，以此协调各个同事对象来实现协作行为
* Colleague（抽象同事类）：定义各个同事类的公有方法，并声明一些抽象方法来供子类实现，同时维持对抽象中介者类的引用
* ConcreteColleague（具体同事类）：抽象同事类的子类；同事对象通过中介者来间接完成与其他同事类的通信

> [何为同事类](https://blog.csdn.net/zhengzhb/article/details/7430098)：
>
> 假如有两个类A和B，类中各有一个数字，并且要保证类B中的数字永远是类A中数字的100倍。也就是说，当修改类A的数时，将这个数字乘以100赋给类B，而修改类B时，要将数除以100赋给类A。
>
> 类A类B互相影响，就A、B就相互称彼此为同事类。

**中介者模式的结构代码**

Mediator：抽象中介者，一般是一个抽象类

```java
public abstract class Mediator {  

    protected ArrayList<Colleague> colleagues; //用于存储同事对象  

    //注册方法，用于增加同事对象  
    public void register(Colleague colleague) {  
        colleagues.add(colleague);  
    }  

    //声明抽象的业务方法  
    public abstract void operation();  
}
```

`ConcreteMediator`：具体中介者

```java
public class ConcreteMediator extends Mediator {      

    //实现业务方法，封装同事之间的调用  
    public void operation() {  
        ...

        //通过中介者调用同事类的方法  
        ((Colleague)(colleagues.get(0))).method1(); 

        ...
    }  
}
```

Colleague：抽象同事类，维持了一个抽象中介者的引用，用于调用中介者的方法

```java
public abstract class Colleague {  

    //维持一个抽象中介者的引用  
    protected Mediator mediator; 

    public Colleague(Mediator mediator) {  
        this.mediator=mediator;  
    }

    //定义自身方法，处理自己的行为  
    public abstract void method1(); 

    //定义依赖方法，与中介者进行通信  
    public void method2() {  
        mediator.operation();  
    }  
}
```

ConcreteColleague：具体同事类

```java
public class ConcreteColleague extends Colleague {  

    public ConcreteColleague(Mediator mediator) {  
        super(mediator);  
    }  

    //实现自身方法  
    public void method1() {  
        ......  
    }  
}
```

Colleague类的代码中，自身方法（Self-Method）与依赖方法\(Depend-Method\)的区别：假如类A在修改自身某属性后，必须要去修改类B的有个属性，那么类A中就要有两个方法，`updateSelf()`以及`updateB()`，那么对A而言，`updateSelf()`就是自身方法，`updateB()`就是类A的依赖方法。不引入中介者的话，因为有`updateB()`这个操作，类A就必须持有类B的引用，而引入中介者后，这个依赖方法就可以交给中介者去做（有没有很像门面模式？？？）。在这种模式下，每个对象都会在自己的状态改变时通知中介者；每个对象都会对中介者所发出的请求作出回应（有没有很像观察者模式？？？）。

**举个例子**：现在你想在电脑上播放一张CD，需要光驱读取，光驱读取后将数据交给CPU处理，CPU处理完之后，将CD中的视频信息交给显卡去显示，将音频信息交给声卡去播放。而主板相当于中介者，光驱、CPU、显卡、声卡之间的交互都通过主板来完成。

Colleague：抽象同事类

```java
public abstract class Colleague {  

    //维持一个抽象中介者的引用  
    private Mediator mediator; 

    public Colleague(Mediator mediator) {  
        this.mediator=mediator;  
    }

    public Mediator getMediator(){
        return this.mediator;
    }

}
```

ConcreteColleague：有四个，分别是光驱、CPU、显卡、声卡

```java
public class CDDriver extends Colleague{

    public CDDriver(Mediator mediator){
        super(mediator);
    }

    //光驱读取的数据
    private String data;

    public String getDate(){
        return this.data;
    }

    public void readCD(){
        //模拟CD数据
        this.data="[声音信息...],[视频信息...]";
        //通过中介调用依赖方法：通知主板，自己读取到了数据data
        this.getMediator.change(this);
    }
}
```

```java
public class CPU extends Colleague{

    public CPU(Mediator mediator){
        super(mediator);
    }

    //解析出的视频数据
    private String vedioData;
    //解析出的音频数据
    private String soundDate;

    public String getVedioDate(){
        return this.vedioData;
    }
    public String getSound(){
        return this.sooundData;
    }

    //解析数据
    public void resolveData(String cdData){
        //解析CDDriver读取的原始CD数据
        String[] data = cdData.split(",");
        this.soundDate = data[0];
        this.vedioDate = data[1];
        //通过中介调用依赖方法：通知主板，自己解析数据了
        this.getMediator.change(this);
    }
}
```

```java
public class VedioCard extends Colleague{

    public VedioCard(Mediator mediator){
        super(mediator);
    }

    public void showVedio(String vedioData){
        System.out.print("正在观看：" + vedioData);
    }
}
```

```java
public class SoundCard extends Colleague{

    public SoundCard(Mediator mediator){
        super(mediator);
    }

    public void playSound(String soundData){
        System.out.print("画外音：" + soundData);
    }
}
```

Mediator

```java
public interface Mediator{
    void changed(Colleague colleague);
}
```

ConcreteMediator，中介的具体实现，即主板

```java
public class MotherBoard implements Mediator{

    private CDDriver cdDriver;
    private CPU cpu;
    private VedioCard vedioCard;
    private SoundCard soundCard;

    public void setCDDriver(CDDriver driver){
        this.cdDriver = driver;
    }    

    public void setCPU(CPU cpu){
        this.cpu = cpu;
    }

    public void setVedioCard(VedioCard vedioCard){
        this.vedioCard = vedioCard;
    }

    public void setSoundCard(SoundCard soundCard){
        this.soundCard = soundCard;
    }

    public void changed(Colleague col){
        if(col == this.cdDriver){
            //获取光驱读取的数据
            String data = col.getData();
            //交给CPU解析
            this.cpu.resolveData(data);
        }else if(col == this.cpu){
            //获取CPU解析出的信息
            String vedio = col.getVedioData();
            String sound = col.getSoundData();
            //交给声卡和显卡去显示或播放
            this.vedioCard.showVedio(vedio);
            this.soundCard.playSound(sound);
        }
    }
}
```

Client

```java
public class Client{

    public static void main(String[] args){
        //创建中介
        Mediator mediator = new MotherBoard();
        //创建同事类
        CDDriver driver = new CDDriver(mediator);
        CPU cpu = new CPU(mediator);
        VedioCard vedioCard = new VedioCard(mediator);
        SoundCard soundCard = new SoundCard(mediator);
        //让中介知道所有同事类
        mediator.setCDDriver(driver);
        mediator.serCPU(cpu);
        mediator.setVedioCard(vedioCaard);
        mediator.setSoundCard(soundCard);

        //开始看电影
        driver.readCD();
    }

}
//输出如下
正在播放：[视频信息...]
画外音：[声音信息...]
```

## 参考

[23种设计模式（7）：中介者模式](https://blog.csdn.net/zhengzhb/article/details/7430098)

[设计模式 ——— 中介者模式](https://www.jianshu.com/p/21d27fd06e86)

实例代码来自：《研磨设计模式--第10章 中介者模式（Mediator）》 ****

