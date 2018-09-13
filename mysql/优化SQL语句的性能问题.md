## 定位及优化SQL语句的性能

在现如今的软件开发中，关系型数据库是做数据存储最重要的工具。无论是`Oracle`还是`Mysql`，都是需要通过`SQL`语句来和数据库进行交互的，这种交互我们通常称之为`CRUD`。在`CRUD`操作中，最最常用的也就是Read操作了。而对于不同的表结构，采用不同的SQL语句，性能上可能千差万别。本文，就基于`MySql`数据库，来介绍一下如何定位SQL语句的性能问题。

对于低性能的SQL语句的定位，最重要也是最有效的方法就是使用执行计划。

![img](assets/640)

#### 1. 执行计划

我们知道，不管是哪种数据库，或者是哪种数据库引擎，在对一条SQL语句进行执行的过程中都会做很多相关的优化，对于查询语句，最重要的优化方式就是使用索引。

而执行计划，就是显示数据库引擎对于SQL语句的执行的详细情况，其中包含了是否使用索引，使用什么索引，使用的索引的相关信息等。

![img](assets/640)

基本语法

```
explain select ...
```

`mysql`的explain 命令可以用来分析select 语句的运行效果。

除此之外，explain 的extended 扩展能够在原本explain的基础上额外的提供一些查询优化的信息，这些信息可以通过`mysql`的show warnings命令得到。

```sql
mysql> explain extended select * from account;
******** 1. row ***************************
          id: 1
select_type: SIMPLE
       table: account
        type: ALL
possible_keys: NULL
         key: NULL
     key_len: NULL
         ref: NULL
        rows: 1
    filtered: 100.00
       Extra:
1 row in set, 1 warning (0.00 sec)

mysql> show warnings;
*************1. row ***************************
Level: Note
  Code: 1003
Message: select `dbunit`.`account`.`id` AS `id`,`dbunit`.`account`.`name` AS `name` from `dbunit`.`account`
1 row in set (0.00 sec)
```

另外，对于分区表的查询，需要使用partitions命令。

```
explain partitions select ...
```

#### 2. 执行计划包含的信息

不同版本的Mysql和不同的存储引擎执行计划不完全相同，但基本信息都差不多。mysql执行计划主要包含以下信息:

![img](assets/640)

#### **id**

由一组数字组成。表示一个查询中各个子查询的执行顺序;

- id相同执行顺序由上至下。

![img](assets/640)

- id不同，id值越大优先级越高，越先被执行。

![img](assets/640)

- id为`null`时表示一个结果集，不需要使用它查询，常出现在包含`union`等查询语句中。

![img](assets/640)

#### **select_type**

每个子查询的查询类型，一些常见的查询类型。

| id   | select_type  | description                               |
| ---- | ------------ | ----------------------------------------- |
| 1    | SIMPLE       | 不包含任何子查询或union等查询             |
| 2    | PRIMARY      | 包含子查询最外层查询就显示为 `PRIMARY`    |
| 3    | SUBQUERY     | 在`select`或 `where`字句中包含的查询      |
| 4    | DERIVED      | `from`字句中包含的查询                    |
| 5    | UNION        | 出现在`union`后的查询语句中               |
| 6    | UNION RESULT | 从UNION中获取结果集，例如上文的第三个例子 |

#### **table**

查询涉及到的数据表。

如果查询使用了别名，那么这里显示的是别名，如果不涉及对数据表的操作，那么这显示为null，如果显示为尖括号括起来的<derived N>就表示这个是临时表，后边的N就是执行计划中的id，表示结果来自于这个查询产生。如果是尖括号括起来的<union M,N>，与<derived N>类似，也是一个临时表，表示这个结果来自于union查询的id为M,N的结果集。

#### **type**

访问类型

- `ALL`   扫描全表数据
- `index` 遍历索引
- `range` 索引范围查找
- `index_subquery` 在子查询中使用 ref
- `unique_subquery` 在子查询中使用 eq_ref
- `ref_or_null` 对`Null`进行索引的优化的 ref
- `fulltext` 使用全文索引
- `ref`   使用非唯一索引查找数据
- `eq_ref` 在`join`查询中使用`PRIMARY KEY`or`UNIQUE NOT NULL`索引关联。
- `const` 使用主键或者唯一索引，且匹配的结果只有一条记录。
- `system const` 连接类型的特例，查询的表为系统表。

![img](assets/640)

性能从好到差依次为：system，const，eq_ref，ref，fulltext，ref_or_null，unique_subquery，index_subquery，range，index_merge，index，ALL，除了ALL之外，其他的type都可以使用到索引，除了index_merge之外，其他的type只可以用到一个索引。

**所以，如果通过执行计划发现某张表的查询语句的type显示为ALL，那就要考虑添加索引，或者更换查询方式，使用索引进行查询。**

#### **possible_keys**

可能使用的索引，注意不一定会使用。查询涉及到的字段上若存在索引，则该索引将被列出来。当该列为 `NULL`时就要考虑当前的`SQL`是否需要优化了。

#### **key**

显示MySQL在查询中实际使用的索引，若没有使用索引，显示为NULL。

`TIPS:`查询中若使用了覆盖索引(覆盖索引：索引的数据覆盖了需要查询的所有数据)，则该索引仅出现在key列表中。

select_type为index_merge时，这里可能出现两个以上的索引，其他的select_type这里只会出现一个。

#### **key_length**

索引长度 char()、varchar()索引长度的计算公式：

```
(Character Set：utf8mb4=4,utf8=3,gbk=2,latin1=1) * 列长度 + 1(允许null) + 2(变长列)
```



其他类型索引长度的计算公式: ex:

```
CREATE TABLE `student` (
 `id` int(11) unsigned NOT NULL AUTO_INCREMENT,
 `name` varchar(128) NOT NULL DEFAULT '',
 `age` int(11),
 PRIMARY KEY (`id`),
 UNIQUE KEY `idx` (`name`),
 KEY `idx_age` (`age`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4;
```

name 索引长度为： 编码为utf8mb4,列长为128,不允许为`NULL`,字段类型为`varchar(128)`。`key_length = 128 * 4 + 0 + 2 = 514;`

![img](assets/640-1536112472841)

age 索引长度：int类型占4位，允许`null`,索引长度为5。

![img](assets/640-1536112114991)

#### **ref**

表示上述表的连接匹配条件，即哪些列或常量被用于查找索引列上的值

如果是使用的常数等值查询，这里会显示const，如果是连接查询，被驱动表的执行计划这里会显示驱动表的关联字段，如果是条件使用了表达式或者函数，或者条件列发生了内部隐式转换，这里可能显示为func

#### **rows**

返回估算的结果集数目，注意这并不是一个准确值。

#### **extra**

`extra`的信息非常丰富，常见的有： 

1. Using index 使用覆盖索引
2. Using where 使用了用where子句来过滤结果集
3. Using filesort 使用文件排序，使用非索引列进行排序时出现，非常消耗性能，尽量优化。
4. Using temporary 使用了临时表。

#### 3.一些SQL优化建议

**1、SQL语句不要写的太复杂。**

一个SQL语句要尽量简单，不要嵌套太多层。

**2、使用『临时表』缓存中间结果。**

简化SQL语句的重要方法就是采用临时表暂存中间结果，这样可以避免程序中多次扫描主表，也大大减少了阻塞，提高了并发性能。

**3、使用like的时候要注意是否会导致全表扫**

有的时候会需要进行一些模糊查询比如

```
select id from table where username like ‘%hollis%’
```

关键词%hollis%，由于hollis前面用到了“%”，因此该查询会使用全表扫描，除非必要，否则不要在关键词前加%，

**4、尽量避免使用!=或<>操作符**

在where语句中使用!=或<>，引擎将放弃使用索引而进行全表扫描。

**5、尽量避免使用 or 来连接条件**

在 where 子句中使用 or 来连接条件，引擎将放弃使用索引而进行全表扫描。

```
可以使用

select id from t where num=10
union all
select id from t where num=20

替代
select id from t where num=10 or num=20
```

**6、尽量避免使用in和not in**

在 where 子句中使用 in和not in，引擎将放弃使用索引而进行全表扫描。

```
可以使用

select id from t where num between 10 and 20

替代
select id from t where num in (10,20)
```

**7、可以考虑强制查询使用索引**

```
select * from table force index(PRI) limit 2;(强制使用主键)

select * from table force index(hollis_index) limit 2;(强制使用索引"hollis_index")

select * from table force index(PRI,hollis_index) limit 2;(强制使用索引"PRI和hollis_index")
```

**8、尽量避免使用表达式、函数等操作作为查询条件**

**9、尽量避免大事务操作，提高系统并发能力。**

**10、尽量避免使用游标**

**11、任何地方都不要使用 select \* from t ，用具体的字段列表代替“*”，不要返回用不到的任何字段。**

**12、尽可能的使用 varchar/nvarchar 代替 char/nchar**

**13、尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。**

**14、索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率**

**15、并不是所有索引对查询都有效，SQL是根据表中数据来进行查询优化的，当索引列有大量数据重复时，SQL查询可能不会去利用索引**