# 访问者模式

**访问者模式（Visitor）：**表示一个作用于某对象结构中的各元素的操作。它可以使你在不改变各元素类的前提下，定义作用于这些元素上的新操作。

**使用场景：**1、对象结构中对象对应的类很少改变，但经常需要在此对象结构上定义新的操作。 2、需要对一个对象结构中的对象进行很多不同的并且不相关的操作，而需要避免让这些操作"污染"这些对象的类，也不希望在增加新操作时修改这些类。

**访问者模式中的角色**

* **Visitor：**访问者接口，为所有的访问者对象声明一个visit方法，用来代表为对象结构添加的功能，理论上可以代表任意的功能。
* **ConcreteVisitor：**具体访问者实现对象，实现要真正被添加到对象结构中的功能。
* **Element：**抽象的元素对象，对象结构的顶层接口，定义接受访问的操作。
* **ConcreteElement：**具体元素对象，对象结构中具体的对象，也是被访问的对象，通常会回调访问者的真实功能，同时开放自身的数据供访问者使用。
* **ObjectStructure：**通常包含多个被访问的对象，它可以遍历这多个被访问的对象，也可以让访问者访问它的元素。可以是一个复合或是一个集合，如一个列表或无序集合。

**访问者模式结构代码**

Visitor：访问者接口

```java
public interface Visitor {

    //访问元素A，相当于给元素A增加功能
    void visitConcreteElementA(ConcreteElementA elementA);

    //访问元素B，相当于给元素B增加功能
    void visitConcreteElementB(ConcreteElementB elementB);

}
```

Element：抽象元素类

```java
public abstract class Element {

    public abstract void accept(Visitor visitor);
}
```

ConcreteElement

```java
//具体元素对象
public class ConcreteElementA extends Element{
    @Override
    public void accept(Visitor visitor) {
        visitor.visitConcreteElementA(this);
    }
}
//具体元素对象
public class ConcreteElementB extends Element{
    @Override
    public void accept(Visitor visitor) {
        visitor.visitConcreteElementB(this);
    }
}
```

ConcreteVisitor

```java
//实现某一功能的具体访问者
public class ConcreteVisitor1 implements Visitor{

    @Override
    public void visitConcreteElementA(ConcreteElementA elementA) {

    }

    @Override
    public void visitConcreteElementB(ConcreteElementB elementB) {

    }
}
//实现某一功能的具体访问者
public class ConcreteVisitor2 implements Visitor{

    @Override
    public void visitConcreteElementA(ConcreteElementA elementA) {

    }

    @Override
    public void visitConcreteElementB(ConcreteElementB elementB) {

    }
}
```

ObjectStructure：元素集合类

```java
public class ObjectStructure {

    private Collection<Element> collection = new ArrayList<>();

    public void addElement(Element e){
        this.collection.add(e);
    }
    //提供给客户端使用
    public void handleRequest(Visitor visitor){
        for (Element e : collection){
            e.accept(visitor);
        }
    }
}
```

Client

```java
public class Client {

    public static void main(String[] args) {

        ObjectStructure os = new ObjectStructure();
        os.addElement(new ConcreteElementA());
        os.addElement(new ConcreteElementB());

        Visitor visitor = new ConcreteVisitor1();
        os.handleRequest(visitor);

    }
}
```

