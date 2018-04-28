# 一些小技巧

## 生产环境中如何不重启Tomcat更换JSP文件

如果你在生产环境中想要更换一个jsp文件，但又不想重启Tomcat，你可以这么干：

1、首先，用新的jsp文件替换掉旧的jsp文件

2、进入 `CATALINA_HOME/work/Catalina/localhost/contentName/org/apache/jsp` 目录，删除 `your_jsp.java` 和 `your_jsp.class` 文件。这两个文件将会在下次需要的时候重新被创建。

## Windows环境下如何让Tomcat开机后台启动

Windows下安装好了tomcat了以后，可以直接进如 `bin` 目录双击 `startup.bat` 来启动。

如何后台启动，步骤如下：

1、管理员身份运行cmd

2、目录切换到tomcat安装目录的bin目录下，执行 service.bat install tomcat\_xxxx

3、运行services.msc

4、找到tomcat服务（服务名为第2步指定的tomcat\_xxxx），设置开机启动即可







## 参考

[Why are my JSP changes are not reflected without restarting Tomcat?](https://stackoverflow.com/questions/2004676/why-are-my-jsp-changes-are-not-reflected-without-restarting-tomcat)



