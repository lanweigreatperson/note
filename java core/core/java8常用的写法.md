### List->Map

Example:

~~~java
public List listToMap(list) {
    return list.stream().map(e -> e.getName()).colllect(Collectors.toList());
}

public Map<String, List> listToMap(list) {
    return list.stream().colllect(Collectors.toMap(k -> k, v -> Lists.newArrayList(v.getName()), (List l1, List l2) -> {
        l1.addAll(l2);
        return l1;
    }));
}
~~~

### 字符拆分

~~~java
// 最好使用guava带的Splitter， 健壮性
Map<String, String> typeLimit = Splitter.on(",").omitEmptyStrings().trimResults().withKeyValueSeparator(":").split(limitConfig);
List<String> list = Splitter.on(",").omitEmptyStrings().trimResults().splitToList(config);
~~~

