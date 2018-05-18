# 其他问题

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

#### 5、properties文件采用`${xxx}`赋值，真正的值是配置在`pom`的`profile`中，但是未生效？

```text
<build>        
    <resources>            
        <resource>   
              <directory>${project.basedir}/src/main/resources</directory>      
             <!-- 加入这一行 -->
            <filtering>true</filtering>   
        </resource>        
        <resource>        
            <directory>${project.basedir}/bin</directory> 
               <targetPath>/bin</targetPath>   
                <!-- 加入这一行 -->             
               <filtering>true</filtering>            
        </resource>        
    </resources>    
</build>
```

#### 6、maven下载慢的解决方法

修改`${maven.home}/conf`或者`${user.home}/.m2`文件夹下的`settings.xml`文件，在`<mirrors>`标签下加入如下内容即可。

```text
<mirror>
     <id>alimaven</id>
     <mirrorOf>central</mirrorOf>
     <name>aliyun maven</name>
     <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
</mirror>
```

#### 7、新到公司，配置私服地址？

假如公司私服为：`https://mvn.maxwell.com/nexus/#welcome`

提取关键词：`maxwell.com`

按照如下修改即可。

```text
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

    <mirrors>
        <mirror>
            <id>maxwell</id>
            <mirrorOf>*</mirrorOf>
            <url>http://mvn.maxwell.com/nexus/content/groups/public/</url>
        </mirror>
    </mirrors>

    <profiles>
        <profile>
            <id>dev</id>
            <!--仓库地址-->
            <repositories>
                <repository>
                    <id>maxwell</id>
                    <url>http://mvn.maxwell.com/nexus/content/groups/public/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </repository>
            </repositories>
            <!--插件库地址-->
            <pluginRepositories>
                <pluginRepository>
                    <id>maxwell</id>
                    <url>http://mvn.maxwell.com/nexus/content/groups/public/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
    </profiles>

    <activeProfiles>
        <!--激活生效dev-->
        <activeProfile>dev</activeProfile>
    </activeProfiles>
</settings>
```

#### 8、有一天，Maven管理的项目不再使用Maven了，需要将pom中的依赖都下载到项目中的lib目录下，怎么办？

```bash
# -DoutputDirectory  指定输出目录
# -DincludeScope 指定复制哪些依赖
mvn dependency:copy-dependencies -DoutputDirectory=/xxx/yyy/lib -DincludeScope=compile
```





















## 参考

[Where is Maven' settings.xml located on mac os?](https://stackoverflow.com/questions/3792842/where-is-maven-settings-xml-located-on-mac-os)

[既使用maven编译，又使用lib下的Jar包](https://blog.csdn.net/catoop/article/details/48489365)

[使用maven运行Java main的2种方式](https://blog.csdn.net/mn960mn/article/details/49664701)

[清空仓库中lastUpdated文件](https://my.oschina.net/lhplj/blog/201832)

[maven如何过滤占位符](http://www.cnblogs.com/xiohao/p/5080327.html)

[使用Maven filter和profile隔离不同环境的配置文件](https://my.oschina.net/jackieyeah/blog/716503)  


  


