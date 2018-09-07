### 一行代码初始化ArrayList

很多时候，我们需要初始化一个集合，元素可能就是String或者Integer这种简单数据类型，最常规的方法如下：

```java
List<String> colors = new ArrayList<String>();
colors.add("red");
colors.add("white");
colors.add("black");
```

但如上写法，显得繁琐，可考虑使用如下方式：

```java
①匿名内部类的形式
List<String> list = new ArrayList<String>() {{
    add("red");
    add("white");
    add("black");
}};

②借助Arrays.asList()方法
List<String> list = new ArrayList<String>(Arrays.asList("red", "white", "black"));

③如果只是list，不要求ArrayList，可直接使用Arrays.asList()方法，但要注意此时集合不可变
List<String> list = Arrays.asList("red", "white", "black");

④如果只是想创建不可变集合，可借助guava提供的ImmutableList
List<String> list = ImmutableList.of("red", "white", "black");
⑤如果只是想创建不可变集合，且只有一个元素，除了借助guava提供的ImmutableList，还可以使用JDK自带的Collections
List<String> list = Collections.singletonList("color");

⑥如果只是想创建某个对象的n副本集合
List<String> list = Collections.nCopies(1000, "red");

⑦自己写工具类
public static <T> ArrayList<T> createArrayList(T ... elements) {
        ArrayList<T> list = new ArrayList<T>();
        for (T element : elements) {
            list.add(element);
        }
        return list;
}
```

## List去重

方式1：循环对比，可以保持原来的顺序

```java
List<Integer> list = new ArrayList<>(Arrays.asList(6, 2, 2, 6, 8));
for (int i = 0; i < list.size() - 1; i++) {
    for (int j = list.size() - 1; j > i; j--) {
        if (list.get(j).equals(list.get(i))) {
            list.remove(j);
        }
    }
}
//去重之后
6,2,8
```

方式2：通过HashSet/LinkedHashSet重复元素

```java
//使用HashSet，不保证顺序，：2,6,8
List<Integer> list = new ArrayList<>(Arrays.asList(6, 2, 2, 6, 8));
HashSet set = new HashSet(list);
list.clear();
list.addAll(set);

//使用LinkedHashSet，保证顺序，：6,2,8
List<Integer> list = new ArrayList<>(Arrays.asList(6, 2, 2, 6, 8));
HashSet set = new LinkedHashSet(list);
list.clear();
list.addAll(set);
```

方式3：通过新集合和list.contain\(\)，可以保持原来的顺序

```java
List<Integer> list = new ArrayList<>(Arrays.asList(6, 2, 2, 6, 8));

List newList = new ArrayList();
for (int i = 0; i < list.size(); i++) {
    if (!newList.contains(list.get(i))) {
        newList.add(list.get(i));
    }
}
//newList：6,2,8
```



