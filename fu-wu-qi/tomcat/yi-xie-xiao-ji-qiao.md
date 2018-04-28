# 一些小技巧

## 生产环境中如何不重启Tomcat更换JSP文件

如果你在生产环境中想要更换一个jsp文件，但又不想重启Tomcat，你可以这么干：

①首先，用新的jsp文件替换掉旧的jsp文件

②进入 `CATALINA_HOME/work/Catalina/localhost/contentName/org/apache/jsp` 目录，删除 `your_jsp.java` 和 `your_jsp.class` 文件。这两个文件将会在下次需要的时候重新被创建。

