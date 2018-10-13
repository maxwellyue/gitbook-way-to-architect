# @Transactional注解在什么情况下会失效

## @Transactional注解在什么情况下会失效？

以下情况下，@Transactional注解会失效：

* @Transactional 注解只能应用到 public 可见度的方法上。 如果应用在protected、private或者 package可见度的方法上，也不会报错，不过事务设置不会起作用。
* 在 Spring 的 AOP 代理下，只有目标方法由外部调用，目标方法才由 Spring 生成的代理对象来管理，这会造成自调用问题。若同一类中的其他没有@Transactional 注解的方法内部调用有@Transactional 注解的方法，有@Transactional 注解的方法的事务被忽略，不会发生回滚。

```text
@Service
public class OrderService {
    private void insert() {
        insertOrder();
    }
    @Transactional
    public void insertOrder() {
        //insert log info
        //insertOrder
        //updateAccount
    }
}
```

insertOrder 尽管有@Transactional 注解，但它被内部方法 insert 调用，事务被忽略，出现异常事务不会发生回滚。

上面的两个问题@Transactional 注解只应用到 public 方法和自调用问题，是由于使用 Spring AOP 代理造成的。为解决这两个问题，使用 AspectJ 取代 Spring AOP 代理。

```text
<tx:annotation-driven mode="aspectj" />
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
    <property name="dataSource" ref="dataSource" />
</bean>
</bean class="org.springframework.transaction.aspectj.AnnotationTransactionAspect" factory-method="aspectOf">
    <property name="transactionManager" ref="transactionManager" />
</bean>
```

## 解决Transactional注解不回滚

* 检查你方法是不是public的
* 你的异常类型是不是unchecked异常 （默认对只对unchecked异常回滚）
* 数据库引擎要支持事务，如果是MySQL，注意表要使用支持事务的引擎，比如innodb，如果是myisam，事务是不起作用的
* 是否开启了对注解的解析
* spring是否扫描到你这个包
* 检查是不是同一个类中的方法调用（如a方法调用同一个类中的b方法）

内容来源：

[透彻的掌握 Spring 中@transactional 的使用](https://www.ibm.com/developerworks/cn/java/j-master-spring-transactional-use/index.html) [@Transactional注解事务不回滚不起作用无效](http://www.cnblogs.com/powerwu/articles/8392606.html)

