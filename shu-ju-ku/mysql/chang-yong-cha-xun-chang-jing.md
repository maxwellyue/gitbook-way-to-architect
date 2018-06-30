# 常用查询场景

#### 1、按某字段排序后，增加排名字段

比如，用户表中，按照年龄排序后，增加rank字段：

```sql
# 建表
CREATE TABLE user (id int, name varchar(20), age int, gender char(1));
# 测试数据
INSERT INTO user VALUES (1, 'Bob', 25, 'M');
INSERT INTO user VALUES (2, 'Jane', 20, 'F');
INSERT INTO user VALUES (3, 'Jack', 30, 'M');
INSERT INTO user VALUES (4, 'Bill', 32, 'M');
INSERT INTO user VALUES (5, 'Nick', 22, 'M');
INSERT INTO user VALUES (6, 'Kathy', 18, 'F');
INSERT INTO user VALUES (7, 'Steve', 36, 'M');
INSERT INTO user VALUES (8, 'Anne', 25, 'F');
```

查询语句：

```sql
SELECT    name,
          age,
          gender,
          @curRank := @curRank + 1 AS rank
FROM      user, (SELECT @curRank := 0) r
ORDER BY  age;
```

查询结果：

```sql
+-------+------+--------+------+
| name  | age  | gender | rank |
+-------+------+--------+------+
| Kathy | 18   | F      | 1    |
| Jane  | 20   | F      | 2    |
| Nick  | 22   | M      | 3    |
| Bob   | 25   | M      | 4    |
| Anne  | 25   | F      | 5    |
| Jack  | 30   | M      | 6    |
| Bill  | 32   | M      | 7    |
| Steve | 36   | M      | 8    |
+-------+------+--------+------+
8 rows in set (0.02 sec)
```



