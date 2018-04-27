# 常用命令

## 

#### 

#### 

#### 

#### 

#### 

#### 

#### 编译或打包时如何跳过测试

```text
# 使用-DskipTests：不执行测试用例，但编译测试用例类生成相应的class文件至target/test-classes下
mvn clean package -DskipTests
# 使用-Dmaven.test.skip=true：不执行测试用例，也不编译测试用例类。
mvn clean package -Dmaven.test.skip=true 
```

  


