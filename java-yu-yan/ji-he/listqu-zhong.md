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



