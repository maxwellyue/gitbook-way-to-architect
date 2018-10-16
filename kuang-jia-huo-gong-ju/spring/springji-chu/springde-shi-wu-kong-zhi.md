## 注解位置

---

@Transactional可以放在两个位置

**Service层的实现类上**：该类中所有方法都进行事务处理

```java
@Service
@Transactional
public class OrgServiceImpl implements OrgService {
          ...
}
```

**Service层的实现类的具体方法上**：该方法进行事务处理

```java
@Service
public class OrgServiceImpl implements OrgService {
    ...
    @Override
    @Transactional
    public int addOrg(Org org) {
        ...
    }
    ...
}
```

## @Transactional的属性

---

先看一个表格总结：

| 属性名 | 功 能 描 述 |
| --- | --- |
| readOnly | 设置当前事务是否为只读事务，默认false |
| propagation | 设置事务的传播行为，默认为Propagation.REQUIRED |
| isolation | 设置底层数据库的事务隔离级别。默认值为Isolation.DEFAULT |
| timeout | 设置事务的超时秒数，默认值为-1，永不超时 |
| rollbackFor | 设置需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，则进行事务回滚。 |
| rollbackForClassName | 设置需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，则进行事务回滚。 |
| noRollbackFor | 设置不需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，不进行事务回滚。 |
| noRollbackForClassName | 设置不需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，不进行事务回滚。 |

再来看几个重要属性的具体释义。

### 是否只读（readOnly）

是否是只读事务。

| 取值 | 含义 |
| --- | --- |
| false（默认） | 说明为读写事务 |
| true | 说明为只读事务，对于JDBC而言，只读事务会有一定的速度优化。此时，事务控制的其他配置会采用默认值，事务的隔离级别\(isolation\) 为DEFAULT（采用底层数据源的隔离级别），事务的传播行为\(propagation\)则是REQUIRED，所以还是会有事务存在。 |

[Spring 事务 readOnly 到底是怎么回事](http://www.cnblogs.com/hackem/p/3890656.html)中给出的结论是：

> 1 readonly并不是所有数据库都支持的，不同的数据库下会有不同的结果。  
> 2 设置了readonly后，connection都会被赋予readonly，效果取决于数据库的实现。  
> 3 在ORM中，设置了readonly会赋予一些额外的优化，例如在Hibernate中，会被禁止flush等。

在将事务设置成只读后，相当于将数据库设置成只读数据库，此时若要进行写的操作，会出现错误。如果你使用的是mysql数据库，出现了这样的报错信息，那么很有可能是你在增删改等修改操作的方法上不小心加上了`readOnly=true`。

```java
Caused by: java.sql.SQLException: Connection is read-only. Queries leading to data modification are not allowed
     at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:910)
     at com.mysql.jdbc.PreparedStatement.execute(PreparedStatement.java:792)
```

在增删改方法中，采用默认`readOnly=false`即可，也就是不需要写这个属性。  
在查询方法中，对于是否配置`readOnly=true`，目前我是配置了的，不过还有待研究。。。

### 传播行为（propagation）

事务的传播行为，有以下几种取值。

| 取值 | 含义 |
| --- | --- |
| Propagation.REQUIRED（默认） | 如果有事务则加入事务，没有的话就新建一个事务 |
| Propagation.REQUIRES\_NEW | 不管是否存在事务，都创建一个新的事务，原来的挂起，新的执行完毕，继续执行老的事务 |
| Propagation.MANDATORY | 必须在一个已有的事务中执行，否则抛出异常 |
| Propagation.NEVER | 必须在一个没有的事务中执行，否则抛出异常\(与Propagation.MANDATORY相反\) |
| Propagation.SUPPORTS | 如果其他bean调用这个方法，在其他bean中声明事务，那就用事务；如果其他bean没有声明事务，那就不用事务 |
| Propagation.NOT\_SUPPORTED | 不为这个方法开启事务 |

**Propagation.REQUIRED**

```java
ServiceA {   
     @Transactional（propagation = Propagation.REQUIRED）
     void methodA() {   
         ServiceB.methodB();   
     }   
}   

ServiceB {   
     @Transactional（propagation = Propagation.REQUIRED）
     void methodB() {  

     }   
}  

public class Controller(){
     public void methodC(){
          ServiceA.methodA();   
     }
}
```

这种情况下：methodA会开启新的事务，methodB会加入这个事务之中。无论在何时何处发生异常，两者都会回滚。

业务应用中绝大部分都是这种场景，所以Spring将其设为了默认值。

**PROPAGATION\_REQUIRES\_NEW**

```java
ServiceA {   
     @Transactional（propagation = Propagation.REQUIRED）
     void methodA() {   
         ServiceB.methodB();   
     }   
}   

ServiceB {   
     @Transactional（propagation = Propagation.PROPAGATION_REQUIRES_NEW）
     void methodB() {  

     }   
}  

public class Controller(){
     public void methodC(){
          ServiceA.methodA();   
     }
}
```

这种情况下：methodA会开启新的事务，当执行到methodB的时候，methodA所在的事务就会被挂起，methodB会开启一个新的事务，等待methodB的事务提交以后，methodA才继续执行。

回滚情况：

* methodB已经提交，methodA发生异常，则回滚methodA，而methodB不回滚
* methodB发生异常（未提交），则methodB都回滚
* * 如果methodA内部catch了methodB的异常，methodA没有发生异常，methodA可以成功提交，不会回滚
  * 如果methodA内部没有catch住methodB的异常，methodA会回滚

**PROPAGATION\_SUPPORTS**

如果当前环境有事务，就加入到当前事务；如果没有事务，就以非事务的方式执行。听起来跟普通方法没什么两样，它与普通方法的区别如下：

* 加了PROPAGATION\_SUPPORTS的方法可以获取和当前事务环境一致的Connection或Session，而普通方法获取到的是最新的；

* 加了PROPAGATION\_SUPPORTS的方法可以在挂起事务、恢复事务的时侯执行回调方法，而普通方法做不到。

Spring的文档是这么说的

> **NOTE:**For transaction managers with transaction synchronization,`PROPAGATION_SUPPORTS`is slightly different from no transaction at all, as it defines a transaction scope that synchronization might apply to. As a consequence, the same resources \(a JDBC`Connection`, a Hibernate`Session`, etc\) will be shared for the entire specified scope. Note that the exact behavior depends on the actual synchronization configuration of the transaction manager!
>
> In general, use`PROPAGATION_SUPPORTS`with care! In particular, do not rely on`PROPAGATION_REQUIRED`or`PROPAGATION_REQUIRES_NEW`withina`PROPAGATION_SUPPORTS`scope \(which may lead to synchronization conflicts at runtime\). If such nesting is unavoidable, make sure to configure your transaction manager appropriately \(typically switching to "synchronization on actual transaction"\).

###  事务隔离级别（isolation）

事务隔离级别，有以下几种取值：

| 取值 | 含义 |
| --- | --- |
| Isolation.DEFAULT（默认） | 使用底层数据源的配置（下面4种之一） |
| Isolation.READ\_UNCOMMITTED | 读取未提交数据\(会出现脏读, 不可重复读\) 基本不使用 |
| Isolation.READ\_COMMITTED | 读取已提交数据\(会出现不可重复读和幻读\) |
| Isolation.REPEATABLE\_READ | 可重复读\(会出现幻读\) |
| Isolation.SERIALIZABLE | 串行化 |

一般不需要设置显式这个属性，采用默认值即可。

### 回滚（rollbackFor和noRollbackFor）

默认是，当抛出一个unchecked异常（也就是运行时异常RuntimeException或其子类例的实例）时，会进行事务回滚。从事务方法中抛出的Checked exceptions将不被标识进行事务回滚。

概念补充：什么是unchecked和checked异常

> unchecked：运行时异常RuntimeException或其子类例的实例都是unchecked异常，可以理解为那些不用写try-catch的异常。  
> checked：除了上面说的，都是checked异常

如何改变默认规则：  
a.让checked异常也回滚：设置@Transactional\(rollbackFor=Exception.class\)  
b.让unchecked异常不回滚： 设置@Transactional\(notRollbackFor=RunTimeException.class\)  




## 

## 

## 参考

---

[Spring 事务 readOnly 到底是怎么回事？](http://www.cnblogs.com/hackem/p/3890656.html)  
[MySQL学习笔记（二）：事务管理](http://www.jianshu.com/p/0d0981f4be62)

[使用@Transactional\(propagation = Propagation.SUPPORTS\)和不加@Transactional 有什么区别？](https://miaoxinguo.github.io/spring/2016/05/03/spring.tx.supposts.html)

