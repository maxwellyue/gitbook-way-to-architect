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

---

TODO：通过实例理解浅拷贝和深拷贝

