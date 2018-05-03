# 观察者模式

---

观察者模式=发布-订阅模式

Android中常用的[EventBus](https://github.com/greenrobot/EventBus)以及[Google Guava](https://github.com/google/guava)包中的[event bus](https://github.com/google/guava/tree/master/guava/src/com/google/common/eventbus)等都是对观察者模式的实现。

个人理解，从广义上讲，消息队列也是一种观察者模式的实现。

**观察者模式的本质就是触发联动**：在修改目标对象的状态的时候，就会触发相应的通知，然后会循环调用所有观察者对象相应的方法。



# Java中观察者模式的实现

---





