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

迭代器模式\(Iterator Pattern\)：提供一种方法来访问聚合对象，而不用暴露这个对象的内部表示。

迭代器模式中的角色

* Iterator（迭代器）：定义访问和遍历元素的方法，如first\(\)、next\(\)、hasNext\(\)、currentItem\(\)等；

* ConcreteIterator（具体迭代器）：实现Iterator接口；

* Aggregate（抽象聚合类）：存储和管理元素对象，并声明一个createIterator\(\)方法用于创建一个迭代器对象；

* ConcreteAggregate（具体聚合类）：实现存储和管理元素对象的方法，并实现createIterator\(\)方法；



