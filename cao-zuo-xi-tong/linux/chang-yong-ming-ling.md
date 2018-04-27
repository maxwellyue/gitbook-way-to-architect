# 常用命令

## 根据端口查看进程并杀死

```bash
# 查看某端口的进程
lsof -i:端口号
# 杀死进程
kill -9 PID
```

## 创建一个文件

①`vim/vi foo.txt`：文件已存在，则打开，不存在，则新建打开（为空）  
②`echo "aaaa" > foo.txt`  
③`cat > foo.txt`  
④`emacs foo.txt`  
⑤`touch foo.txt`

## 怎么看一个Java线程的资源耗用

①使用`top`或者`ps -ef | grep java`找到该线程对应的进程，假设pid为22385  
②使用命令`top -p 22385`查看该进程  
③在`top`界面，按`shift+h`，查看该进程的所有线程的信息，此时pid列（除了第一个）即为线程pid，假设要查看线程pid为22399的线程  
④使用`jstack 22385|grep -A 10 577f`查看该线程的信息，其中577f是22399的16进制表示，-A 10表示查找到所在行的后10行。

## Thread dump文件如何分析

在dump中，线程一般存在如下几种状态：①`RUNNABLE`，线程处于执行中；②`BLOCKED`，线程被阻塞；③`WAITING`，线程正在等待。  
使用`jstack pid > threaddump.txt`命令将进程id为pid的java进程的线程信息输出到文件threaddump.txt中。  
查看类似java.lang.Thread.State: WAITING \(parking\)这种信息，可以看到各线程的当前状态。

## 找出占用空间最大的文件

```bash
$ cd/path/to/some/where
$ du -hsx * | sort -rh | head -10
//命令解释
du : 计算出单个文件或者文件夹的磁盘空间占用.
sort : 对文件行或者标准输出行记录排序后输出.
head : 输出文件内容的前面部分.
```

## /etc/hosts文件什么做用

hosts文件的作用相当于DNS，提供IP地址hostname的对应。早期的互联网计算机少，单机hosts文件里足够存放所有联网计算机。不过随着互联网的发展，这就远远不够了。于是就出现了分布式的DNS系统，由DNS服务器来提供类似的IP地址到域名的对应。Linux系统在向DNS服务器发出域名解析请求之前会查询/etc/hosts文件，如果里面有相应的记录，就会使用hosts里面的记录

## 硬链接和软链接的区别

TODO

## cat命令

①一次显示整个文件`cat filename`  
②创建一个新文件`cat > filename`只能创建新文件,不能编辑已有文件.  
③将几个文件合并为一个文件`cat file1 file2 > file`

```bash
-A, --show-all           等价于 -vET
-b, --number-nonblank    对非空输出行编号
-e                       等价于 -vE
-E, --show-ends          在每行结束处显示 $
-n, --number     对输出的所有行编号，由1开始对所有输出的行数编号
-s, --squeeze-blank  有连续两行以上的空白行，就代换为一行的空白行 
-t                       与 -vT 等价
-T, --show-tabs          将跳格字符显示为 ^I
-u                       (被忽略)
-v, --show-nonprinting   使用 ^ 和 M- 引用，除了 LFD 和 TAB 之外
```

## echo命令

①输出字符串

```bash
[root@iZ2zeap997asuc4yr0bw77Z /]
# echo hahhahah
hahhahah
```

②将字符串写到文件中\(使用&gt;&gt;表示在原有内容基础上追加，使用&gt;表示清空原来内容，替换\)

```bash
[root@iZ2zeap997asuc4yr0bw77Z /]# touch aa.txt
[root@iZ2zeap997asuc4yr0bw77Z /]# echo hahahahah >>aa.txt
[root@iZ2zeap997asuc4yr0bw77Z /]# cat aa.txt
hahahahah
```

③显示命令的结果

```bash
[root@iZ2zeap997asuc4yr0bw77Z /]# echo`date`
2017年 12月 12日 星期二 11:11:49 CST
```

## 以某一个用户的身份执行某一个命令

在控制台中以某个用户的身份运行一条命令可以用 

```bash
# 命令格式
su -c "command" user
# 当前登录用户是xiaohong，现在要以xiaoming的身份执行/home/www/test.sh 这个脚本
su -c “/home/www/test.sh” xiaoming
(前提是，xiaoming对/home/www/test.sh这个脚本文件有执行权限)
```

## 切换用户身份

```bash
# 当前用户为小明，需要切换到root用户
sudo su -
# 从root用户切换到xiaoming用户
su - xiaoming
```

> 注意：- 与 -l 是一样的，都表示要切换到后面指定的用户（未指定，则默认为root），并加载其对应的环境变量

## 远程主机或特定端口是否可达

```bash
# 查看192.168.12.21是否可达
ping 192.168.12.21
# 查看192.168.12.21的6379端口是否可达
telnet 192.168.12.21 6379
```



