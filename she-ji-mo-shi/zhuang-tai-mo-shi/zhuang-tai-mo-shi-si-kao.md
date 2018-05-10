上面的所有代码中演示的是：某一对象，在不同状态下，拥有同一方法声明，但具体实现不同。

还有另外一种情况就是：某一对象，需要在不同的状态之间进行切换，而状态转换时，需要采取的某些行为。这种情况的实际场景就是工作流（比如请假，你需要先申请，然后团队小组长审核，部门领导审核，xxx审核）。这种情况怎么办呢？状态模式可以解决这类问题吗？

答案是肯定的！

下面，使用状态模式来实现以下需求：

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
                //小于三天，将流程状态设置为结束
                request.setState(new AuditOverState());
            }else {
                //大于三天，交给部门经理继续审核
                request.setState(new DepManagerState());
            }

        }else {
            //项目经理不同意，则将流程状态设置为结束
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





