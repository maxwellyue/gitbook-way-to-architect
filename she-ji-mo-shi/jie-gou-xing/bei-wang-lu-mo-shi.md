# 备忘录模式

---

备忘录模式：在不破坏封装的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，这样可以在以后将对象恢复到原先保存的状态。（在操作一个对象前，把这个对象临时保存一份，以便将来恢复）。

从字面意思来看，备忘录，首先要有备忘的内容，其次考虑把这个要备忘的内容存在哪。

备忘录最佳应用场景：当应用需要提供撤销功能时！！！如下棋游戏中的“悔棋”，Office Word的撤销操作等等。

备忘录模式中的三个角色

* Originator：要备忘的内容就是Originator这个对象的状态

* Memento：备忘的内容要存储在哪？没错，就是要存在Memento这个对象上

* Caretaker：备忘的内容存在Memento上，Caretaker负责保管Memento，并在需要的时候，提供之前保管的Memento。

**备忘录模式代码演示**

`Originator`

```java
public class Originator {  

    private String state;  

    public Originator(){}  

　　// 创建一个备忘录对象  
    public Memento createMemento() {  
　　　　return new Memento(this);  
    }  

　　 // 根据备忘录对象恢复原发器状态  
    public void restoreMemento(Memento m) {  
　　　　 state = m.state;  
    }  

    public void setState(String state) {  
        this.state=state;  
    }  

    public String getState() {  
        return this.state;  
    }  
}
```

`Memento`

```java
class Memento {  

    private String state;  

    public Memento(Originator o) {  
　　　　state = o.getState();  
    }  

    public void setState(String state) {  
        this.state=state;  
    }  

    public String getState() {  
        return this.state;  
    }  
}
```

> Memento通常提供与Originator相对应的属性（可以是全部，也可以是部分）用于存储原发器的状态；
>
> 除了Originator类，不允许其他类来调用备忘录类Memento的构造函数与相关方法，如果不考虑封装性，允许其他类调用setState\(\)等方法，将导致在备忘录中保存的历史状态发生改变，通过撤销操作所恢复的状态就不再是真实的历史状态，备忘录模式也就失去了本身的意义。
>
> 在使用Java语言实现备忘录模式时，一般通过将Memento类与Originator类定义在同一个包\(package\)中来实现封装，在Java语言中可使用默认访问标识符来定义Memento类，即保证其包内可见。只有Originator类可以对Memento进行访问，而限制了其他类对Memento的访问。

`Caretaker`

```java
public class Caretaker {  

    private Memento memento;  

    public Memento getMemento() {  
        return memento;  
    }  

    public void setMemento(Memento memento) {  
        this.memento=memento;  
    }  
}
```

客户端

```java
public class Client {

    public static void main(String[] args) {
        //创建Caretaker
        Caretaker ck = new Caretaker();
        //初始状态
        Originator obj = new Originator();
        obj.setStatus("status--0");
        //备忘
        Memento memento = obj.createMemento();
        ck.setMemento(memento);
        //改变状态
        obj.setStatus("status--1");
        //恢复状态
        obj.restoreMemento(ck.getMemento());
    }
}
```

**这么看，备忘录其实就是在操作对象之前，先将现在的状态（可能是全部属性，可能是部分属性）保存起来，等对象状态改变之后，想要恢复，在设置为之前保存的状态**。

**看一个更具体的例子：实现下象棋时候的悔棋功能（**[**代码来源**](https://gof.quanke.name/%E6%92%A4%E9%94%80%E5%8A%9F%E8%83%BD%E7%9A%84%E5%AE%9E%E7%8E%B0%E2%80%94%E2%80%94%E5%A4%87%E5%BF%98%E5%BD%95%E6%A8%A1%E5%BC%8F%EF%BC%88%E4%B8%89%EF%BC%89.html)**）**

Originator，就是棋子

```java
public class Chessman {  

    private String label;  
    private int x;  
    private int y;  

    public Chessman(String label,int x,int y) {  
        this.label = label;  
        this.x = x;  
        this.y = y;  
    }  

    public void setLabel(String label) {  
        this.label = label;   
    }  

    public void setX(int x) {  
        this.x = x;   
    }  

    public void setY(int y) {  
        this.y = y;   
    }  

    public String getLabel() {  
        return (this.label);   
    }  

    public int getX() {  
        return (this.x);   
    }  

    public int getY() {  
        return (this.y);   
    }  

    //保存状态  
    public ChessmanMemento save() {  
        return new ChessmanMemento(this.label,this.x,this.y);  
    }  

    //恢复状态  
    public void restore(ChessmanMemento memento) {  
        this.label = memento.getLabel();  
        this.x = memento.getX();  
        this.y = memento.getY();  
    }  
}
```

Memento：

```java
class ChessmanMemento {  

    private String label;  
    private int x;  
    private int y;  

    public ChessmanMemento(String label,int x,int y) {  
        this.label = label;  
        this.x = x;  
        this.y = y;  
    }  

    public void setLabel(String label) {  
        this.label = label;   
    }  

    public void setX(int x) {  
        this.x = x;   
    }  

    public void setY(int y) {  
        this.y = y;   
    }  

    public String getLabel() {  
        return (this.label);   
    }  

    public int getX() {  
        return (this.x);   
    }  

    public int getY() {  
        return (this.y);   
    }     
}
```

Caretaker

```java
class MementoCaretaker {  
    private ChessmanMemento memento;  

    public ChessmanMemento getMemento() {  
        return memento;  
    }  

    public void setMemento(ChessmanMemento memento) {  
        this.memento = memento;  
    }  
}
```

Client

```java

public class Client {  
    public static void main(String args[]) {  
        MementoCaretaker mc = new MementoCaretaker();  
        
        Chessman chess = new Chessman("车",1,1);  
        display(chess);  
        
        mc.setMemento(chess.save()); //保存状态       
        chess.setY(4);  
        display(chess);  
        
        mc.setMemento(chess.save()); //保存状态  
        display(chess);  
        
        chess.setX(5);  
        display(chess);  
        
        System.out.println("******悔棋******");     
        
        chess.restore(mc.getMemento()); //恢复状态  
        display(chess);  
    }

    public static void display(Chessman chess) {  
        System.out.println("棋子" + chess.getLabel() + "当前位置为：" + "第" + chess.getX() + "行" + "第" + chess.getY() + "列。");  
    } 
}

//输出如下
棋子车当前位置为：第1行第1列。
棋子车当前位置为：第1行第4列。
棋子车当前位置为：第1行第4列。
棋子车当前位置为：第5行第4列。
******悔棋******
棋子车当前位置为：第1行第4列。
```

上面的代码中，我们只能后悔一次。假如我们需要实现多次悔棋，则只需要修改备忘录管理类（[代码来源](https://gof.quanke.name/%E6%92%A4%E9%94%80%E5%8A%9F%E8%83%BD%E7%9A%84%E5%AE%9E%E7%8E%B0%E2%80%94%E2%80%94%E5%A4%87%E5%BF%98%E5%BD%95%E6%A8%A1%E5%BC%8F%EF%BC%88%E5%9B%9B%EF%BC%89.html)）：

```java
class MementoCaretaker {  

    //定义一个集合来存储多个备忘录  
    private ArrayList mementolist = new ArrayList();  

    public ChessmanMemento getMemento(int i) {  
        return (ChessmanMemento)mementolist.get(i);  
    }  

    public void setMemento(ChessmanMemento memento) {  
        mementolist.add(memento);  
    }  
}
```

客户端

```java
public class Client {  

    private static int index = -1; //定义一个索引来记录当前状态所在位置  
    private static MementoCaretaker mc = new MementoCaretaker();  

    public static void main(String args[]) {  
        Chessman chess = new Chessman("车",1,1);  
        play(chess);          
        chess.setY(4);  
        play(chess);  
        chess.setX(5);  
        play(chess);      
        undo(chess,index);  
        undo(chess,index);    
        redo(chess,index);  
        redo(chess,index);  
    }  

    //下棋  
    public static void play(Chessman chess) {  
        mc.setMemento(chess.save()); //保存备忘录  
        index ++;   
        System.out.println("棋子" + chess.getLabel() + "当前位置为：" + "第" + chess.getX() + "行" + "第" + chess.getY() + "列。");  
    }  
    //悔棋  
    public static void undo(Chessman chess,int i) {  
        System.out.println("******悔棋******");  
        index --;   
        chess.restore(mc.getMemento(i-1)); //撤销到上一个备忘录  
        System.out.println("棋子" + chess.getLabel() + "当前位置为：" + "第" + chess.getX() + "行" + "第" + chess.getY() + "列。");  
    }  
    //撤销悔棋  
    public static void redo(Chessman chess,int i) {  
        System.out.println("******撤销悔棋******");   
        index ++;   
        chess.restore(mc.getMemento(i+1)); //恢复到下一个备忘录  
        System.out.println("棋子" + chess.getLabel() + "当前位置为：" + "第" + chess.getX() + "行" + "第" + chess.getY() + "列。");  
    }  
}

//输出如下：
棋子车当前位置为：第1行第1列。
棋子车当前位置为：第1行第4列。
棋子车当前位置为：第5行第4列。
******悔棋******
棋子车当前位置为：第1行第4列。
******悔棋******
棋子车当前位置为：第1行第1列。
******撤销悔棋******
棋子车当前位置为：第1行第4列。
******撤销悔棋******
棋子车当前位置为：第5行第4列。
```

**总结**

①有时候，需要备忘Originator的全部信息，此时，可以考虑结合原型模式（Java中的clone\(\)）来获取备忘录，而不用单独写一个备忘录类。

②为解决备忘录Memento的可见性问题，可以使用内部类的方式，将Memento写为Originator的内部类；此时，Caretaker无法访问Memento，所以要定义一个标识接口（如IMemento），让Memento实现该接口，而Caretaker则持有该接口（而不是具体的Memento）。

③TODO ：多撤销问题的深入（待到实际工作中用到再考虑）。

④Originator，翻译为原发器，原发器是什么？

④对后端开发，使用备忘录模式的场景我没想起来...











### 参考

---

[设计模式之备忘录模式](https://www.jianshu.com/p/cd29a2ca97c5)

[《JAVA与模式》之备忘录模式](http://www.cnblogs.com/java-my-life/archive/2012/06/06/2534942.html) ：讲的很全面



