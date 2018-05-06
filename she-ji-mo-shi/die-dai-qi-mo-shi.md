# 迭代器模式

---

作为Java程序员，我想下面的代码每个人都很熟悉：

```java
List<String> list = Arrays.asList("a", "b", "c", "d", "e");
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()){
    iterator.next();
}
```

这就是迭代器模式！！！

**迭代器模式\(Iterator Pattern\)：提供一种方法来访问聚合对象，而不用暴露这个对象的内部表示。**

**迭代器模式中的角色**

* Iterator（迭代器）：定义访问和遍历元素的方法，如first\(\)、next\(\)、hasNext\(\)、currentItem\(\)等；

* ConcreteIterator（具体迭代器）：实现Iterator接口；

* Aggregate（抽象聚合类）：存储和管理元素对象，并声明一个createIterator\(\)方法用于创建一个迭代器对象；

* ConcreteAggregate（具体聚合类）：实现存储和管理元素对象的方法，并实现createIterator\(\)方法；

**迭代器结构代码**

`Iterator`

```java
public interface Iterator {  
    void first(); 
    void next();
    boolean hasNext();
    Object currentItem();
}
```

> 抽象迭代器接口的设计非常重要，
>
> ①需要充分满足各种遍历操作的要求，尽量为各种遍历方法都提供声明（ 如果在具体迭代器中为聚合对象增加新的遍历方法，则必须修改抽象迭代器和具体迭代器的源代码，这将违反“开闭原则”，因此在设计时要考虑全面，避免之后修改接口。  ）
>
> ②不能包含太多方法，接口中方法太多将给子类的实现带来麻烦。
>
> 因此，可以使用抽象类来设计抽象迭代器，在抽象类中为每一个方法提供一个空的默认实现。

`ConcreteIterator`

```java
public class ConcreteIterator implements Iterator {  
    
    //具体聚合对象的引用   
    private ConcreteAggregate objects;   
    //定义一个游标，记录当前访问位置      
    private int cursor;
    
    public ConcreteIterator(ConcreteAggregate objects) {  
        this.objects=objects;  
    }  

    public void first() {  ......  }  

    public void next() {  ......  }  

    public boolean hasNext() {  ......  }  

    public Object currentItem() {  ......  }  
}
```

Aggregate

```java
public interface Aggregate {  
    Iterator createIterator();  
    //other Aggragate itself functional method
}
```

ConcreteAggregate

```java
public class ConcreteAggregate implements Aggregate {    
      
    public Iterator createIterator() {  
          return new ConcreteIterator(this);  
    }  
    
    //other Aggragate itself functional method implements
}
```

TODO：理解迭代器模式中具体聚合类与具体迭代器类之间存在的依赖关系和关联关系。

















