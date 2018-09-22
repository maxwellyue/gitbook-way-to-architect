MySQL中的数据类型大致分为3种：数字类型、日期类型、字符串类型。

## 数字类型

---

MySQL中数字类型可分为两类：

* 整型：TINYINT\(1字节\)，SMALLINT\(2字节\)，MEDIUNINT\(3字节\)，INT\(4字节\)，BIGINT\(8字节\)；

* 非整形：FLOAT\(4字节\)，DOUBLE\(8字节\)，DECIMAL\(M,D\)—— m:总个数，d:小数位；

**使用建议**

（1）如果确认为无符号，则务必指定为无符号（ungsighed）；

（2）使用尽可能小的类型

* * 对于常见的有限集，比如某对象的status或者type，如果用数字类型来表示，使用tinyint就够了
  * 对于对象的id，一般选择为BIGINT

（3）创建数字类型的字段时，不必向创建字符串类型的字段一样指定其长度，为数字类型的字段指定的长度仅仅是为了可视化时的显示长度（display width），对于存储等没有任何效果。如下所示，tinyint\(1\)中的1并非指定status的长度为1，tinyint类型的字段的长度是固定的1字节。

```sql
//应尽量避免这种标识，以免产生歧义
CREATE TABLE `domain_name` (
    ... ...
    `status` tinyint(1) NOT NULL DEFAULT '1' COMMENT '状态(1/启用，0/停用，-1/删除)'
)
```

（4）小数类型选择decimal，禁止使用float和double。

* * float和double在存储的时候，存在精度损失的问题，很可能在值的比较时，得到不正确的结果。
  * 如果存储的数据范围超过decimal的范围，建议将数据拆成整数和小数分开存储













## 日期类型

---

MySQL中日期类型有以下几种：

| 类型 | 大小 | 范围 | 格式 | 用途 |
| :--- | :--- | :--- | :--- | :--- |
| DATE | 3字节 | 1000-01-01/9999-12-31 | YYYY-MM-DD | 日期值 |
| TIME | 3字节 | '-838:59:59'/'838:59:59' | HH:MM:SS | 时间值或持续时间 |
| YEAR | 1字节 | 1901/2155 | YYYY | 年份值 |
| DATETIME | 8字节 | 1000-01-01 00:00:00/9999-12-31 23:59:59 | YYYY-MM-DD HH:MM:SS | 混合日期和时间值 |
| TIMESTAMP | 4字节 | 1970-01-01 00:00:00/2038；结束时间是第**2147483647**秒，北京时间**2038-1-19 11:14:07**，格林尼治时间 2038年1月19日 凌晨 03:14:07 | YYYYMMDD HHMMSS | 混合日期和时间值，时间戳 |

**使用建议**

（1）尽量选择恰当的类型：如果只是记录年，则使用YEAR，如果只是记录时间，则使用TIME，不要一概而论地都使用DATATIME

（2）虽然TIMESTAMP具有在创建或更新时自动设置为当前时间的功能，但是这种控制还是交给程序，让MySQL做它应该做的事情，况且，TIMESTAMP只能表示到2038年1月19日，之后呢？所以，建议使用DATATIME这种类型。

（3）关于时区转换

* * 在MySQL只有TIMESTAMP会做时区转换：无论是在读取还是写入`TIMESTAMP`类型的值，都会根据connection的时区（对于Java程序而言，就是Java程序所在的JVM的时区）做转换。也就是说，如果JVM与MySQL不在一个时区，那么JVM去设置一个TIMESTAMP类型的字段值时，MySQL会根据连接获取到JVM所在的时区，然后转换为本地时区的时间值再写入；同样，在JVM去读取TIMESTAMP类型的字段值时，读到的是JVM所在的时区的时间。
  * 不要在服务器端做日期时间的字符串格式化（DATE\_FORMAT\(\)），因为返回的结果是服务端的时区，而不是connection的时区（对于JDBC来说就是JVM时区）。
  * CURRENT\_TIMESTAMP\(\), CURRENT\_TIME\(\), CURRENT\_DATE\(\)可以安全的使用，返回的结果会转换成connection时区（对于JDBC来说就是JVM时区）。

## 字符串类型

---

MySQL中字符串类型有以下几种：CHAR，VARCHAR，TEXT，ENUM和SET，此外还有BLOB。

| 类型 | 最大长度 | 说明 |
| :--- | :--- | :--- |
| CHAR | 255字节 | CHAR是定长的，即在创建某字段时，如果指定长度为CHAR\(4\)，则无论实际存储的字符是否有4这么长，MySQL都会分配4的空间 |
| VARCHAR | 65535字节 | VARCHAR是变长的，即使指定长度为VARCHAR\(16\)，如果实际存储的值只有8的长度，则MySQL实际存储时，也只会分配8的空间 |
| TEXT | TINYTEXT  /  TEXT  /  MEDIUMTEXT  /  LONGTEXT | 创建索引时，必须指定长度；用于存储大文本，较少使用 |
| BLOB | - | 以二进制形式（字节）存取，不必指定字符集，比如存储视频/图片等数据，但是一般不会使用 |
| ENUM/SET | - | 枚举类型 |

**使用建议**

（1）在MySQL中，1个英文字母或1个数字占用1个字节，而1个汉字则根据不同的编码方式，会占用3或4个字节；

（2）在创建字符类型字段时，通过形如CAHR\(n\)或VARCHAR\(n\)指定的长度n，其含义为：该字段所能存储的字符的最大个数，而非最大字节数；这里的1各字符，可以是1个汉字，也可以1个字母，也可以是1个数字。如一个字段声明为VARCHAR\(16\)，则该字段最多可以存储16个汉字或者16个字母或16个数字。

（3）如果存储的字符串长度几乎相等，使用char定长字符串类型。

（4）varchar是可变长字符串，不预先分配存储空间，长度不要超过5000，如果存储长度大于此值，定义字段类型为text，独立出来一张表，用主键来对应，避免影响其它字段索引效率。

（3）慎用ENUM

* * 要么是数字，要么是字符串，要么是时间，ENUM里面是什么？ENUM更像业务数据的特征信息，不是数据类型
  * 更改ENUM的代价大，不如使用TINYINT或者CHAR灵活，在海量数据下，假如后来发现ENUM需要增加一种枚举值，则需要ALTER TABLE，而使用TINYINT或者CHAR，则无需如此。
  * ENUM类型不是SQL标准，属于MySQL，而其他DBMS不一定有原生的支持。



## 参考

---

[数据库时区那些事儿 - MySQL的时区处理](https://segmentfault.com/a/1190000016426048)

[varchar\(n\)中的n和int\(n\)中的n到底是什么意思](https://www.jianshu.com/p/888da110be2c)

[MySQL 枚举类型的“八宗罪”](https://www.zcfy.cc/article/a-href-title-komlenic-com-komlenic-com-a)

[8 Reasons Why MySQL's ENUM Data Type Is Evil](http://komlenic.com/244/8-reasons-why-mysqls-enum-data-type-is-evil/)

《阿里巴巴Java开发手册v1.2》



