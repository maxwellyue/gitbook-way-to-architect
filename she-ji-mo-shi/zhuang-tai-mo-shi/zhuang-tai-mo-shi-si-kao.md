上面的所有代码中演示的是：某一对象，在不同状态下，拥有同一方法声明，但具体实现不同。

还有另外一种情况就是：某一对象，需要在不同的状态之间进行切换，而状态转换时，需要采取的某些行为。这种情况的实际场景就是工作流（比如请假，你需要先申请，然后团队小组长审核，部门领导审核，xxx审核）。这种情况怎么办呢？状态模式可以解决这类问题吗？

答案是肯定的！

下面，使用状态模式来实现以下需求：

![](/assets/屏幕快照 2018-05-10 下午8.17.43.png)





Context

为了让代码更具通用性（比如，在其他流程如报销中，也可以重复使用一部分代码），引入StateMachine（其实就是Context的更上一层的抽象）

```java
//通用的Contex
public class StateMachine {

    //持有的状态对象
    private State state;

    //处理状态过程中需要的业务数据
    private Object businessVO;

    public void process(){
        state.process(this);
    }

    ...省略state和businessVO的getter和setter方法

}

//请假流程中使用的Context
public class LeaveRequestContext extends StateMachine {

}
```

State：同样引入了更上一层的接口

```java
//通用的State
public interface State{

    void process(StateMachine context);
}
//请假流程中使用的State
public interface LeaveRequestState extends State {

}
```

ConcreteState：有三个

```java
//项目经理状态
public class ProjectManagerState implements LeaveRequestState {

    @Override
    public void process(StateMachine request) {

        LeaveRequestModel model = (LeaveRequestModel) request.getBusinessVO();

        //模拟审批过程
        mockAudit(model);

        if("同意".equals(model.getResult())){
            if(model.getLeaveDays() < 3){
                request.setState(new AuditOverState());
            }else {        
                request.setState(new DepManagerState());
            }
        }else {   
            request.setState(new AuditOverState());
        }    
    }

    private void mockAudit(LeaveRequestModel model) {
        boolean res =  new Random().nextBoolean(); 
        if(res){
            model.setResult("同意");
        }else {
            model.setResult("不同意");
        }
    }
}

//部门经理状态
public class DepManagerState implements LeaveRequestState {

    @Override
    public void process(StateMachine request) {

        LeaveRequestModel model = (LeaveRequestModel) request.getBusinessVO();

        //模拟审批过程
        mockAudit(model);

        //将流程状态设置为结束
        request.setState(new AuditOverState());


    }

    private void mockAudit(LeaveRequestModel model) {
        boolean res =  new Random().nextBoolean(); 
        if(res){
            model.setResult("同意");
        }else {
            model.setResult("不同意");
        }
    }
}
//审批结束状态
public class AuditOverState implements LeaveRequestState {
    @Override
    public void process(StateMachine stateMachine) {
        LeaveRequestModel model = (LeaveRequestModel)stateMachine.getBusinessVO();
        System.out.println("请假审批结束，审批结果：" + model.getResult());
    }
}
```

业务数据

```java
public class LeaveRequestModel {

    //请假人
    private String user;
    //请假开始时间
    private String beginDate;
    //请假天数
    private int leaveDays;
    //请假结果
    private String result;

    ....省略以上属性的getter和setter方法


}
```

客户端：就是构建一个状态机，然后让状态机开始工作

```java
StateMachine context = new LeaveRequestContext();
context.setBusinessVO(model);
context.setState(new ProjectManagerState());
context.process();
```

在上面的示例中，客户端调用时，就是为了创建一个状态机，来对某一次流程进行处理，而构建状态机中需要的业务数据和状态机的当前状态，即business和state，这两个参数是需要保存在某个地方的，一般是数据库。

当然，也可以像上个投票的例子一样，将这些数据保存在Context中（这样仅仅是示例，实际应用中一般是持久化存储的）。

无论是投票的例子，还是工作流的例子：某个对象，对不同状态下，同一方法声明在不同的实现不同，这些不同的逻辑分开放在不同的状态类中，而不是用if-else全部写在同一方法中，因为实际场景下，某一个状态下，业务逻辑可能就很复杂。

对象的状态一般需要持久化到数据库中，而状态改变后，何时去进行下个状态的操作，取决于具体的场景：①有可能是定时去数据库获取特定状态的业务数据；②也有可能是等待用户去获取某特定状态的业务数据；





