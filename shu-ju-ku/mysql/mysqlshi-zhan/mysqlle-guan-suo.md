# MySQL乐观锁

---

在[MySQL中锁概述](/shu-ju-ku/mysql/mysqlsuo/mysqlzhong-suo-de-fen-lei.md)中，讲述了MySQL中的不同类型的锁，这些锁会将对应的数据进行锁定，以保持数据访问的正确性，即这些锁总是认为这些数据总是会被其他事务也使用到，这是一种悲观的策略，即总是认为不好的行为的发生（对应到数据库，就是总是认为数据被并发访问）。但是，如果这些数据根本就不会被并发修改，那么加锁就是一种为保证数据正确而做的无谓的保护措施。这其实就是悲观锁。

假如数据在大多数情况下，仅仅会被单事务所访问，那么，乐观锁就是一种更好的控制数据正确性的方式。

乐观锁是基于这样的假设：数据一般情况下不会造成冲突，所以在数据进行提交更新的时候，才会正式对数据的冲突与否进行检测，如果发现冲突了，则让返回用户错误的信息，让用户决定如何去做。

实现乐观锁的实现一般有以下2种方式。

**使用版本Version**

这是乐观锁最常用的一种实现方式：为数据增加一个版本标识，一般是通过为数据库表增加一个数字类型的 “version” 字段来实现。

读取数据时，将version字段的值一同读出；

当修改数据时，判断数据库表对应记录的当前版本信息与第一次取出来的version值进行比对，如果数据库表当前版本号与第一次取出来的version值相等，则予以更新，否则认为是过期数据。修改数据后，对此version值加一。

数据修改时的伪代码如下，假如要更新user\(uid, name, version\)的name字段：

```sql
update(){
    # res表示更新语句影响的行数
    int res = 0;

    # 无限循环更新
    for(;;){
        # 查询出当前的version， 假设结果为v1
        SELECT version FROM user where uid = 1;
        # 更新，设置额外条件version=v1，并同时更新version字段
        res = UPDATE user SET name=maxwell, version=v1+1 WHERE uid=1 AND version=v1;
        # 说明更新成功
        if(res == 1){
            break;
        }
    }
}
```

因为使用乐观锁的业务场景就是并发不会太多，所以这里采用了for\(;;\)的无限循环的方式来保证修改成功。当然，可以根据业务需要，只尝试固定的次数，如：

```sql
tryUpdate(int times){
    # res表示在最多times次尝试后，最终是否更新成功
    boolean res = false;
    for(x in times){
        # 查询出当前的version， 假设结果为v1
        SELECT version FROM user where uid = 1;
        # 更新，设置额外条件version=v1，并同时更新version字段
        int i = UPDATE user SET name=maxwell, version=v1+1 WHERE uid=1 AND version=v1;
        if(i == 1 ){
            res = true;
            break;
        }
    }
    return res;
}
```

**使用时间戳**

这种方式与第一种类似，在表中增加一个类型使用时间戳的字段time， 和上面的version类似，也是在更新的时候检查当前数据库中数据的时间戳和当前更新前取到的时间戳进行对比，如果一致则OK，否则就是版本冲突。

数据修改时的伪代码如下，假如要更新user\(uid, name, time\)的name字段：

```sql
update(){
    int res = 0;
    for(;;){
        SELECT time FROM user where uid = 1; # 假设结果为time1
        res = UPDATE user SET name=maxwell, time=CURRENT_TIME WHERE uid=1 AND time=time1;
        if(res == 1){
            break;
        }
    }
}

tryUpdate(int times){
    boolean res = false;
    for(x in times){
        SELECT time FROM user where uid = 1; # 假设结果为time1
        int i = UPDATE user SET name=maxwell, time=CURRENT_TIME WHERE uid=1 AND time=time1;
        if(i == 1 ){
            res = true;
            break;
        }
    }
    return res;
}
```

在实际应用中，一般表中都会含有update\_time字段来表示记录的最后一次更新的时间，直接使用该字段来进行版本控制实现乐观锁即可。

**实际场景分析**

以上这两种方式，我们都是为表增加业务无关的字段来表示数据的版本，在真实的场景中，有些业务字段本身就具有版本的意义，且往往要仅需要安全地更新该字段，此时，无需额外增加字段，直接使用该字段也可以实现乐观锁这种思想。

举一个例子：两张表：host\(id, disk\_size\)，disk\(id, host\_id, size\)，其中，host表中有disk\_size字段来表示机器的总磁盘大小，disk表中有size表示某个磁盘的大小，host和disk为一对多的关系。

假如我们不加乐观锁控制，直接更新，伪代码如下：

```sql
updateDisk(){
    UPDATE disk SET size = ? WHERE id = 1;                                           --- ①      
    updateHost();
}
updateHost(){
    int size = SELECT SUM(size) FROM disk WHERE host_id = 1;                            --- ②
    UPDATE host SET disk_size=size  WHERE id=1;
}

# 初始数据     
disk(1, 1, 1500)   
disk(2, 1, 500)
host(1, 2000)
```

updateDisk为并发操作，且加了事务控制。假设现在要事务A和事务B去执行updateDisk\(\)操作：

```sql
事务A：disk(1, 1, 1500) 》 disk(1, 1, 2000)
事务B：disk(2, 1, 500)  》 disk(2, 1, 1500)
```

假设A和B同时执行完了②，则A获取的size为500+2000=2500，B获取的size为1500+1500=3000；如果事务A先拿到host\(id=1\)的行锁，即按照A&gt;B的顺序执行update host，则最终host\(1, 2000\)会被更新为host\(1, 3000\)，如果事务B先拿到host\(id=1\)的行锁，即按照B&gt;A的顺序执行update host，则最终host\(1, 2000\)会被更新为host\(1, 2500\)。这两种情况，最终都不会将host\(1, 2000\)更新为我们想要的host\(1,3500\)。

现在，我们以host中的disk\_size作为实现乐观锁的数据版本字段，改写updateDisk，伪代码如下

```sql
updateDisk(){
    UPDATE disk SET size = ? WHERE id = 1;                                            --- ①
    updateHost();
}
updateHost(){
    # 首先查出host的当前磁盘大小
    int size = SELECT SUM(size) FROM disk WHERE host_id = 1;                          --- ②
    int initSize = SELECT disk_size FROM host WHERE id = 1;                           --- ③
    # 尝试更新
    for(;;){
        int count = UPDATE host SET disk_size=size WHERE id=1 AND disk_size=initSize; --- ④
        if(count == 1){
            break;
        }
    }
}

# 初始数据     
disk(1, 1, 1500)   
disk(2, 1, 500)
host(1, 2000)
```

updateDisk为并发操作，且加了事务控制。同样现在要事务A和事务B去执行updateDisk\(\)操作：

```sql
事务A：disk(1, 1, 1500) 》 disk(1, 1, 2000)
事务B：disk(2, 1, 500)  》 disk(2, 1, 1500)
```

假设事务A，B同时执行到④，则A获取的initSize为500+2000=2500，B获取的initSize为1500+1500=3000；假设A先拿到id=1的行锁，则执行更新，count=1，A从updateDisk返回，将disk\_size更新为了2500；此时，B由于A释放了id=1的行锁而拿到行锁，执行④，但是此时的disk\_size已经被A更新为2500，不再是3000，所以count=0，会继续尝试，下次尝试获取的initSize为2500，即可成功将disk\_size更新为了3500。

假如不适用乐观锁，而是使用悲观锁，上述伪代码可以改写为：

```sql
updateDisk(){
    UPDATE disk SET size = ? WHERE id = 1; 
    updateHost();
}
updateHost(){
    int size = SELECT SUM(size) FROM disk WHERE host_id = 1 FOR UPDATE; 
    UPDATE host SET disk_size=size  WHERE id=1;
}
```

这样，disk中\(host\_id=1\)的数据就会被加上排它锁，阻止其他事务的任何对这些记录的













