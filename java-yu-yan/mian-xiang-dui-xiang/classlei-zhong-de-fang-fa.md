# Class类中的方法

---

```java
public native boolean isAssignableFrom(Class<?> cls);
```

isAssignableFrom：Assignable的意思是可分配的，clazz1.isAssignableFrom\(clazz2\)是说clazz1是否能从clazz2中分配，或者clazz1能否分配为clazz2，对应到Java中的接口/类这些概念中，意思是clazz2是否为clazz1的父类或者接口（或者说clazz1是否是clazz2的子类或者实现类），是的话，返回true。

例如：

```java
System.out.println(ArrayList.class.isAssignableFrom(List.class));//false
System.out.println(List.class.isAssignableFrom(ArrayList.class));//true
System.out.println(AbstractList.class.isAssignableFrom(ArrayList.class));//true
```



