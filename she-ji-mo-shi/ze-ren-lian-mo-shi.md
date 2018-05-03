# 责任链模式

---

**顾名思义，责任链 = 责任 + 链。**

**对谁负责**：可以称为要处理的对象（target），即这些责任是指对这个target有责任。

**有什么责任**：即要怎么处理这个target，理解为处理器Handler

**链**：即把这些Handler串起来，链的英文为chain。将target交给chain来处理，chain上有n个Handler，具体由哪个或哪几个Handler来处理，由Hanlder自身决定；Hanlder怎么处理该trarget，自然也由Hanlder自身决定。

![](/assets/责任链流程示意图.png)

**典型应用场景**

①JS 中的事件冒泡

②Java Web Servlet 的Filter

③SpringMVC中的Inteceptor

④Netty中的ChannelHanler和ChannelPipeline



实践经验：

①有可能某一个Handler处理后，不再向后传递，直接结束链

②各个Hanlder可能有顺序要求，即流向必须是hanlder1&gt;handler2&gt;handler&gt;3



TODO：阅读Filter/Netty/SpringMVC中责任链的实现细节。



[**示例1**](https://github.com/maxwellyue/JavaLanguage/tree/master/design-pattern/pattern-responsibility-chain/src/main/java/com/maxwell/learning/designpattern/resposibilitychain)



# 参考

---

[责任链模式](http://www.runoob.com/design-pattern/chain-of-responsibility-pattern.html)

