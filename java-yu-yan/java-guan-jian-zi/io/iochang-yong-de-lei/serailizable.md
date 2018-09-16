# Serializable

Java 中通过Serializable接口来提供**对象序列化机制：**将对象转换为字节序列，而反序列化即将字节序列转换为对象，一般用来持久化对象或网络传输对象。

以Serializable来标识类是否可以序列化与反序列化，背后是通过**`ObjectInputSteram/ObjectOutputStream`**来完成这些工作的。

```java
//ObjectOutputStream：序列化
private void writeObject0(Object obj, boolean unshared) throws IOException {
      ...
    if (obj instanceof String) { 
        writeString((String) obj, unshared);
    } else if (cl.isArray()) { 
        writeArray(obj, desc, unshared);
    } else if (obj instanceof Enum) {
        writeEnum((Enum) obj, desc, unshared);
    } else if (obj instanceof Serializable) {
        writeOrdinaryObject(obj, desc, unshared);
    } else {
        if (extendedDebugInfo) {
            throw new NotSerializableException(cl.getName() + "\n"
                    + debugInfoStack.toString());
        } else {
            throw new NotSerializableException(cl.getName());
        }
    }
    ...
} 
//ObjectInputSteram：反序列化
public final Object readObject()
        throws IOException, ClassNotFoundException
    {
        if (enableOverride) {
            return readObjectOverride();
        }

        // if nested read, passHandle contains handle of enclosing object
        int outerHandle = passHandle;
        try {
            Object obj = readObject0(false);
            handles.markDependency(outerHandle, passHandle);
            ClassNotFoundException ex = handles.lookupException(passHandle);
            if (ex != null) {
                throw ex;
            }
            if (depth == 0) {
                vlist.doCallbacks();
            }
            return obj;
        } finally {
            passHandle = outerHandle;
            if (closed && depth == 0) {
                clear();
            }
        }
    }
```

从上述代码可知，如果被写对象的类型是String，或数组，或Enum，或Serializable，那么就可以对该对象进行序列化，否则将抛出NotSerializableException。

对于一个类，如果不想将某个字段进行序列化，可以使用关键字transient来标识：

```java
private transient String name;
```

当对该类序列化时，会自动忽略被 transient 修饰的属性。



