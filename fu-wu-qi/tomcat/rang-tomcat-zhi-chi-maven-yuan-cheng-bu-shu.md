# 让Tomcat支持Maven远程部署

第一步：修改` {tomcat}/webapps/manager/META-INF/context.xml`文件

```text
# 原来的内容
<Context antiResourceLocking="false" privileged="true" >
    <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
</Context>
# 修改为
<Context antiResourceLocking="false" privileged="true" >
    <!--
        <Valve className="org.apache.catalina.valves.RemoteAddrValve"
             allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" />
     -->   
</Context>
```

第二步：修改_`{tomcat_home}/conf/tomcat-users.xml`_文件，在节点`tomcat-users`中添加两个角色和一个用户（注意要修改用户名和密码，防止有人恶意进入）。

```text
<role rolename="manager-gui"/>
<role rolename="manager-script"/>
<user username="xiaoming" password="xiaoming_123456" roles="manager-script,manager-gui"/>
```

> 下面是tomcat文档中对于各个角色的权限说明：

```text
- manager-gui — Access to the HTML interface.
- manager-status — Access to the "Server Status" page only.
- manager-script — Access to the tools-friendly plain text interface that is described in this document, and to the "Server Status" page.
- manager-jmx — Access to JMX proxy interface and to the "Server Status" page.
```

 第三步：在项目中的pom.xml文件的 build &gt; plugins节点中添加远程部署插件：

```markup
<!--远程部署插件-->
<plugin>
    <groupId>org.apache.tomcat.maven</groupId>
    <artifactId>tomcat7-maven-plugin</artifactId>
    <version>2.2</version>
    <configuration>
        <url>http://192.168.12.21:8080/manager/text</url>
        <username>xiaoming</username>
        <password>xiaoming_123456</password>
        <path>/${project.artifactId}</path>
    </configuration>
</plugin>
```

OK，现在可以部署了：

```text
mvn tomcat7:redeploy -Ptest
```

## 参考

[Access Tomcat Manager App from different host](https://stackoverflow.com/questions/36703856/access-tomcat-manager-app-from-different-host)

