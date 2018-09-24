# 使用EXPLAIN

使用EXPLAIN可以获取查询执行计划的信息，可以帮助开发者了解一条SQL查询语句具体是怎样执行的，比如使用了什么索引，关联表的顺序是怎样的，扫描了多少行记录等等，从而写出更优化的查询语句。

## EXPLAIN的输出

---

**在任意的SELECT查询语句的前面加上**`EXPLAIN`**这个词**，就可以分析MySQL在执行该语句时的具体信息，它的输出大致如下：

```sql
mysql> explain select 1\G
*************************** 1. row ***************************
id           : 1
select_type  : SIMPLE
table        : NULL
type         : NULL
possible_keys: NULL
key          : NULL
key_len      : NULL
ref          : NULL
rows         : NULL
Extra        : No tables used
1 rows in set (0.05 sec)
```

EXPLAIN各参数含义：

**id**

表示在本次查询语句中该SELECT的执行序号

* * id相同，执行顺序由上到下
  * id不同，越大则优先级越高，越先被执行
  * 如果是子查询，id的序号会递增；

**select\_type**

表示在本次查询语句中该SELECT是简单类型还是复杂类型。

如果是简单类型，则为`SIMPLE`：没有子查询，也没有`UNION`，则为简单类型。

如果是复杂类型，则最外层标记为`PRIMARY`，其他部分标记如下：

* * `SUBQUERY`：表示不在`FROM`子句中的子查询
* * `DERIVED`：表示在`FROM`子句中的子查询
* * `UNION`：表示在`UNION`中的第二个以及随后的SELECT
* * `UNION RESULT`：表示在`UNION`的临时表检索结果的SELECT
  * 除次之外，`SUBQUERY`和`UNION`还可以标记为`DEPENDENT`（意味着SELECT依赖与外层查询中发现的数据）和`UNCACHEABLE`。

**table**

表示该SELECT对应的表，即对应行正在访问哪个表。

**type**

访问类型，即MySQL如何查找表中的行。从最差到最优的顺序为：

* `ALL`：全表扫描，按从第一行到最后一行的顺序去查找需要的行。
* `index`：与全表扫描一样，只是MySQL扫描表时按照索引次序进行而不是行。
* `range`：范围扫描，一个有限制的索引扫描，比全索引扫描要好，因为不用遍历全部索引，主要是带有`BETWEEN`和`WHERE`子句里含有`>`的查询。
* * 当使用IN\(\)和OR，且字段有索引时，也会显示为range，但与上述的range扫描在性能上有差异
* `ref`：索引访问，也叫索引查找，它返回所有匹配某个单个值的行，但有可能会返回多个符合条件的行，因此，它是查找和扫描的结合体，只有使用非唯一索引或者唯一索引的非唯一性前缀时才会发生。
  * `ref_or_null`是`ref`的变种，它表示MySQL必须在初次查找的结果里进行第二次查找以找出`NULL`条目。
* `eq_ref`：使用这种索引查找，意味着MySQL知道最多只返回一条符合条件的记录，比如使用主键或者唯一索引查找时。
* `const, system`：使用主键索引或唯一索引的等值查询时，最多只返回一行数据，就是该const类型。
* * `system`：表中只有一行数据时
* `NULL`：这种访问类型意味着MySQL能在优化阶段分解查询语句，在执行阶段甚至用不着再访问表或者索引。例如，从一个索引列里选取最小值就可以通过单独查找索引来完成，不需要再去访问表。

SQL 性能优化的目标：至少要达到 range 级别，要求是 ref 级别，如果可以是 consts 最好。

**possible\_keys**

表示查询可以使用哪些索引，这是基于访问的列和使用的比较操作符来判断的，在后续优化中可能并不会实际用到这里列出的索引。

**key**

表示MySQL实际上采用了哪个索引。如果该索引没有出现在`possible_keys`中，那么MySQL可能选择了一个覆盖索引。

**key\_len**

表示MySQL在索引里使用的字节的长度。

**ref**

表示之前的表在key列记录的索引中查找值所用的列或常量。

**rows**

表示MySQL为了找到所需要的行而读取的行数。这个rows可能并不准确，且它不是结果集的行数，只是MySQL认为的必须读取的行的平均数，但实际上也可能不会真的读它估的所有行。

**Extra**

这一列包含的是不适合在其他列显示的额外信息（但是很重要），主要如下：

* `Using index`：表示MySQL将使用覆盖索引，以避免访问表。不要把覆盖索引和type列中的`index`访问类型弄混了。
* `Using where`：表示MySQL将在存储引擎检索行后再进行过滤，
* * 不是所有带有WHERE子句的查询都会显示`Using where`：假如存储引擎通过WHERE条件中涉及索引的列完成了所有的过滤，则不会显示`Using where`。
* `Using temporary`：表示MySQL在对查询结果排序时会使用一个临时表。

* `Using filesort`：表示MySQL会对结果使用文件排序（而非索引排序）。

## EXPLAIN分析示例

---



















参考：《高性能MySQL》

