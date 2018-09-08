### **HashMap中单链表与红黑树转换的规则**

在JDK8之前，桶结构是链表，JDK8中，桶结构会根据桶中元素个数在链表与红黑树两种数据结构中转换。桶中数量超过8个，就由链表转换为红黑树，桶中数量减少到6，就转换为链表。为什么是8和6呢？红黑树的平均查找时间复杂度为O\(log\(n\)\)，链表平均查找时间复杂度O\(n/2\)

| 桶中元素个数 | 链表平均查找时间复杂度O\(n/2\) | 红黑树平均查找时间复杂度O\(log\(n\)\) |
| :--- | :--- | :--- |
| 1 | 0.5 | 0 |
| 2 | 1 | 1 |
| 3 | 1.5 | 1.58 |
| 4 | 2 | 2 |
| 5 | 2.5 | 2.32 |
| 6 | 3 | 2.58 |
| 7 | 3.5 | 2.81 |
| 8 | 4 | 3 |
| 9 | 4.5 | 3.17 |
| 10 | 5 | 3.32 |
| 12 | 6 | 3.58 |
| 16 | 8 | 4 |
| 32 | 16 | 5 |

在元素个数≤6的时候，两种数据结构的平均查找复杂度差别不大，而在≥8的时候，红黑树相比于链表的平均时间复杂度显示出明显的优势。链表结构较为简单，而红黑树的实现则很复杂，这种设计是在复杂度和效率上寻求一种平衡。

### HashMap的负载因子为什么是0.75

默认负载因子（0.75）在时间和空间成本上提供了很好的折衷：负载因子较大时，table数组扩容的可能性就会少，所以相对占用内存较少（空间上较少），但是每条entry链上的元素会相对较多，查询的时间也会增长（时间上较多）。反之当负载因子较少的时候，给table数组扩容的可能性就高，那么内存空间占用就多，但是entry链上的元素就会相对较少，查出的时间也会减少。

在理想情况下，使用随机哈希码，不同键值对table数组中各个元素中出现的频率遵循**泊松分布**。TODO：理解注释中的这段话。

```
Ideally, under random hashCodes, the frequency of nodes in bins follows 
a Poisson distribution (http://en.wikipedia.org/wiki/Poisson_distribution)
with a parameter of about 0.5 on average for the default resizing
threshold of 0.75, although with a large variance because of
resizing granularity. Ignoring variance, the expected
occurrences of list size k are (exp(-0.5) * pow(0.5, k) /
factorial(k)). The first values are:

0:    0.60653066
1:    0.30326533
2:    0.07581633
3:    0.01263606
4:    0.00157952
5:    0.00015795
6:    0.00001316
7:    0.00000094
8:    0.00000006
more: less than 1 in ten million
```



**HashMap在高并发下如果没有处理线程安全会有怎样的安全隐患，具体表现是什么？**

①put后可能导致get死循环，具体表现为CPU使用率100%：

②put的时候可能导致元素丢失：执行addEntry\(hash, key, value, i\)，如果有产生哈希碰撞，导致两个线程得到同样的bucketIndex去存储，就可能会出现覆盖丢失的情况：



内容参考

[HashMap在并发下可能出现的问题分析](http://www.cnblogs.com/binyue/p/3726403.html)

[面试中对hashMap的再次理解，负载因子为什么为0.75原](https://my.oschina.net/iioschina/blog/897208)

[HashMap的loadFactor为什么是0.75？](https://www.jianshu.com/p/64f6de3ffcc1)

  


  










