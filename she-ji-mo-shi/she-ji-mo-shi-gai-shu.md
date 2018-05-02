
### 设计模式有啥用
---
设计模式是一套被反复使用的、多数人知晓的、经过分类编目的、代码设计经验的总结。使用设计模式是为了重用代码、让代码更容易被他人理解、保证代码可靠性。

设计模式已经经历了很长一段时间的发展，它们提供了软件开发过程中面临的一般问题的最佳解决方案。学习这些模式有助于经验不足的开发人员通过一种简单快捷的方式来学习软件设计。

总体思想是：高内聚、低耦合。


### 设计模式的原则：SOLID
---

简称|全称|含义
--|--|--
SRP|	The Single Responsibility Principle	|单一责任原则
OCP|	The Open Closed Principle	|开放封闭原则
LSP|	The Liskov Substitution Principle|	里氏替换原则
ISP	|The Interface Segregation Principle	|接口分离原则
DIP	|The Dependency Inversion Principle|	依赖倒置原则

* **单一职责**：一个类或对象只体现一类特征或只干一件事或一组相关的事；
* **开放封闭**：对扩展开放，对修改关闭。在程序需要进行扩展的时候，尽量不要去修改原有的代码。
* **里氏替换**：任何基类可以出现的地方，子类一定可以出现。LSP 是继承复用的基石，只有当派生类可以替换掉基类，且软件单位的功能不受到影响时，基类才能真正被复用，而派生类也能够在基类的基础上增加新的行为。里氏代换原则是对开闭原则的补充。实现开闭原则的关键步骤就是抽象化，而基类与子类的继承关系就是抽象化的具体实现，所以里氏代换原则是对实现抽象化的具体步骤的规范。
* **接口分离**：使用多个隔离的接口，而不是使用单个接口。
* **依赖倒转**：面对接口编程，依赖于抽象而不依赖于具体。这是是开闭原则的基础。
*还有另外两个：*
* 迪米特法则，又称**最少知道**原则（Demeter Principle）：一个实体应当尽量少地与其他实体之间发生相互作用，使得系统功能模块相对独立。
* **合成复用**（Composite Reuse Principle）：尽量使用合成/聚合的方式，而不是使用继承。



### 设计模式的分类
---

###### 创建型模式
>这些设计模式提供了一种在创建对象的同时隐藏创建逻辑的方式，而不是使用 new 运算符直接实例化对象。这使得程序在判断针对某个给定实例需要创建哪些对象时更加灵活。	
* **工厂模式（Factory Pattern）**
* **抽象工厂模式（Abstract Factory Pattern）**
* **单例模式（Singleton Pattern）**
* **建造者模式（Builder Pattern）**
* **原型模式（Prototype Pattern）**

###### 结构型模式
>这些设计模式关注类和对象的组合。继承的概念被用来组合接口和定义组合对象获得新功能的方式。	
* **适配器模式（Adapter Pattern）**
* **桥接模式（Bridge Pattern）**
* **过滤器模式（Filter、Criteria Pattern）**
* **组合模式（Composite Pattern）**
* **装饰器模式（Decorator Pattern）**
* **外观模式（Facade Pattern）**
* **享元模式（Flyweight Pattern）**
* **代理模式（Proxy Pattern）**

###### 行为型模式
>这些设计模式特别关注对象之间的通信。	
* **责任链模式（Chain of Responsibility Pattern）**
* **命令模式（Command Pattern）**
* **解释器模式（Interpreter Pattern）**
* **迭代器模式（Iterator Pattern）**
* **中介者模式（Mediator Pattern）**
* **备忘录模式（Memento Pattern）**
* **观察者模式（Observer Pattern）**
* **状态模式（State Pattern）**
* **空对象模式（Null Object Pattern）**
* **策略模式（Strategy Pattern）**
* **模板模式（Template Pattern）**
* **访问者模式（Visitor Pattern）**



