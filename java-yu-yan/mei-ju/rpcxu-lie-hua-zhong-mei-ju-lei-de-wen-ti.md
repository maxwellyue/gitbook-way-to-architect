假定服务端做的序列化，客户端做的是反序列化，假定原来使用相同的API版本，但发生了以下情况：

**1、服务端枚举多了一个枚举值**

```java
//服务端
public enum Color {
  Red,
  Blue,
  Green;
}
//客户端
public enum Color {
  Red,
  Blue
}
```











**  
**













| 12345 | publicenum A {  X,  Y,  Z;} |
| :--- | :--- |


客户端的枚举类为

| 1234 | publicenum A {  X,  Y;} |
| :--- | :--- |


如果服务端返回一个A.Z给客户端，此时hessian反序列化调用枚举类的valueOf方法来获取反序列化，但是客户端的枚举类中没有Z，那么客户端反序列化直接跑异常。

2.服务端枚举ordinal值以及枚举类成员变量值和客户端不一致  
假设服务端的枚举类为

| 123456 | publicenum A {  X\("aaa"\),  Y\("bbbb"\);//此时Y的ordinal为1，对应的value为bbb  String value;  A\(String value\) {this.value=value}} |
| :--- | :--- |


客户端的枚举类为

| 1234567 | publicenum A {  X\("aaa"\),  Z\("ccc"\),  Y\("ddd"\);// 此时Y的ordinal为2对应的value为ddd  String value;  A\(String value\) {this.value=value;}} |
| :--- | :--- |


假如入服务端传递给客户端的是A.Y，此时客户端拿到的A.Y对应的ordinal为2，对应的value为ddd。  
上面这个点非常重要。  
3.枚举是单例的

| 1234567891011121314151617181920212223242526272829303132 | publicenum TestEnum {        XX\("xx"\);        TestEnum\(String value\) {        this.value = value;    }        String value;        public String getValue\(\) {        return value;    }        publicvoidsetValue\(String value\) {        this.value = value;    }}publicclassTest {publicstaticvoidmain\(String\[\] rgs\) {        TestEnum testEnum1 = TestEnum.XX;        TestEnum testEnum2 = TestEnum.XX;                testEnum1.setValue\("XX"\);        testEnum2.setValue\("YY"\);        System.out.println\(testEnum1.value\); // 输出 YY        System.out.println\(testEnum2.value\); // 输出 YY    }} |
| :--- | :--- |


testEnum1和testEnum2其实指向了同一个枚举引用。每次修改的都是同一个对象，所以前一个set的值被后面的set给覆盖了。

#### 三.总结 {#三-总结}

* 还是不要在RPC的接口中直接使用枚举类了，直接使用String就行
* 在枚举类中使用字符串时直接使用name\(\)就行，不要再做过度封装，尽量保持枚举类的简洁
* 枚举类使用在RPC接口上的时候就一定要小心，重构的时候要注意保持ordinal
* 枚举在序列化和反序列化的时候，除了name值，其他啥都不带的
* 禁止给枚举提供set方法，没用的



