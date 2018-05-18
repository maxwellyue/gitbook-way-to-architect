# 组合模式续

注意到，上面的示例代码中，叶子节点由于不支持add/remove/getChild等操作，不得不提示用户不支持次操作或为空实现，这样就会造成很多代码重复。解决办法有两个：

**透明方式**

将add、remove、getChild等操作在Component抽象类中提供默认实现，这样只需在支持这些操作的Composite中重写这些方法即可。

```java
public abstract class AbstractFile {  
    public void add(AbstractFile file) {  
        System.out.println("对不起，不支持该方法！");  
    }  

    public void remove(AbstractFile file) {  
        System.out.println("对不起，不支持该方法！");  
    }  

    public AbstractFile getChild(int i) {  
        System.out.println("对不起，不支持该方法！");  
        return null;  
    }  

    public abstract void killVirus();  
}
```

之所以说是透明，是指客户端在使用Composite或Leaf时，所有的方法都是一样的，对用户来说是一致的。

其实，对于叶子节点来说，因为叶子节点不可能有下一个层次的对象，因此为其提供add\(\)、remove\(\)以及getChild\(\)等方法是没有意义的。对叶子节点使用add等方法是不安全的（假如没有提供相应的错误处理）。

**安全方式**

这种方式中，COmponent中只保留Leaf和Composite的公共操作（例子中的`killVirus()`），Lead中不会出现add/remove等方法，只在Composite中保留add/remove等方法。这种方式虽然安全，但是客户端不能完全面向抽象编程。在实际应用中，安全组合模式的使用频率也非常高，在Java AWT中使用的组合模式就是安全组合模式。

## 组合模式适用场景

在以下情况下可以考虑使用组合模式：

\(1\) 在具有整体和部分的层次结构中，希望通过一种方式忽略整体与部分的差异，客户端可以一致地对待它们。

\(2\) 在一个使用面向对象语言开发的系统中需要处理一个树形结构。

\(3\) 在一个系统中能够分离出叶子对象和容器对象，而且它们的类型不固定，需要增加一些新的类型。

## 参考

内容来自：[组合模式-Composite Pattern](https://gof.quanke.name/组合模式-Composite%20Pattern.html)

