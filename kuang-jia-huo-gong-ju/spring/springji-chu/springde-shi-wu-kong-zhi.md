---
### 一、注解位置

@Transactional可以放在两个位置
* Service层的实现类上：该类中所有方法都进行事务处理
```
如
@Service
@Transactional
public class OrgServiceImpl implements OrgService {
          ...
}
```

* Service层的实现类的具体方法上：该方法进行事务处理
```
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

---
### 二、@Transactional的属性
先看一个表格总结：

属性名 | 功 能 描 述
--|--
readOnly | 	该属性用于设置当前事务是否为只读事务，设置为true表示只读，false则表示可读写，**默认值为false**。例如：@Transactional(readOnly=true)
propagation | 该属性用于设置事务的传播行为。**默认值为Propagation.REQUIRED** 例如：@Transactional(propagation=Propagation.NOT_SUPPORTED,readOnly=true)
isolation | 该属性用于设置底层数据库的事务隔离级别。**默认值为Isolation.DEFAULT**。事务隔离级别用于处理多事务并发的情况，通常使用数据库的默认隔离级别即可，基本不需要进行设置
timeout | 该属性用于设置事务的超时秒数，**默认值为-1，表示永不超时**
rollbackFor | 该属性用于设置需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，则进行事务回滚。例如： 指定单一异常类：@Transactional(rollbackFor=RuntimeException.class) 指定多个异常类：@Transactional(rollbackFor={RuntimeException.class, Exception.class})
rollbackForClassName | 该属性用于设置需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，则进行事务回滚。例如： 指定单一异常类名称：@Transactional(rollbackForClassName="RuntimeException") 指定多个异常类名称：@Transactional(rollbackForClassName={"RuntimeException","Exception"})
noRollbackFor | 该属性用于设置不需要进行回滚的异常类数组，当方法中抛出指定异常数组中的异常时，不进行事务回滚。例如： 指定单一异常类：@Transactional(noRollbackFor=RuntimeException.class) 指定多个异常类：@Transactional(noRollbackFor={RuntimeException.class, Exception.class})
noRollbackForClassName | 该属性用于设置不需要进行回滚的异常类名称数组，当方法中抛出指定异常名称数组中的异常时，不进行事务回滚。例如： 指定单一异常类名称：@Transactional(noRollbackForClassName="RuntimeException") 指定多个异常类名称： @Transactional(noRollbackForClassName={"RuntimeException","Exception"})

再来看几个重要属性的具体释义

##### 2.1 readOnly

是否是只读事务。

取值|含义
--|--
false（默认）| 说明为读写事务
true|说明为只读事务，对于JDBC而言，只读事务会有一定的速度优化。此时，事务控制的其他配置会采用默认值，事务的隔离级别(isolation) 为DEFAULT（采用底层数据源的隔离级别），事务的传播行为(propagation)则是REQUIRED，所以还是会有事务存在。

补充：在将事务设置成只读后，相当于将数据库设置成只读数据库，此时若要进行写的操作，会出现错误。
建议阅读：[Spring 事务 readOnly 到底是怎么回事？](http://www.cnblogs.com/hackem/p/3890656.html)，给出的结论是：
>1 readonly并不是所有数据库都支持的，不同的数据库下会有不同的结果。
2 设置了readonly后，connection都会被赋予readonly，效果取决于数据库的实现。
3 在ORM中，设置了readonly会赋予一些额外的优化，例如在Hibernate中，会被禁止flush等。

如果你使用的是mysql数据库，出现了这样的报错信息：
```
Caused by: java.sql.SQLException: Connection is read-only. Queries leading to data modification are not allowed
2     at com.mysql.jdbc.SQLError.createSQLException(SQLError.java:910)
3     at com.mysql.jdbc.PreparedStatement.execute(PreparedStatement.java:792)
```
那么很有可能是你在增删改等修改操作的方法上不小心加上了`readOnly=true`。

在增删改方法中，采用默认`readOnly=false`即可，也就是不需要写这个属性。
在查询方法中，对于是否配置`readOnly=true`，目前我是配置了的，不过还有待研究。。。

##### 2.2 propagation
事务的传播行为，有以下几种取值。

取值|含义
--|--
Propagation.REQUIRED（默认）|  如果有事务, 那么加入事务, 没有的话新建一个
Propagation.NOT_SUPPORTED  |  容器不为这个方法开启事务
Propagation.REQUIRES_NEW  |  不管是否存在事务,都创建一个新的事务,原来的挂起,新的执行完毕,继续执行老的事务
Propagation.MANDATORY  |  必须在一个已有的事务中执行,否则抛出异常
Propagation.NEVER  |  必须在一个没有的事务中执行,否则抛出异常(与Propagation.MANDATORY相反)
Propagation.SUPPORTS  |  如果其他bean调用这个方法，在其他bean中声明事务，那就用事务；如果其他bean没有声明事务，那就不用事务.

一般不需要设置显式这个属性，采用默认值即可；查询方法可以单独配置Propagation.NOT_SUPPORTED

##### 2.3 isolation
事务隔离级别，有以下几种取值：

取值|含义
--|--
Isolation.DEFAULT（默认）|使用底层数据源的配置（下面4种之一）
Isolation.READ_UNCOMMITTED |  读取未提交数据(会出现脏读, 不可重复读) 基本不使用
Isolation.READ_COMMITTED |  读取已提交数据(会出现不可重复读和幻读)
Isolation.REPEATABLE_READ |  可重复读(会出现幻读)
Isolation.SERIALIZABLE |  串行化
具体释义可以参考[MySQL学习笔记（二）：事务管理](http://www.jianshu.com/p/0d0981f4be62)
一般不需要设置显式这个属性，采用默认值即可。

##### 2.4 rollbackFor和noRollbackFor
默认是，当抛出一个unchecked异常（也就是运行时异常RuntimeException或其子类例的实例）时，会进行事务回滚。从事务方法中抛出的Checked exceptions将不被标识进行事务回滚。

概念补充：什么是unchecked和checked异常
>unchecked：运行时异常RuntimeException或其子类例的实例都是unchecked异常，可以理解为那些不用写try-catch的异常。
checked：除了上面说的，都是checked异常

如何改变默认规则：
a.让checked例外也回滚：在整个方法前加上 @Transactional(rollbackFor=Exception.class)
b.让unchecked例外不回滚： @Transactional(notRollbackFor=RunTimeException.class)
c.不需要事务管理的(只查询的)方法：@Transactional(propagation=Propagation.NOT_SUPPORTED)