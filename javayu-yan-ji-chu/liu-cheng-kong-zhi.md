#### 1、假设输入值为a=2，以下程序的输出结果是什么？

```
public void calculate(int a){
    int result = 0;
    switch (a){
        case 1:
            result = a;
        case 2:
            result = result + a*2;
        case 3:
            result = result + a*3;
    }
    System.out.println(result);
}
```

如果你的答案是10，那么恭喜你，可以跳过该小节内容。

考察的是：switch语句在没有break的情况下，会如何执行：**从匹配到的case开始，顺序执行之后的所有case，直到遇到break或者default**。

所以上面的程序会执行：

```
result = result + a*2;
result = result + a*3;
```

所以，最终输出结果是10。

正常写代码的时候，不要挑战这种错误用法，要在每种case后面，加上break。如果多种case需要执行相同的逻辑，可以使用：

```
case 常量表达式1: 
    doX();
    break;
case 常量表达式2: case 常量表达式3: 
    doY(2);
    break;
```

---

##### 2、使用switch时，可以使用字符串常量作为case标签吗？

case标签支持三种类型：①char\byte\short\int；②枚举常量；③从Java1.7开始，支持字符串字面量。

所以，可以。

