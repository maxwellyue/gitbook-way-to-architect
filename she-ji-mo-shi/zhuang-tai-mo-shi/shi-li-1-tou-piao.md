考虑一个在线投票系统的应用，要实现控制同一个用户只能投一票，如果一个用户反复投票，而且投票次数超过5次，则判定为恶意刷票，要取消该用户投票的资格，当然同时也要取消他所投的票；如果一个用户的投票次数超过8次，将进入黑名单，禁止再登录和使用系统。

要使用状态模式实现，首先需要把投票过程的各种状态定义出来，根据以上描述大致分为四种状态：正常投票、反复投票、恶意刷票、进入黑名单。然后创建一个投票管理对象（相当于Context）。

Status

```java
public interface VoteState {
    /**
     * 处理状态对应的行为
     * 
     * @param user    投票人
     * @param voteItem    投票项
     * @param voteManager    投票上下文
     *                        
     */
    public void vote(String user,String voteItem,VoteManager voteManager);
}
```

ConceteState

```java
//正常投票
public class NormalVoteState implements VoteState {

    @Override
    public void vote(String user, String voteItem, VoteManager voteManager) {
        //正常投票，记录到投票记录中
        voteManager.getMapVote().put(user, voteItem);
        System.out.println("投票成功");
    }

}

//重复投票
public class RepeatVoteState implements VoteState {

    @Override
    public void vote(String user, String voteItem, VoteManager voteManager) {
        //重复投票，暂时不做处理
        System.out.println("请不要重复投票");
    }

}

//恶意投票
public class SpiteVoteState implements VoteState {

    @Override
    public void vote(String user, String voteItem, VoteManager voteManager) {
        // 恶意投票，取消用户的投票资格，并取消投票记录
        String str = voteManager.getMapVote().get(user);
        if(str != null){
            voteManager.getMapVote().remove(user);
        }
        System.out.println("你有恶意刷屏行为，取消投票资格");
    }

}

//黑名单
public class BlackVoteState implements VoteState {

    @Override
    public void vote(String user, String voteItem, VoteManager voteManager) {
        //记录黑名单中，禁止登录系统
        System.out.println("进入黑名单，将禁止登录和使用本系统");
    }

}
```

Context

```java
public class VoteManager {

    private VoteState state = null;

    //记录用户投票的结果，<用户名称，投票的选项>
    private Map<String,String> mapVote = new HashMap<String,String>();

    //记录用户投票次数，<用户名称，投票的次数>
    private Map<String,Integer> mapVoteCount = new HashMap<String,Integer>();

    public Map<String, String> getMapVote() {
        return mapVote;
    }

    public void vote(String user,String voteItem){
        //1.为该用户增加投票次数
        Integer oldVoteCount = mapVoteCount.get(user);
        if(oldVoteCount == null){
            oldVoteCount = 0;
        }
        oldVoteCount += 1;
        mapVoteCount.put(user, oldVoteCount);

        //2.判断该用户的投票类型
        if(oldVoteCount == 1){
            state = new NormalVoteState();
        }
        else if(oldVoteCount > 1 && oldVoteCount < 5){
            state = new RepeatVoteState();
        }
        else if(oldVoteCount >= 5 && oldVoteCount <8){
            state = new SpiteVoteState();
        }
        else if(oldVoteCount > 8){
            state = new BlackVoteState();
        }
        //3.转调状态对象来进行相应的操作
        state.vote(user, voteItem, this);
    }
}
```

客户端

```java
public class Client {

    public static void main(String[] args) {

        VoteManager vm = new VoteManager();
        for(int i = 0;i < 9; i++){
            vm.vote("user-1","A");
        }
    }

}

//输出如下
投票成功
请不要重复投票
请不要重复投票
请不要重复投票
你有恶意刷屏行为，取消投票资格
你有恶意刷屏行为，取消投票资格
你有恶意刷屏行为，取消投票资格
你有恶意刷屏行为，取消投票资格
进入黑名单，将禁止登录和使用本系统
```



