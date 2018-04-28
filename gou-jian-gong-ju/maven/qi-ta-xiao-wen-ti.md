# 其他小问题

#### 1、Mac中Maven的setting.xml文件在哪？

`/Users/usename/.m2/settings.xml`或者`/usr/local/Cellar/maven/<version>/libexec/conf`

#### 2、如何将项目中WEB-INF/lib目录下的jar加入编译环境？

由于历史等原因，我们在项目中`WEB-INF/lib`目录下手动添加了一些jar文件，此时`Maven`并不知道它的存在的。  
此时，可以在`maven-compiler-plugin`插件中指定额外的编译参数即可，如下所示：

```text
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>2.3.2</version>
    <configuration>
        <source>1.7</source>
        <target>1.7</target>
        <encoding>utf-8</encoding>
        <compilerArguments>
            <!--将lib下的文件包含进编译目录-->
            <extdirs>${project.basedir}/src/main/webapp/WEB-INF/lib</extdirs>
        </compilerArguments>
    </configuration>
</plugin>
```

如果只有少量的jar，也可以通过如下方式解决。

```text
<dependency> 
    <groupId>org.apache</groupId> 
    <artifactId>test</artifactId> 
    <version>1.0</version> 
    <scope>system</scope> 
    <systemPath>${project.basedir}/src/main/webapp/WEB-INF/lib/xxxxxxxxx.jar</systemPath> 
</dependency> 
```

#### 3、使用maven运行Java main的2种方式

①直接在命令行运行

```text
mvn clean compile exec:java -Dexec.mainClass="com.demo.App" -Dexec.args="aaa bbb"  
```

②使用插件运行

```text
<plugin>
	<groupId>org.codehaus.mojo</groupId>
	<artifactId>exec-maven-plugin</artifactId>
	<version>1.4.0</version>
	<executions>
		<execution>
			<phase>test</phase>
			<goals>
				<goal>java</goal>
			</goals>
			<configuration>
				<mainClass>com.demo.App</mainClass>
				<arguments>
					<argument>aaa</argument>
					<argument>bbb</argument>
				</arguments>
			</configuration>
		</execution>
	</executions>
</plugin>
```

配置好插件后，执行`mvn test` 即可执行`main`方法

#### 4、如何删除仓库中下载失败的`.lastUpdated`文件？

通过Maven远程下载jar时，总出现下载到`.jar.lastUpdated`后缀的文件。此时，正常的jar包就下载不了，只能将其删除再次下载。

```text
# 执行以下命令（属于find和exec的联合用法，注意分号前的\必须保留）
find ~/.m2  -name "*.lastUpdated" -exec grep -q "Could not transfer" {} \; -print -exec rm {} \;
```

## 参考

[Where is Maven' settings.xml located on mac os?](https://stackoverflow.com/questions/3792842/where-is-maven-settings-xml-located-on-mac-os)

\[既使用maven编译，又使用lib下的Jar包\]\([https://blog.csdn.net/catoop/article/details/48489365](https://blog.csdn.net/catoop/article/details/48489365)\)

\[使用maven运行Java main的2种方式\]\([https://blog.csdn.net/mn960mn/article/details/49664701](https://blog.csdn.net/mn960mn/article/details/49664701)\)

  


