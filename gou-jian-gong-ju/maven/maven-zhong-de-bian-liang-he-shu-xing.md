# Maven中的变量和属性

#### 1、**内置属性**

内置属性是Maven预定义的，用户可以直接使用，内置属性如下：

```bash
${basedir}                      表示项目根目录，即包含pom.xml文件的目录
${version}                      表示项目版本
${project.basedir}              同${basedir}
${project.baseUri}              表示项目文件地址
${maven.build.timestamp}        表示项目构件开始时间
${maven.build.timestamp.format} 表示属性${maven.build.timestamp}的展示格式,默认值为yyyyMMdd-HHmm
      可通过如下方式自定义其格式,其类型可参考java.text.SimpleDateFormat。
      <properties>
           <maven.build.timestamp.format>yyyy-MM-dd HH:mm:ss</maven.build.timestamp.format>
      </properties>
```

#### 2、**POM属性**

使用pom属性可以引用到pom.xml文件对应元素的值

```bash
${project.build.directory}         表示主源码路径;
${project.build.sourceEncoding}    表示主源码的编码格式;
${project.build.sourceDirectory}   表示主源码路径;
${project.build.finalName}         表示输出文件名称;
${project.version}                 表示项目版本,与${version}相同;
```

#### 3、**自定义属性**

是指在pom.xml文件的&lt;properties&gt;标签下定义的Maven属性

```markup
<project> 
    ...
    <properties> 
        <my.pro>abc</my.pro> 
    </properties>
    ...
</project>
```

这样就可以在其他地方用`${my.pro}`使用该属性值。

#### 4、**系统属性和**环境变量属性

所有的系统属性都可以使用Maven属性引用，比如`${user.home}`表示用户主目录。

所有的环境变量都可以用以env.开头的Maven属性引用，比如`${env.JAVA_HOME}`表示`JAVA_HOME`环境变量的值。

小技巧1：使用`mvn help:system`命令查看所有属性

小技巧2：Java代码中使用`System.getProperties()`可得到所有的属性

```text
# 使用mvn help:system命令，就会列出如下信息（有删减，混个脸熟）
===============================================================================
System Properties
===============================================================================
java.vm.version=25.131-b11
path.separator=:
java.vm.name=Java HotSpot(TM) 64-Bit Server VM
user.dir=/Users/yue/Documents/workspace/idea/meicai/trace-demo
os.name=Mac OS X
classworlds.conf=/usr/local/maven/3.5.0/apache-maven-3.5.0/bin/m2.conf
sun.jnu.encoding=UTF-8
java.library.path=/Users/yue/Library/Java/Extensions:/Library/Java/Extensions:/Network/Library/Java/Extensions:/System/Library/Java/Extensions:/usr/lib/java:.
maven.conf=/usr/local/maven/3.5.0/apache-maven-3.5.0/conf
java.class.version=52.0
os.version=10.13.4
user.home=/Users/yue
file.encoding=UTF-8
user.name=yue
java.home=/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/jre
sun.arch.data.model=64
user.language=zh
java.vm.info=mixed mode
java.version=1.8.0_131
maven.home=/usr/local/maven/3.5.0/apache-maven-3.5.0
file.separator=/

===============================================================================
Environment Variables
===============================================================================
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home
JAVA_7_HOME=/Library/Java/JavaVirtualMachines/jdk1.7.0_79.jdk/Contents/Home
HOME=/Users/yue
M2_HOME=/usr/local/maven/3.5.0/apache-maven-3.5.0/bin
JAVA_8_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home
XPC_SERVICE_NAME=0
Apple_PubSub_Socket_Render=/private/tmp/com.apple.launchd.06WaNyP61O/Render
LOGNAME=yue
GOBIN=/usr/local/Cellar/go/1.9.2/bin
USER=yue
SHELL=/bin/zsh
ZSH=/Users/yue/.oh-my-zsh
CLASSPATH=.:/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/lib/dt.jar:/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/lib/tools.jar
GOPATH=/usr/local/Cellar/go/1.9.2
GRADLE_USER_HOME=/Users/yue/.gradle
GRADLE_HOME=/Users/yue/Library/Android/gradle/gradle-3.5.1
```

#### 5、**settings.xml文件属性**

与pom属性同理，用户使用以`settings.`开头的属性引用`settings.xml`文件中的`XML`元素值，比如`${settings.localRepository}`表示本地仓库的地址。

