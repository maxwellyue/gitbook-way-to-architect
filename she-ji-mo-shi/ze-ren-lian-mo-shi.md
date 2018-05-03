# 责任链模式

---

顾名思义，责任链 = 责任 + 链。

对谁负责，可以称为要处理的对象（target），即这些责任是指对这个对象有责任。

有什么责任，即要怎么处理这个target，理解为处理器Handler

链，即把这些Handler串起来，链的英文为chain

将target交给chain来处理，chain上有n个Handler，

具体由哪个或哪几个Handler来处理，由Hanlder自身决定；

Hanlder怎么处理该trarget，自然也由Hanlder自身决定。



![](/assets/责任链流程示意图.png)



























# 参考

---

[责任链模式](http://www.runoob.com/design-pattern/chain-of-responsibility-pattern.html)



