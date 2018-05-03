1、经常有这样的事情发生，当你正在进行项目中某一部分的工作，里面的东西处于一个比较杂乱的状态，而你想转到其他分支上进行一些工作。问题是，你不想提交进行了一半的工作。解决这个问题的办法就是`git stash`命令。

现在你在分值aaa，已经做了一些修改，现在想要分支bbb做一些事情，但又不想提交aaa上的一些修改：

```
# 保存分支aaa的工作状态
git stash
# 切换到分支bbb
git checkout bbb
# 在分支bbb上做一些操作后，返回分支aaa
git checkout aaa
# 恢复之前的工作状态
git stash apply 或者 git stash pop
```



