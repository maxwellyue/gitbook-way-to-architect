**ClassPathBeanDefinitionScanner**

在Spring中，负责读取Class文件中定义的BeanDefiniton的是ClassPathBeanDefinitionScanner。它会扫描类上标记的注解，如果发现符合条件的，就读取它来获取BeanDefiniton。这个条件，就是类上标注了特定的注解，这些特定的注解是是通过它的includeFilters变量来记录的，默认情况下，包含以下注解：

```

```

它内部使用AnnotationTypeFilter来表示需要







参考

[Spring类注册笔记      
](https://fangjian0423.github.io/2017/06/15/spring-bean-register-note/)

