# 常见查询场景

## 增加排名字段

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

```text
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

## 分组后取组内TOP 1或者Top N

假如，有成绩表，现在，需要按照课程进行分组，然后取出每门课程成绩最高或Top N 的学生和分数：

```sql
# 建表
CREATE TABLE `score` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) NOT NULL COMMENT '用户id',
  `course_id` int(11) NOT NULL COMMENT '课程id',
  `score` int(11) NOT NULL COMMENT '分数',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=24 DEFAULT CHARSET=utf8

# 测试数据
INSERT INTO score (user_id, course_id, score) VALUES(1,1,60);
INSERT INTO score (user_id, course_id, score) VALUES(1,2,80);
INSERT INTO score (user_id, course_id, score) VALUES(1,3,90);
INSERT INTO score (user_id, course_id, score) VALUES(1,4,88);
INSERT INTO score (user_id, course_id, score) VALUES(1,5,61);

INSERT INTO score (user_id, course_id, score) VALUES(2,1,45);
INSERT INTO score (user_id, course_id, score) VALUES(2,2,78);
INSERT INTO score (user_id, course_id, score) VALUES(2,3,35);
INSERT INTO score (user_id, course_id, score) VALUES(2,4,89);
INSERT INTO score (user_id, course_id, score) VALUES(2,5,71);

INSERT INTO score (user_id, course_id, score) VALUES(3,1,36);
INSERT INTO score (user_id, course_id, score) VALUES(3,2,98);
INSERT INTO score (user_id, course_id, score) VALUES(3,3,100);
INSERT INTO score (user_id, course_id, score) VALUES(3,4,35);
INSERT INTO score (user_id, course_id, score) VALUES(3,5,70);

INSERT INTO score (user_id, course_id, score) VALUES(4,1,36);
INSERT INTO score (user_id, course_id, score) VALUES(4,2,98);
INSERT INTO score (user_id, course_id, score) VALUES(4,3,100);
INSERT INTO score (user_id, course_id, score) VALUES(4,4,35);
INSERT INTO score (user_id, course_id, score) VALUES(4,5,70);

```

### TOP 1

### 方案1：自连接

查询语句如下：

```sql
SELECT
	s.course_id ,
	s.score ,
	s.user_id
FROM
	score s
JOIN(
	SELECT
		course_id ,
		max(score) AS score
	FROM
		score
	GROUP BY
		course_id
) s1 ON s.course_id = s1.course_id AND s.score = s1.score
ORDER BY s.course_id

```

查询结果：

```text
+-----------+-------+---------+
| course_id | score | user_id |
+-----------+-------+---------+
| 1         | 60    | 1       |
| 2         | 98    | 3       |
| 2         | 98    | 4       |
| 3         | 100   | 3       |
| 3         | 100   | 4       |
| 4         | 89    | 2       |
| 5         | 71    | 2       |
+-----------+-------+---------+
7 rows in set (0.01 sec)
# 因为课程2、3的最高成绩对应的用户有多条，所以会有多条记录
```

#### 方案2：子查询

```sql
SELECT
	course_id ,
	score ,
	user_id
FROM
	score s
WHERE
	score =(
		SELECT
			MAX(score)
		FROM
			score s1
		WHERE
			s.course_id = s1.course_id
	)
ORDER BY
	course_id
```

查询结果：

```text
+-----------+-------+---------+
| course_id | score | user_id |
+-----------+-------+---------+
| 1         | 60    | 1       |
| 2         | 98    | 3       |
| 2         | 98    | 4       |
| 3         | 100   | 3       |
| 3         | 100   | 4       |
| 4         | 89    | 2       |
| 5         | 71    | 2       |
+-----------+-------+---------+
7 rows in set (0.01 sec)
```

### TOP N

假设取TOP 2，即取每门课程成绩较高的2个分数和对应的用户。

#### 方案1：使用UNION ALL

如果结果集比较小，可以用程序查询单个分组结果后拼凑，也可以使用UNION ALL拼凑。

查询语句：如果课程共有5中，则写5条单独查，再使用UNION ALL聚合。

```sql
( SELECT course_id , score , user_id FROM score WHERE course_id = 1 ORDER BY score DESC LIMIT 2) 
UNION ALL
( SELECT course_id , score , user_id FROM score WHERE course_id = 2 ORDER BY score DESC LIMIT 2) 
UNION ALL
( SELECT course_id , score , user_id FROM score WHERE course_id = 3 ORDER BY score DESC LIMIT 2)
 UNION ALL
( SELECT course_id , score , user_id FROM score WHERE course_id = 4 ORDER BY score DESC LIMIT 2) 
UNION ALL
( SELECT course_id , score , user_id FROM score WHERE course_id = 5 ORDER BY score DESC LIMIT 2)
```

查询结果：（共10条记录）

```text
+-----------+-------+---------+
| course_id | score | user_id |
+-----------+-------+---------+
| 1         | 60    | 1       |
| 1         | 45    | 2       |
| 2         | 98    | 4       |
| 2         | 98    | 3       |
| 3         | 100   | 4       |
| 3         | 100   | 3       |
| 4         | 89    | 2       |
| 4         | 88    | 1       |
| 5         | 71    | 2       |
| 5         | 70    | 4       |
+-----------+-------+---------+
10 rows in set (0.01 sec)
```

这种方案有一个问题：同一课程，前2高的分数，可能会对应2个以上用户，即可能会漏数据。

#### 方案2：自身左连接

查询语句：

```sql
SELECT
	s.course_id ,
	s.score ,
	s.user_id
FROM
	score s
LEFT JOIN score s1 ON s.course_id = s1.course_id AND s.score < s1.score
GROUP BY
	s.user_id ,
	s.course_id ,
	s.score
HAVING
	COUNT(s1.id) < 2
ORDER BY
	s.course_id ,
	s.score DESC
```

查询结果：

```text
+-----------+-------+---------+
| course_id | score | user_id |
+-----------+-------+---------+
| 1         | 60    | 1       |
| 1         | 45    | 2       |
| 2         | 98    | 3       |
| 2         | 98    | 4       |
| 3         | 100   | 3       |
| 3         | 100   | 4       |
| 4         | 89    | 2       |
| 4         | 88    | 1       |
| 5         | 71    | 2       |
| 5         | 70    | 4       |
| 5         | 70    | 3       |
+-----------+-------+---------+
11 rows in set (0.01 sec)
```

#### 方案3：子查询

查询语句：

```sql
SELECT
	course_id ,
	score ,
	user_id
FROM
	score s
WHERE 
	(
		SELECT COUNT(*) FROM score s1 
		WHERE s1.course_id = s.course_id AND s1.score > s.score
	) < 2
ORDER BY course_id ASC , score DESC

# 或者如下子查询
SELECT
	course_id ,
	score ,
	user_id
FROM
	score s
WHERE
	(
		SELECT COUNT(*) FROM score s1
		LEFT JOIN score s2 ON s1.course_id = s2.course_id AND s1.score < s2.score
		WHERE s1.id = s.id
	) < 2
ORDER BY s.course_id ASC, score DESC
```

查询结果：

```text
+-----------+-------+---------+
| course_id | score | user_id |
+-----------+-------+---------+
| 1         | 60    | 1       |
| 1         | 45    | 2       |
| 2         | 98    | 3       |
| 2         | 98    | 4       |
| 3         | 100   | 3       |
| 3         | 100   | 4       |
| 4         | 89    | 2       |
| 4         | 88    | 1       |
| 5         | 71    | 2       |
| 5         | 70    | 3       |
| 5         | 70    | 4       |
+-----------+-------+---------+
11 rows in set (0.01 sec)
```

#### 方案4 ：使用用户变量

查询语句：

```sql
set @num := 0, @course_id := 0;
SELECT
	course_id ,
	score ,
	user_id
FROM
	(
		SELECT
			course_id ,
			score ,
			user_id ,
			@num := IF(@course_id = course_id , @num + 1 , 1) AS row_number ,
			@course_id := course_id AS dummy
		FROM
			score
		ORDER BY
			course_id, score DESC
	) AS s
WHERE s.row_number <= 2
```

查询结果：

```text
+-----------+-------+---------+
| course_id | score | user_id |
+-----------+-------+---------+
| 1         | 60    | 1       |
| 1         | 45    | 2       |
| 2         | 98    | 3       |
| 2         | 98    | 4       |
| 3         | 100   | 3       |
| 3         | 100   | 4       |
| 4         | 89    | 2       |
| 4         | 88    | 1       |
| 5         | 71    | 2       |
| 5         | 70    | 3       |
+-----------+-------+---------+
10 rows in set (0.01 sec)
```

这种方案同样有一个问题：同一课程，前2高的分数，可能会对应2个以上用户，即可能会漏数据。

## 参考

[Rank function in MySQL](https://stackoverflow.com/questions/3333665/rank-function-in-mysql)

[MySQL获取分组后的TOP 1和TOP N记录](http://sangei.iteye.com/blog/2359584)

