# 观察者模式

---

观察者模式=发布-订阅模式

Android中常用的[EventBus](https://github.com/greenrobot/EventBus)以及[Google Guava](https://github.com/google/guava)包中的[event bus](https://github.com/google/guava/tree/master/guava/src/com/google/common/eventbus)等都是对观察者模式的实现。

个人理解，从广义上讲，消息队列也是一种观察者模式的实现。

**观察者模式的本质就是触发联动**：在修改目标对象的状态的时候，就会触发相应的通知，然后会循环调用所有观察者对象相应的方法。



**下面，我们实现以下功能：**

当有新员工入职时，为其开通公司内部系统账号，并为其制作工牌；

当有员工离职时，为其关闭公司内部系统账号，并收回其工牌；



**示例1：使用JDK中已有的观察者模式实现**

目标类

```java
public class Employee extends Observable {

    private String name;

    private Status status;
    
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Status getStatus() {
        return status;
    }

    public void setStatus(Status status) {
        this.status = status;
        //有员工变动，则手动设置目标变化
        this.setChanged();
        //说明有员工入职
        if(status == Status.NEW){
            this.notifyObservers(this);
        }else if(status == Status.LEAVE){
            this.notifyObservers(this);
        }
    }

    enum Status{
        NEW, LEAVE
    }
}
```

观察者1

```java
public class AccountObserver implements Observer {

    @Override
    public void update(Observable o, Object arg) {
        Employee employee = (Employee)o;
        Employee.Status status = employee.getStatus();
        if(status == Employee.Status.NEW){
            System.out.println(String.format("create account for [employee : %s]", employee.getName()));
        }else if(status == Employee.Status.LEAVE){
            System.out.println(String.format("delete account for [employee : %s]", employee.getName()));
        }
    }
    
}
```

观察者2

public class WorkCardObserver implements Observer {



    @Override

    public void update\(Observable o, Object arg\) {

        Employee employee = \(Employee\)o;

        Employee.Status status = employee.getStatus\(\);

        if\(status == Employee.Status.NEW\){

            System.out.println\(String.format\("make an work card for \[employee : %s\]", employee.getName\(\)\)\);

        }else if\(status == Employee.Status.LEAVE\){

            System.out.println\(String.format\("reclaim \[employee : %s\] work card", employee.getName\(\)\)\);

        }

    }

}

验证

```java
public class Client {
    
    public static void main(String[] args){

        Employee employee = new Employee();
        employee.setName("xiaoming");

        employee.addObserver(new AccountObserver());
        employee.addObserver(new WorkCardObserver());

        employee.setStatus(Employee.Status.NEW);
        System.out.println("1 year later...");
        employee.setStatus(Employee.Status.LEAVE);

    }
    
}
```

输出如下：

```
make an work card for [employee : xiaoming]
create account for [employee : xiaoming]
1 year later...
reclaim [employee : xiaoming] work card
delete account for [employee : xiaoming]
```



**示例2：使用Google Guava中的event bus实现**

目标

```java
public class Employee {

    private String name;

    private Status status;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Status getStatus() {
        return status;
    }

    public void setStatus(Status status) {
        this.status = status;
    }

    enum Status {
        NEW, LEAVE
    }
}
```

订阅者或观察者

```java
public class EmployeeEventSubscriber {

    @Subscribe
    public void handleNew(Employee employee) {
        System.out.println(String.format("create account for [employee : %s]", employee.getName()));
        System.out.println(String.format("make an work card for [employee : %s]", employee.getName()));
    }

    @Subscribe
    public void handleLeave(Employee employee) {
        System.out.println(String.format("delete account for [employee : %s]", employee.getName()));
        System.out.println(String.format("reclaim [employee : %s] work card", employee.getName()));
    }
}
```

验证

```java
public class Client {

    public static void main(String[] args){
        EventBus eventBus = new EventBus();
        eventBus.register(new EmployeeEventSubscriber());

        Employee employee = new Employee();
        employee.setName("xiaoming");
        employee.setStatus(Employee.Status.NEW);
        eventBus.post(employee);

        System.out.println("1 year later...");
        employee.setStatus(Employee.Status.LEAVE);
        eventBus.post(employee);
    }

}
```

输出

```
delete account for [employee : xiaoming]
reclaim [employee : xiaoming] work card
create account for [employee : xiaoming]
make an work card for [employee : xiaoming]
1 year later...
delete account for [employee : xiaoming]
reclaim [employee : xiaoming] work card
create account for [employee : xiaoming]
make an work card for [employee : xiaoming]
```

guava的优势：①不必实现接口；②处理事件或状态变化的方法命名没有约束，而JDK中只能是update

对guava中event bus的源码简单分析，点击[这里](https://github.com/maxwellyue/JavaLanguage/tree/master/event/src/main/java/com/maxwell/learning/event/eventbus)。



### 参考

---

《研磨设计模式--第12章 观察者模式》

菜鸟教程：[观察者模式](http://www.runoob.com/design-pattern/observer-pattern.html)

  


