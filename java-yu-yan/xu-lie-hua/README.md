# 序列化

## 为啥要序列化

Java序列化（即实现Serializable接口）的目的主要有两个：

* 网络传输
* 对象持久化（即文件存储）

## 序列化方式有哪些

* **Java自带的序列化方式**：实现Serializable接口
  * 缺点：
    * ①无法跨语言；
    * ②序列化后的码流太大；
    * ③序列化性能太低（即序列化过程太慢）
* **Google的Protobuf**
  * 使用二进制编码，在空间和性能上具有更大优势，很多RPC框架都选用`Protobuf`做编解码框架。
  * 它的主要特点如下：
    * 结构化数据存储格式（`XML`、`JSON`等）
    * 高效的编解码性能
    * 语言无关、平台无关、扩展性好
    * 支持多种语言：`Java、C++、Python、C++、Objective-C、C#、JavaNano、JavaScript、Ruby、Go、PHP、Dart`
* **Facebook的Thrift**

  //todo

* **JBoss Marshalling**

  //todo

* **Json**

  //todo

内容来源：

《Netty权威指南》

