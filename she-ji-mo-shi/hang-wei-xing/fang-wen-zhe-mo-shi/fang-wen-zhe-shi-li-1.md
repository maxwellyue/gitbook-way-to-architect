# 访问者实例1

现在要开发一套客户管理系统，公司的客户分为两种：企业客户和个人客户。现需要处理客户提出的服务请求、对客户进行行为预测以及分析客户价值。

Visitor

```java
public interface Visitor {

    public interface Visitor {

    void visitPersonalCustomer(PersonalCustomer customer);

    void visitEnterpriseCustomer(EnterpriseCustomer customer);
}
```

Element：客户Customer

```java
public abstract class Customer{
    public abstract void accept(Visitor visitor);
}
```

ConcreteElement：两种客户类型

```java
//个人客户
public class PersonalCustomer extends Customer {

    private String name;
    private int age;

     ... 省略getter setter...

    @Override
    public void accept(Visitor visitor) {
        visitor.visitPersonalCustomer(this);
    }
}

//企业用户
public class EnterpriseCustomer extends Customer {

    private String name;

    private String address;

    ... 省略getter setter...


    @Override
    public void accept(Visitor visitor) {
        visitor.visitEnterpriseCustomer(this);
    }
}
```

ConcreteVisitor

```java
//客户申请服务
public class ServiceRequestVisitor implements Visitor {

    @Override
    public void visitPersonalCustomer(PersonalCustomer customer) {
        System.out.println("个人客户" + customer.getName() + "提出服务请求");
    }

    @Override
    public void visitEnterpriseCustomer(EnterpriseCustomer customer) {
        System.out.println("企业客户" + customer.getName() + "提出服务请求");
    }
}

//客户行为预测
public class PredictionAnalyzeVisitor implements Visitor {

    @Override
    public void visitPersonalCustomer(PersonalCustomer customer) {
        System.out.println("对个人客户" + customer.getName() + "进行行为预测");
    }

    @Override
    public void visitEnterpriseCustomer(EnterpriseCustomer customer) {
        System.out.println("对企业客户" + customer.getName() + "进行行为预测");
    }
}
//客户价值分析
public class WorthAnalyzeVisitor implements Visitor {
    @Override
    public void visitPersonalCustomer(PersonalCustomer customer) {
        System.out.println("对个人客户" + customer.getName() + "进行价值分析");
    }

    @Override
    public void visitEnterpriseCustomer(EnterpriseCustomer customer) {
        System.out.println("对企业客户" + customer.getName() + "进行价值分析");
    }
}
```

ObjectStructure：客户集合

```java
public class CustomerStructure {

    private Collection<Customer> collection = new ArrayList<>();

    public void addElement(Customer customer){
        this.collection.add(customer);
    }

    public void handleRequest(Visitor visitor){
        for (Customer customer : collection){
            customer.accept(visitor);
        }
    }
}
```

Client

```java
public class Client {

    public static void main(String[] args) {
        CustomerStructure structure = new CustomerStructure();

        Customer xiaoming = new PersonalCustomer();
        ((PersonalCustomer) xiaoming).setName("xiaoming");

        Customer google=  new EnterpriseCustomer();
        ((EnterpriseCustomer) google).setName("google");

        structure.addElement(xiaoming);
        structure.addElement(google);

        //对客户进行行为预测
        Visitor predictionAnalyze = new PredictionAnalyzeVisitor();
        structure.handleRequest(predictionAnalyze);

        //客户申请服务
        Visitor serviceRequest = new ServiceRequestVisitor();
        structure.handleRequest(serviceRequest);

        //对客户进行价值分析
        Visitor worthAnalyze = new WorthAnalyzeVisitor();
        structure.handleRequest(worthAnalyze);
    }
}

//输出如下
对个人客户xiaoming进行行为预测
对企业客户google进行行为预测
个人客户xiaoming提出服务请求
企业客户google提出服务请求
对个人客户xiaoming进行价值分析
对企业客户google进行价值分析
```

可以看出，要对被访问者（通常是元素集合）增加新功能，只需要新增一个访问者类。 Element中accept哪种访问者，就可以调用该访问者实现的功能。

