工作这么多年都没有好好总结下常用的java集合。有些时候被问到，还得想想。现在总结下，后续将对部分集合做源码分析：这个也是经常面试被问到的。

**Java最常用的集合类**

- Java最常用的集合类
  - List接口，允许元素重复，如：`ArrayList`、`LinkedList`、`Vector`、`Stack`、`CopyOnWriteArrayList`
  - Set接口，不允许有重复元素，如：`ConcurrentSkipListSet`、`CopyOnWriteArraySet`、`EnumSet`、`HashSet`、`LinkedHashSet`、`TreeSet`
- Map接口
  - `HashMap`、`TreeMap`、`ConcurrentHashMap`、`EnumMap`、`LinkedHashMap`、`HashTable`

**Collection需要掌握的重点方法**

- `add(E)`-->往集合中添加对象
- `remove(E)`-->删除集合中的对象
- `get(index)` -->获取索引为index的对象
- `iterator()`-->迭代器
- `contain(E)`-->是否存在某元素
- `sort(Comparator)`-->排序

**Map需要掌握的重点方法**

- `put(Object key, Object value) `-->添加元素
- `remove(Object key) ` -->删除key的元素
- `get(Object key) ` -->获取key对应的value
- `containsKey(Object key) `-->是否存在key
- `keySet() `-->key的Set集合





