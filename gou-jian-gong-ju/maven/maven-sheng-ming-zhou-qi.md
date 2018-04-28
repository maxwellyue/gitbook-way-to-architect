# Maven生命周期

Maven有三种不同的生命周期（Lifecycle），分别是Clean、Default、Site。每个生命周期又包含不同的阶段（Phase）。

当我们在运行一个命令的时候，该命令都会对应某个生命周期的某个阶段，并且该生命周期中的在此阶段之前的命令都会被执行。比如，`mvn clean` 等同于 `mvn pre-clean clean` ，如果我们运行 `mvn post-clean` ，那么` pre-clean，clean` 都会被运行。这是Maven很重要的一个规则，可以大大简化命令行的输入。

具体某个阶段应该做什么工作，怎么做，Maven有默认的行为，当然也可以通过插件进行定制修改。

理解上面这两段话的含义非常重要！

## **Clean Lifecycle** 

在进行真正的构建之前进行一些清理工作，Clean声明周期包含如下三个Phase。

* pre-clean  执行一些需要在clean之前完成的工作
* clean  移除所有上一次构建生成的文件
* post-clean  执行一些需要在clean之后立刻完成的工作

比如，我们在执行mvn clean的时候，其实是让Maven做它的Clean生命周期中的clean阶段应该去做的工作（当然，包含Clean生命周期中clean阶段之前的所有阶段的工作）。

## **Default Lifecycle**

构建的核心部分，编译，测试，打包，部署等等。  


## **Site Lifecycle**

生成项目报告，站点，发布站点。

* pre-site     执行一些需要在生成站点文档之前完成的工作
* site    生成项目的站点文档
* post-site     执行一些需要在生成站点文档之后完成的工作，并且为部署做准备
* site-deploy     将生成的站点文档部署到特定的服务器上



