# 其他事项

### 如何在foreach中进行break或者continue?

传统的循环写法中，我们可以使用break关键字来结束整个循环操作，使用continue关键字来跳过当前循环：

```java
List<String> colors = new ArrayList<>(Arrays.asList("white", "black", "red", "blue", "green"));
//当遇到颜色时，结束整个循环
for (String color : colors) {
    if (color.equals("red")) {
        break;
    }
    System.out.println(color);
}
//当遇到蓝颜色时，跳过本次循环
for (String color : colors) {
    if (color.equals("blue")) {
        continue;
    }
    System.out.println(color);
}
```

但`lambda`表达式的`foreach`并不支持这两个关键字，如果想要使用`break`或者`continue`这种语义，可以使用以下折中的办法：

#### **使用`return`关键字实现`continue`的语义**

```java
List<String> colors = new ArrayList<>(Arrays.asList("white", "black", "red", "blue", "green"));
colors.forEach(color -> {
    if (color.equals("blue")) {
        return;
    }
    System.out.println(color);
});
//输出如下：
white
black
red
green
```

#### 使用抛出异常的方式实现`break`的语义

```java
//首先需要自定义一个异常
public class BreakException extends RuntimeException {}
//使用该异常实现break的语义
List<String> colors = new ArrayList<>(Arrays.asList("white", "black", "red", "blue", "green"));
try {
    colors.forEach(color -> {
        if (color.equals("blue")) {
            throw new BreakException();
        }
        System.out.println(color);
    });
} catch (BreakException e) {
    System.out.println("foreach while break");
}
//输出如下
white
black
red
foreach while break
```

但是在实践中，并不推荐使用这种折中的方式去实现break或者continue的语义，即不应该使用foreach。而是应该根据程序的意图，去选择stream中提供的其他方法来达到目的。

比如，如果想要在一个集合中找到符合特定条件的第一个元素，可以使用`filter`和`findFirst`的组合：

```java
//在colors集合中，寻找blue
List<String> colors = new ArrayList<>(Arrays.asList("white", "black", "red", "blue", "green"));
Optional<String> target = colors.stream().filter(color -> color.equals("blue")).findFirst();
String targetColor = target.get();
System.out.println(targetColor);
//输出如下
blue
```

 再比如，如果想要判断一个元素在不在这个集合中，可以使用`anyMatch`这个方法：

```java
//判断blue在不在colors集合中
List<String> colors = new ArrayList<>(Arrays.asList("white", "black", "red", "blue", "green"));
boolean blueExists = colors.stream().anyMatch(color -> color.equals("blue"));
System.out.println(blueExists);
//输出如下
true
```

 

## 内容来源

[Break or return from Java 8 stream forEach?](https://stackoverflow.com/questions/23308193/break-or-return-from-java-8-stream-foreach)



