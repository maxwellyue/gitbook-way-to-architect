

springMVC默认支持的绑定类型有：

HttpServletReequest对象：通过request对象可以获取参数信息

HttpservletResponse对象：通过response对象可以处理响应信息

HTTPSession对象：获取session中存储的对象

Model/ModelMap：Model是一个接口，ModelMap是一个接口的实现。作用是将模型数据填充到request域。



1、直接在controller中定义一个变量，但是此种传输方式有一个限制就是参数名和请求中的参数名必须保持一致，否则是接收不到数据的。

比如说:

```
Controller : public void controllerTest(Integer id){} 
request : http://localhost:8080/controllerTest?id=2; 在这儿必须写成"id=2"而不能写成“id”这个属性名不可变
```

2、使用@RequestParam进行参数绑定，在使用这个注解进行绑定的时候，参数名无需和请求中的参数名保持一致。

比如说：

```
Controller : public void controllerTest(@RequestParam(value="id") Integer goods_id){} 
request : http://localhost:8080/springDemo/controllerTest?id=2; 在这儿传入的参数名为id
```

@RequestParam\(required=true\)，表示当前参数必须传入  
@RequestParam\(defaultValue="aaa"\)，表示默认值

3、POJO的绑定

在Controller中可以直接定义POJO类型的参数来接收请求中的数据。

这种使用方式的条件是：在页面中input的name属性的值必须和POJO的属性一一对应！




---
内容来源：
[springmvc中的controller中的几种参数绑定](http://blog.csdn.net/qq_16071145/article/details/51235735)



