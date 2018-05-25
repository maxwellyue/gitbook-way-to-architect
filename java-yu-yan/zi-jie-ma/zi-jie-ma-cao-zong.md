# 字节码操纵

最为流行的字节码操纵框架包括：

* [ASM](http://asm.ow2.org/)
* [AspectJ](http://www.eclipse.org/aspectj/)
* [BCEL](http://commons.apache.org/proper/commons-bcel/)
* [Byte Buddy](http://bytebuddy.net/#/)
* [CGLIB](https://github.com/cglib/cglib)
* [Cojen](https://github.com/cojen/Cojen/wiki)
* [Javassist](http://www.csg.ci.i.u-tokyo.ac.jp/~chiba/javassist/)
* [Serp](http://serp.sourceforge.net/)

这些字节码操纵框架的功能是修改类的字节码，具体怎么生效是不关心的，它们的工作仅仅是为从原始类到改装类这个过程中的操作提供便利。

TODO：具体API等到用到的时候再去看。



### 参考

[字节码操纵技术探秘](http://www.infoq.com/cn/articles/Living-Matrix-Bytecode-Manipulation#anch142537)

扩展阅读：

[通过使用Byte Buddy，便捷地创建Java Agent](http://www.infoq.com/cn/articles/Easily-Create-Java-Agents-with-ByteBuddy)

[skywalking源码分析之javaAgent工具ByteBuddy的应用](http://www.kailing.pub/article/index/arcid/178.html)





