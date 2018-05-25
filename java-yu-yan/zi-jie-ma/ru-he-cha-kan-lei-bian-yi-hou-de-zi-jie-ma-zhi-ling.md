# 如何查看类编译后的字节码指令

方式一：使用javap命令

```text
javap /path/to/Xxxxxx.Class
```

方式二：IDEA 中，使用Byte Editor插件查看

下载[Bytecode Editor](https://plugins.jetbrains.com/plugin/8461-bytecode-editor)插件，安装后，在IDEA中点开某个类的class文件（注意是java代码编译后生成的class文件），点击`View> Show Bytecode`即可

