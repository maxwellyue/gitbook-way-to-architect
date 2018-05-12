# 桥接模式

---

桥接模式\(Bridge Pattern\)的定义：将抽象部分与它的实现部分分离，使它们都可以独立地变化。

如果软件系统中某个类存在两个独立变化的维度，如果采用多层继承的方案，则会使类的数目急速增加，且扩展麻烦；此时，如果将这两个维度分离出来，使两者可以独立发展，就可以将类之间的继承关系变为对象组合关系，使得系统更加灵活，并易于扩展。![](/assets/屏幕快照 2018-05-13 上午1.19.16.png)

**桥接模式中的角色**

* Abstraction（抽象类）：定义抽象类的接口，一般是抽象类而不是接口，其中定义了一个Implementor（实现类接口）类型的对象并维护该对象，它与Implementor之间具有关联关系，它既可以包含抽象业务方法，也可以包含具体业务方法。

* RefinedAbstraction（扩充抽象类）：继承Abstraction，通常情况下是具体类，实现在Abstraction中声明的抽象业务方法，在RefinedAbstraction中可以调用在Implementor中定义的业务方法。

* Implementor（实现类接口）：定义实现类的接口，一般而言，Implementor接口仅提供基本操作，而Abstraction定义的接口可能会做更多更复杂的操作。Implementor接口对这些基本操作进行了声明，而具体实现交给其子类。通过关联关系，在Abstraction中不仅拥有自己的方法，还可以调用到Implementor中定义的方法，使用关联关系来替代继承关系。

* ConcreteImplementor（具体实现类）：具体实现Implementor接口，在不同的ConcreteImplementor中提供基本操作的不同实现，在程序运行时，ConcreteImplementor对象将替换其父类对象，提供给抽象类具体的业务操作方法。

**桥接模式的结构代码**

Implementor

```java
pulbic interface Implementor{
    void operationImpl();
}
```

Abstraction

```java
public abstract class Abstraction{

    private Implementor impl;
    public Abstraction(Implementor impl){
        this.impl = impl;
    }

    public void operation(){
        impl.operationImpl();
    }
}
```

ConcreteImplementor

```java
public class ConcreteImplementorA implements Implementor{
    public void operationImpl(){
        ......
    }
}

public class ConcreteImplementorB implements Implementor{
    public void operationImpl(){
        ......
    }
}
```

RefinedAbstraction

```java
public class RefinedAbstractionA extends Abstraction{

    public RefinedAbstractionA(Implementor impl){
        super(impl);
    }
    
    public void otherOperation(){
        
    }
}
public class RefinedAbstractionB extends Abstraction{

    public RefinedAbstractionB(Implementor impl){
        super(impl);
    }
    
    public void otherOperation(){
        
    }
}
```









