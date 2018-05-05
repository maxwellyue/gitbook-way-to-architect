# 命令模式

---

**命令模式的本质**  
对命令封装，将命令发送者和命令接受者（或执行者）解耦。

**命令模式中的角色**

* **Command**： 声明一个给所有具体命令类的抽象接口

* **ConcreteCommand：**定义一个接收者和行为之间的弱耦合

* **Invoker：**负责调用命令对象执行请求

* **Receiver：**负责具体实施和执行一个命令或请求。

* **Cilent：**命令触发者或者说负责下达命令



**看代码来理解上面这5个角色的职责**

  
`Receiver`

```java
public class Receiver {
    /**
     * 真正执行命令
     */
    public void action() {
        System.out.println("执行操作");
    }
}
```

`Command`

```java
public interface Command {
    void execute();
}
```

`ConcreteCommand`

```java
public class ConcreteCommand implements Command {
    //持有相应的接收者对象
    private Receiver receiver = null;

    public ConcreteCommand(Receiver receiver) {
        this.receiver = receiver;
    }

    @Override
    public void execute() {
        //转调接收者的方法
        receiver.action();
    }
}
```

`Invoker`

```java
public class Invoker {
    //持有命令对象
    private Command command = null;

    public Invoker(Command command) {
        this.command = command;
    }

    public void action() {
        command.execute();
    }
}
```

`Client`

```java
public class Client {
    public static void main(String[] args) {
        //创建接收者
        Receiver receiver = new Receiver();
        //创建命令对象，设定其接收者
        Command command = new ConcreteCommand(receiver);
        //创建请求者，把命令对象设置进去
        Invoker invoker = new Invoker(command);
        //执行方法
        invoker.action();
    }
}
```



**通过录音机实例来理解**

小女孩茱莉\(Julia\)有一个盒式录音机，此录音机有播音Play、倒带Rewind、停止Stop功能。录音机的按钮便是请求者角色Invoker；茱莉\(Julia\)是客户端角色Client，而录音机便是接收者角色Receiver。

`Receiver`：由录音机扮演

```java
public class AudioPlayer {
    public void play() {
        System.out.println("播放……");
    }
    public void rewind() {
        System.out.println("倒带……");
    }
    public void stop() {
        System.out.println("停止……");
    }
}
```

Command

```java
public interface Command {
    void execute();
}
```

`ConcreteCommand`：有三个，`PlayCommand/RewindCommand/StopCommand`

```java
public class PlayCommand implements Command {
    private AudioPlayer audio;
    public PlayCommand(AudioPlayer audio) {
        this.audio = audio;
    }

    @Override
    public void execute() {
        audio.play();
    }
}

public class RewindCommand implements Command {
    private AudioPlayer audio;
    public RewindCommand(AudioPlayer audio) {
        this.audio = audio;
    }

    @Override
    public void execute() {
        audio.rewind();
    }
}
public class StopCommand implements Command {
    private AudioPlayer audio;
    public StopCommand(AudioPlayer audio) {
        this.audio = audio;
    }

    @Override
    public void execute() {
        audio.stop();
    }
}
```

`Invoker`：录音机上的按钮

```java
public class Keypad {
    private Command playCommand;
    private Command rewindCommand;
    private Command stopCommand;
    public void setPlayCommand(Command playCommand) {
        this.playCommand = playCommand;
    }
    public void setRewindCommand(Command rewindCommand) {
        this.rewindCommand = rewindCommand;
    }
    public void setStopCommand(Command stopCommand) {
        this.stopCommand = stopCommand;
    }

    public void play() {
        playCommand.execute();
    }

    public void rewind() {
        rewindCommand.execute();
    }

    public void stop() {
        stopCommand.execute();
    }
}
```

`Client`，茱莉小女孩

```java
public class Julia {
    public static void main(String[] args) {
        //创建接收者对象
        AudioPlayer audioPlayer = new AudioPlayer();
        //创建命令对象
        Command playCommand = new PlayCommand(audioPlayer);
        Command rewindCommand = new RewindCommand(audioPlayer);
        Command stopCommand = new StopCommand(audioPlayer);
        //创建请求者对象
        Keypad keypad = new Keypad();
        keypad.setPlayCommand(playCommand);
        keypad.setRewindCommand(rewindCommand);
        keypad.setStopCommand(stopCommand);
        //发送命令
        keypad.play();
        keypad.rewind();
        keypad.stop();
        keypad.play();
        keypad.stop();
    }
}
```



