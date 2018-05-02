在项目的开发中，一般会有dev、test、statge、prod等环境的区分。

如何管理变量在不同环境中的值？

有多种实践方式

### 第一种

①在项目的根目录下建立如下pom文件：

```
pom.xml(默认)
pom_test.xml
pom_prod.xml
pom_dev.xml
```

②打包时，指定`pom`文件：

```
# 生产环境
mvn clean package -f pom_prod.xml
# 测试环境
mvn clean package -f pom_test.xml
# 开发环境
mvn clean package -f pom_dev.xml
```

由于多个 pom.xml 之间重复配置很多，不容易维护，实际中不推荐使用这种方式。

### 第二种

①在项目根目录中建立如下目录

```
├──profiles 
|   |
|   |——test
│   │   └──my.properties
│   ├──prod    
│   │   └──my.properties
│   └──dev    
│       └──my.properties
```

②在pom文件中，建立对应的profile

```
<profiles>
    <profile>
        <!--默认使用该配置-->
        <id>dev</id>
        <properties>
            <package.environment>dev</package.environment>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <profile>
        <id>test</id>
        <properties>
            <package.environment>test</package.environment>
        </properties>
    </profile>
    <profile>
        <id>prod</id>
        <properties>
            <package.environment>prod</package.environment>
        </properties>
    </profile>
</profiles>
```

③在pom中配置打包插件

```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <version>2.4</version>
    <configuration>
        <resourceEncoding>utf-8</resourceEncoding>
        <webappDirectory>${project.build.directory}/${project.artifactId}</webappDirectory>
        <warName>${project.artifactId}</warName>
        <webResources>
            <!--这里的文件会copy到targetPath下，覆盖掉执行resource:resource目标时复制文件-->
            <resouce>
                <directory>${project.basedir}profiles/${package.environment}</directory>
                <targetPath>WEB-INF/classes</targetPath>
                <!-- 替换占位符 -->
                <filtering>true</filtering>
            </resouce>
        </webResources>
    </configuration>
</plugin>
```

④打包时，指定环境

```
# 开发环境
mvn clean package -Pprod
# 测试环境
mvn clean package -Ptest
# 开发环境
mvn clean package -Pdev
```

> 注意：
>
> * 配置文件所在目录不一定建在项目根目录下，也可以建立在于java文件平级的地方，或者其他任何地方，只要在打包插件中指定就可以了。
> * 目录中配置文件可以有多个，打包时都会包含进去

### 第三种

这种与第二种类似，只是无需新建配置文件，而是直接将变量配置在pom文件中。

①在pom文件中的profile中配置变量

```
<profiles>
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <!-- jdbc -->
            <jdbc_username>xxxx</jdbc_username>
            <jdbc_password>yyyyyy</jdbc_password>
        </properties>
    </profile>

    <profile>
        <id>test</id>
        <properties>
            <!-- jdbc -->
            <jdbc_username>xxxx</jdbc_username>
            <jdbc_password>yyyyyy</jdbc_password>
            <dbcp_max_active>10</dbcp_max_active>
            <dbcp_max_idle>5</dbcp_max_idle>
            <dbcp_min_idle>2</dbcp_min_idle>
        </properties>
    </profile>

    <profile>
        <id>prod</id>
        <properties>
            <jdbc_username>xxxx</jdbc_username>
            <jdbc_password>yyyyyy</jdbc_password>
            <dbcp_max_active>10</dbcp_max_active>
            <dbcp_max_idle>5</dbcp_max_idle>
            <dbcp_min_idle>2</dbcp_min_idle>
        </properties>
    </profile>
</profiles>
```

②在pom中配置resource中的filter为true（目的：将其他地方的占位符替换为pom中配置的真正的值）

```
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <filtering>true</filtering>
    </resource>
</resources>
```

③打包时，指定环境

```
# 开发环境
mvn clean package -Pprod
# 测试环境
mvn clean package -Ptest
# 开发环境
mvn clean package -Pdev
```



个人更倾向于使用第二种方式。







