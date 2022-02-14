# 数据库
## 1. mysql
### 1.1 设计原则
  + 数据类型按照实际情况，范围或者占用空间更小的通常更好
  + 尽量避免NUll
  + datetime 能存储1001年到9999年，精度为秒。与时区无关，使用8个字节。
  + timestamp 保存1970年至今的秒数，只使用4个字节，范围是1970-2038，显示依赖时区。
  + 满足三大范式：
    + 第一范式：属性不可再分；
    + 第二范式：为了减少冗余，所有列都只和其中一列（主键）相关；如果有和多个列相关的可以拆开多个表；
    
      | 学号 |  姓名  |  专业名 | 专业主任 | 课程名称 | 分数 |
      | ---- |  ----  |  ---- | ---- | ---- | ---- |
      
      表可以拆为：
      
      | 学号 |   课程名称 | 分数 |
      | ---- | ---- | ---- |
      
      | 学号 |  姓名  |  专业名 | 专业主任 |
      | ---- |  ----  |  ---- | ----  |
    + 第三范式：每列都需要与主键直接相关，不能间接相关
    
      | 学号 |   课程名称 | 分数 |
      | ---- | ---- | ---- |
      
      | 学号 |   姓名 | 专业名 |
      | ---- | ---- | ---- |
      
      | 专业名 | 专业主任 |
      | ---- | ---- |
### 1.2 事务
  + 事务的四个基本特性：
    + 原子性（Atomicity）：一个事务是一个整体，要么全部执行成功，要么都不执行。
    + 一致性（Consistency）：事务执行前后，数据从一个状态到另一个状态必须是一致的（本来应该+，结果出来成-了）。
    + 隔离性（Isolation）：多个并发事务之间相互隔离，一个不影响另一个。
    + 持久性（Durablity）：事务执行完后，对数据库修改是永久的，没法回滚。
  + 事务并发常见问题
    + 脏读：A事务执行过程中，B事务修改了数据，导致A事务读取了修改后的数据，但是B最后又回退了，所以让A读取了错误的数据。
    + 不可重复读：A执行过程中，B修改了数据，导致A前后读取的数据不一致。
    + 幻读：A执行过程中，B增加了数据，导致A前后读取不一致。
  + 针对事务并发隔离的4个级别：MYSQL默认可重复读，设置隔离级别：SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
    + 读未提交：能读到未提交的事务中所做的修改，这种会产生脏读。性能不会好太多，问题很多，很少用。
    + 读已提交：只能获取已经提交事务后的结果，也就是说事务中所做的修改对其他事务是不可见的。解决的脏读问题。
    + 可重复读：读取一个数据前给数据加共享行锁，更新时加排它行锁，这样就避免在事务中数据被其他事务修改，解决了不可重复读问题。
    + 可串行化：加表锁防止事务并行。innodb通过MCVV快照方式解决幻读问题。
  + innodb处理死锁是将持有最少行级拍他锁的事务进行回滚
### 1.3 索引
  + 为什么用b+树
    + 列表等有序数据：数据量大了不能全部加载到内存来查找
    + 二叉树：可以保存树的前几层，但是极端情况，二叉树就和有序列表一样。
    + 平衡二叉树：没法范围查询
    + B树：
      1. 所有节点都存数据，B+树只有叶子节点存数据，读写磁盘B+树就相对能读的节点多；
      2. B树可能在父节点就查到数据，有可能在子节点，查询效率不稳定，B+树每次都需要查询到子节点；
      3. B+树数据都在叶子节点并且相连，分支都是索引，方便扫库，适合范围查询，而B树只能遍历整个树；
  + 最左原则：联合索引创建时索引是按照从左到右的顺序排列的，比如（a, b）则是先按照a排序，a相同时按照b排序。
  比如a>1 and b=4 只有a走索引，因为在a>1的情况下b是无序的。最左原则就是：最左优先，从左边开始任何连续索引能匹配上，遇到范围查询就回停止匹配。
  + 聚簇索引和非聚簇索引：  
    聚集索引：数据和主键聚集在一起，聚簇索引只能有一个，就是表本身的一种排列方式，类似新华字典正文内容本身就是一种按照一定规则排列的目录。  
    非聚集索引：就是主键和数据分离，主键指向数据，索引是纯粹主键的索引，可以有多个，这种目录纯粹是目录，正文纯粹是正文的排序方式。
    ```
    1）直接创建CREATE INDEX indexName ON mytable(username(length)); 
    2）修改表结构：CREATE INDEX indexName ON mytable(username(length)); 
    3）创建表时创建索引：
    CREATE TABLE mytable(  
    ID INT NOT NULL,   
    username VARCHAR(16) NOT NULL,  
    INDEX [indexName] (username(length))  
    );
    1.添加PRIMARY KEY（主键索引）
    ALTER TABLE `table_name` ADD PRIMARY KEY ( `column` )
    2.添加UNIQUE(唯一索引)
    ALTER TABLE `table_name` ADD UNIQUE ( `column` )
    3.添加INDEX(普通索引)
    ALTER TABLE `table_name` ADD INDEX index_name ( `column` )
    4.添加FULLTEXT(全文索引)
    ALTER TABLE `table_name` ADD FULLTEXT ( `column`)
    5.添加多列索引
    ALTER TABLE `table_name` ADD INDEX index_name ( `column1`, `column2`, `column3` )
    ```

### 1.4 引擎
  + 存储结构  
    MyISAM ：每个MyISAM在磁盘上存储成三个文件。分别： .frm文件存储表定义   .MYD是数据文件的扩展名   .MYI是索引文件的扩展名。(默认)  
    InnoDB ：所有的表都保存在同一个数据文件中。Innodb表的大小只受限于操作系统文件的大小，一般为2GB。（需要制定）  
    所以如果备份、迁移或者删除时，MyISAM比较方便。
  + 存储空间  
    MyISAM：可被压缩，存储空间小。支持三种不同的存储格式：静态表、动态表、压缩表。  
    Innodb：需要更多的内存和存储，它会在主内存中建立专用的缓冲池用于高速缓冲数据和索引。
  + 事务支持  
    MyISAM ：强调的是性能 但不提供事务支持。  
    Innodb：提供事务支持、外部键等高级数据库功能，有事务、回滚、崩溃修复能力。
  + GURD操作  
    MyISAM：不支持行级锁，在增删时会锁定整个表格，效率不如Innodb.  
    Innodb：支持行级锁，在增删时效率更高。
### 1.5 查询和优化
  + 查询步骤
    ```
    (1) FROM <left_table>
    (2) <join_type> JOIN <right_table>
    (3) ON <join_condition>
    (4) WHERE <where_condition>
    (5) GROUP BY <group_by_list>
    (6) WITH {CUBE | ROLLUP}
    (7) HAVING <having_condition>
    (8) SELECT
    (9) DISTINCT
    (9) ORDER BY <order_by_list>
    (10) <TOP_specification> <select_list>
    ```  
     每个步骤都会产生一个虚拟表，作为下一个步骤的输入。
  + 数据库优化  
    ```
      1.优化索引、SQL 语句、分析慢查询; 
      2.设计表的时候严格根据数据库的设计范式来设计数据库;
      3.使用缓存，把经常访问到的数据而且不需要经常变化的数据放在缓存中，能节约磁盘 IO 
      4.优化硬件;采用 SSD，使用磁盘队列技术(RAID0,RAID1,RDID5)等
      5.采用 MySQL 内部自带的表分区技术，把数据分层不同的文件，能够提高磁盘的读取效率; 
      6.垂直分表;把一些不经常读的数据放在一张表里，节约磁盘 I/O; 
      7.主从分离读写;采用主从复制把数据库的读操作和写入操作分离开来; 
      8.分库分表分机器(数据量特别大)，主要的的原理就是数据路由; 
      9.选择合适的表引擎，参数上的优化
      10.进行架构级别的缓存，静态化和分布式;
      11.不采用全文索引;
    ```
  + 查询分析  
  EXPLAIN：可以显示SQL的执行计划。
    ```angular2
    MySQL [rules]> explain select rule from ac_rule;
    +----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------+
    | id | select_type | table   | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
    +----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------+
    |  1 | SIMPLE      | ac_rule | NULL       | ALL  | NULL          | NULL | NULL    | NULL | 2274 |   100.00 | NULL  |
    +----+-------------+---------+------------+------+---------------+------+---------+------+------+----------+-------+
    1 row in set, 1 warning (0.01 sec)
    ```
    id：select执行顺序标识符，从大到小执行。如果在语句中没子查询或关联查询，只有唯一的 SELECT，每行都将显示 1。否则，内层的 SELECT 语句一般会顺序编号，对应于其在原始语句中的位置。id相同表示为一组，执行顺序由上至下。  
    select_type：有很多，如：    
      + SIMPLE：表示简单select，不使用关联或者自查询。
      + PRIMARY：查询中若包含任何复杂的子部分,最外层的select被标记为PRIMARY
      + UNION：UNION中的第二个或后面的SELECT语句 
       
    table：表示引用那个表  
    type：重要指标，表示数据访问类型，从好到坏：system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL。一般来说，得保证查询至少达到 range 级别，最好能达到 ref（使用了索引，但不是唯一索引）。  
    possible_keys：显示查询使用了哪些索引，表示该索引可以进行高效地查找，但是列出来的索引对于后续优化过程可能是没有用的。  
    key：显示 MySQL 实际决定使用的键（索引）。如果没有选择索引，键是 NULL。要想强制 MySQL 使用或忽视 possible_keys 列中的索引，在查询中使用 FORCE INDEX、USE INDEX 或者 IGNORE INDEX。  
    key_len：显示 MySQL 决定使用的键长度。如果键是 NULL，则长度为 NULL。使用的索引的长度。在不损失精确性的情况下，长度越短越好。  
    ref： 列显示使用哪个列或常数与 key 一起从表中选择行。  
    rows： 列显示 MySQL 认为它执行查询时必须检查的行数。注意这是一个预估值。  
    filtered：给出了一个百分比的值，这个百分比值和 rows 列的值一起使用。(5.7才有)  
    Extra： 是 EXPLAIN 输出中另外一个很重要的列，该列显示 MySQL 在查询过程中的一些详细信息，MySQL 查询优化器执行查询的过程中对查询计划的重要补充信息。
    
### 1.6 表关联
    
### 1.7 SQL语句
  + 创建表：
    ```angular2
    CREATE TABLE IF NOT EXISTS `runoob_tbl`(
    `runoob_id` INT UNSIGNED AUTO_INCREMENT,
    `runoob_title` VARCHAR(100) NOT NULL,
    `runoob_author` VARCHAR(40) NOT NULL,
    `submission_date` DATE,
    PRIMARY KEY ( `runoob_id` )
    )ENGINE=InnoDB DEFAULT CHARSET=utf8;
    ```
  + 修改表：
    ```
    ALTER TABLE testalter_tbl MODIFY c CHAR(10);
    ALTER TABLE testalter_tbl DROP i;
    ALTER TABLE testalter_tbl ADD i INT FIRST;
    ALTER TABLE testalter_tbl DROP i;
    ALTER TABLE testalter_tbl ADD i INT AFTER c;
    ALTER TABLE testalter_tbl RENAME TO alter_tbl;
    ```
  + 查数据：
    ```
    MySQL 的 WHERE 子句的字符串比较是不区分大小写的。 你可以使用 BINARY 关键字来设定 WHERE 子句的字符串比较是区分大小写的。
    SELECT * from runoob_tbl WHERE BINARY runoob_author='runoob.com';
    更新：UPDATE table_name SET field1=new-value1, field2=new-value2 [WHERE Clause]
    删除：DELETE FROM table_name [WHERE Clause]
    排序：SELECT field1, field2,...fieldN table_name1, table_name2...
    ORDER BY field1, [field2...] [ASC [DESC]]
    分组：SELECT column_name, function(column_name)
    FROM table_name
    WHERE column_name operator value
    GROUP BY column_name;
    分组后的条件使用 HAVING 来限定，WHERE 是对原始数据进行条件限制。几个关键字的使用顺序为 where 、group by 、having、order by ，例如：
    SELECT name ,sum(*)  FROM employee_tbl WHERE id<>1 GROUP BY name  HAVING sum(*)>5 ORDER BY sum(*) DESC;
    ```
## 2. pgsql

  | 区别点 |   mysql | pg |
  | ---- | ---- | ---- |
  | cpu | 最多使用128 | 无限制 |
  | 版本 | 版本分支多，之间不兼容 | 版本统一只有社区版本，开源免费 |
  | 索引 | 索引类型少 | 多 |
  | 插件 | 不支持 | 丰富 |
  | 数据类型 | 少 | 多，如 数组、ip |
  | 跨库 | 支持跨库 | 不支持跨库查询 |
  | 迭代 | 慢，2-3年 | 快，1年 |
## 3. es
+ 解决问题：
  + 数据库字段太多，查询太慢，索引没有办法再做优化；
  + 数据量大
  + 全文检索
+ 概念：开源搜索引擎，可以处理pb级结构或者非结构数据。
  + Cluster&Node：ES是一个分布式数据库，每台服务器可以运行多个es实例，单个实例称为一个Node，一组实例对外提供服务称为一个集群（cluster）。  
  + index（索引）：一类数据的集合，相当于关系数据库的一个数据库。
  + type（逻辑分组）：类似表，最新默认_doc，我理解type就相当于一个类，这个类下面的数据都是这个类的实例，也就是说type下数据格式都相似。
  + document（文档）：类似行，一组json数据。es是存储一个个文档（json数据）
+ 操作：
  + 添加和更新：
    ```
    curl -H "Content-Type: application/json" -X POST http://localhost:9200/project/person/1 -d '{"name":"James","age":35}'
    ```
    POST是插入数据，可以指定id，不指定会自动生成。  
    PUT是更新数据，必须指定，指定后会覆盖数据。上述命令中project是index,person是type，1是id。  
    服务器返回如下：
    ```
    {
        "_index": "project",
        "_type": "person",
        "_id": "1",
        "_version": 2,
        "result": "updated",
        "_shards": {
            "total": 2,
            "successful": 1,
            "failed": 0
        },
        "_seq_no": 1,
        "_primary_term": 1
    }
    ```
    服务器返回的 JSON 对象，会给出 Index、Type、Id、Version 等信息。其中result的值为updated，说明这条id的数据已经存在，所以这次再插入就是更新。
  + 删除数据：
    ```
    curl -X DELETE http://localhost:9200/project/person/1
    ````
  + 查询数据：
    ```
    curl http://localhost:9200/project/person/1
    ```
    后面可以加参数, 
    ?pretty=true 表示以易读格式返回
    返回的易读格式如下：

    ```
    {
      {
      "_index" : "project",
      "_type" : "person",
      "_id" : "1",
      "_version" : 1,
      "_seq_no" : 4,
      "_primary_term" : 1,
      "found" : true,
      "_source" : {
        "name" : "James",
        "age" : 35
      }
    }
    ```
  + 检索：
      ```
      curl -H "Content-Type: application/json" http://localhost:9200/project/person/_search?pretty=true -d '{"query":{"match":{"age":35}}}'
      ```
    默认一次返回10条结果，可以通过size字段改变这个设置：
    参数{"query":{"match":{"age":35}},"from":10, "size":1}表示从第10条结果开始返回一条。
+ 倒排索引
  + 字段：es默认会自动根据数据类型生成字段类型，同时可以设置mapping指定字段类型。一个字段可以同时拥有多种类型，可以在mapping中通过field指定。
    ```
    PUT my_index
    {
      "mappings": {
        "_doc": {
          "properties": {
            "cityName": {
              "type": "text",
              "fields": {
                "raw": { 
                  "type":  "keyword",
                  "ignore_above" : 256
                }
              }
            }
          }
        }
      }
    }
    ```
    text会分词后存储，keyword不分词。
    cityName字段拥有text类型，也可以通过cityName.raw来使用，此时是keyword类型。对超过 ignore_above 的字符串，analyzer 不会进行处理；所以就不会索引起来。导致的结果就是最终搜索引擎搜索不到了。
  + 索引：如果要是实现搜到包含某个字段得数据，关系数据库需要遍历全表，效率低。ES使用倒排索引解决上面问题。  
    关系数据库中索引为正向索引，通过key，找value。
    倒排索引也是反向索引，通过value找key。
    1. 先将数据分词
    2. 保存各个分词和数据之间关系，如：  
    
    | 单词ID | 单词 | 文档出现频率 | 倒排信息(docID;TF;<POS>) |
    | ---- | ---- | ---- | ---- |
    | 1 | 谷歌 | 5 | (1;1;<1>), (2;2;<4;7>), (3;1;<4>), (4;1;<1>), (5;1;<1>)|
    | 2 | 地图 | 4 | (1;1;<1>), (2;2;<1;8>), (4;1;<4>), (5;1;<1>) |
    | 3 | 之父 | 4 | (2;2;<1;5>), (3;1;<4>), (4;1;<1>), (5;1;<1>) |
    | 4 | 特斯拉 | 2 | (2;2;<1;6>), (5;1;<1;4>) |
    | 5 | 离开 | 1 | (4;3;<1;4;5>) |
    比如单词ID5，表示‘离开’有一个文档包含此单词，在ID为4的文档中，出现了3次，位置分别为1、4、5
+ 各种查询关键字区别
  + term：不分词,所以查询keyword时查询内容需要和数据完全一致；查询text时需要查询内容和分词后的数据某个词一致。
  + match：会分词，所以查询keyword时查询内容需要和数据完全一致；查询text时只要查询的数据分词中有一个和分词后的数据某个词一致。
  + match_phrase：也分词，所以查询keyword时查询内容需要和数据完全一致；查询text时需要查询的数据分词结果必须再数据分词结果中都包含，并且顺序相同，也必须连续。
  + query_string：分词，但是查不出keyword；查询text时需要查询的数据分词结果必须再数据分词结果中都包含，不需要顺序。
## 4. redis
+ 基础数据类型，指的都是数据值的类型，key都是string
  + string：是二进制安全的，也就是可以存数字、字符串、图片或者序列化的对象。
    1. 命令使用：
    
        | 命令 | 简介 | 使用 | 
        | ---- | ---- | ---- | 
        | GET | 获取存储在指定key的值 | GET name |
        | SET | 设置存储在指定key的值 | SET name James |
        | DEL | 删除存储在指定key的值 | DEL name |
        | INCR | key的值+1 | INCR name |
        | DECR | key的值-1 | DECR name |
        | INCRBY | key的值+n | INCRBY name n|
        | DECRBY | key的值-n| DECRBY name n|
    2. 使用场景：
      + 缓存：经典使用场景，把常用信息，字符串，图片或者视频等信息放到redis中，redis作为缓存层，mysql做持久化层，降低mysql的读写压力。
      + 计数器：redis是单线程，一个命令结束才会执行另一个
      + session：一般web用来保存登录信息
  + List：双向链表结构
    1. 命令使用：
    
        | 命令 | 简介 | 使用 | 
        | ---- | ---- | ---- | 
        | RPUSH | 在列表右边添加值 | RPUSH key value |
        | LPUSH | 在列表左边添加值 | LPUSH key value |
        | RPOP | 从列表右边弹出一个值并返回 | RPOP key |
        | LPOP | 从列表左边弹出一个值并返回 | LPOP key |
        | LRANGE | 获取列表中指定范围的值 | LRANGE key 0 -1 |
        | LINDEX | 获取列表中指定索引的值.0是第一个，-1是最后一个 | LINDEX key -1|
    2. 使用场景：
      + 微博TimeLine: 有人发布微博，用lpush加入时间轴，展示新的列表信息。
      + 消息队列
  + Set：通过hash实现，值无序，不重复
    1. 命令使用：
    
        | 命令 | 简介 | 使用 | 
        | ---- | ---- | ---- | 
        | SADD | 向集合添加一个或者多个成员 | SADD key value1,value2...  |
        | SCARD | 获取集合的成员数 | SCARD key |
        | SMEMBER | 返回集合的所有成员 | SMEMBER key |
        | SISMEMBER | 判断集合是否存在该成员，存在返回1 | SISMEMBER key merber |
    2. 使用场景：
      + 标签: 给一个用户打标签，标签不会重复，也无序。
      + 点赞：文章中的那些用户点赞或者收藏
  + Hash：类似python中字典
    1. 命令使用：
    
        | 命令 | 简介 | 使用 | 
        | ---- | ---- | ---- | 
        | HSET | 添加键值对 | HSET name key value  |
        | HGET | 获取指定键的值 | HGET name key |
        | HGETALL | 获取所有的键值对 | HGETALL name |
        | HDEL | 判断集合是否存在该成员，如果有则删除 | HGETALL name key |
    2. 使用场景：
      + 缓存: 比String节省空间
  + Zset：有序集合的成员是唯一的,但分数(score)却可以重复。集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。
    1. 命令使用：
    
        | 命令 | 简介 | 使用 | 
        | ---- | ---- | ---- | 
        | ZADD | 添加成员及其对应的分数 | ZADD name 98 小明  |
        | ZSCORE | 获取成员的分数 | ZSCORE name 小明 |
        | ZRANGE | 按照分数从小到大返回start-stop之间的成员[带分数] | ZRANGE name 0-1 [withccores] |
        | ZREVRANGE | 按照分数从大到小返回start-stop之间的成员[带分数] | ZRANGE name 0-1 [withccores] |
        | ZREM | 判断集合是否存在该成员，如果有则删除 | ZREM name key |
    2. 使用场景：
      + 排行榜: 榜单可以按照用户关注数，更新时间，字数等打分，做排行。
+ 事务：Redis 事务的本质是一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

    | 命令 | 简介 | 说明 | 
    | ---- | ---- | ---- | 
    | MULTI  | 开启一个事务 | 后面所有命令会被加入一个执行队列，除非遇到结束命令  |
    | EXEC | 执行事务中所有命令 | 开始顺序执行队列中所有命令，失败后也会接着执行其他命令 |
    | DISCARD | 取消事务 | 清空当前事务中命令，不再执行，后续命令也不加入队列 |
    | WATCH | 监视一个或多个key,如果事务在执行前，这个key(或多个key)被其他命令修改，则事务被中断，不会执行事务中的任何命令。 | 事务执行结束后WATCH会被自动取消 |
    | UNWATCH | 主动放弃监视 | 取消监视后不管监视后是否有修改都不会影响事务 |
+ 缓存问题：数据库大多数情况都是用户并发访问最薄弱的环节。所以，就需要使用redis做一个缓冲操作，让请求先访问到redis，而不是直接访问Mysql等数据库。这样可以大大缓解数据库的压力。
  
  | 问题 | 原因 | 方案 | 
  | ---- | ---- | ---- | 
  | 缓存穿透 | 缓存和数据库中都没有的数据，而用户不断发起请求。比如查询id为-1，会每次都查询数据库，频繁的攻击会导致数据库压力过大。 | 1. 参数校验；2. 数据库没查到就缓存为null,过期时间设置小点；3. 布隆过滤器让查询快速 | 
  | 缓存击穿 | 某条缓存过期导致缓存没有该数据，数据库中有。大量并发时导致请求都去数据库层，引起数据库压力增大。 | 1. 热点数据不过期；2. 接口限流 | 
  | 缓存雪崩 | 缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。和缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。  | 1. 缓存数据过期时间随机避免同时过期；2. 热点数据不过期 | 
  | 缓存污染 | 缓存的数据只被用了几次就不用了，浪费资源  | 1. 数据设置过期时间；2. 合理的数据淘汰策略 | 
  | 数据一致 | 数据库和缓存更新，就容易出现缓存(Redis)和数据库（MySQL）间的数据一致性问题  | 1. 读的时候缓存没有就读数据库，读到数据存入缓存并返回响应；2. 更新数据先更新数据库然后让缓存失效。如果先让缓存失效可能导致一个线程让缓存失效了，还没更新完数据库，另一个查询又把脏数据缓存了。 | 
## 5. kafka
  + 生产者：
    + 常用参数：  
    
        | 参数 | 简介 | 
        | ---- | ---- | 
        | bootstrap.servers | broker地址  |
        | acks | 多少个副本接收到消息服务端才会向生产者发送响应 |
        | buffer.memory | 生产者的内存缓冲区大小。如果生产者发送消息的速度 >消息发送到kafka的速度，那么消息就会在缓冲区堆积，导致缓冲区不足。这个时候，send()方法要么阻塞，要么抛出异常。 |
        | max.block.ms | 表示send()方法在抛出异常之前可以阻塞多久的时间，默认是60s |
        | retries | 发送失败后重试次数 | 
        | retry.backoff.ms | 重试之间时间间隔 | 
        | batch.size | 存多少数据一起发 | 
        | max.in.flight.requests.per.connection | 一个消息发送到收到响应之间，还能发多少个 | 
        | request.timeout.ms | 生产者在发送消息之后，到收到服务端响应时，等待的时间限制 | 
    + 分区策略：配置参数值
         1. 指定分区
         2. 指定对应的key值
         3. 都没有指定，会随机选区分区，之后数据轮询发送。
    + 数据一致性：一个分区有多个副本，其中有一个leader和多个follower。发送者通过ack（确认已收到）来保持发送数据一致问题。  
    ack有一个应答机制：  
    0 ：生产者只管发送数据 不接受ack，会造成数据丢失 因为我都不知道这个leader的死活 有可能在网络连接的时候就挂掉了。  
    1 ：生产者发送数据leader写入成功后，返回ack，这个里面还是会丢数据，如果这个时候leader挂了，他的follower没有同步完数据然后提升为leader之前未同步数据就会丢失。  
    -1：(ALL) 生产者发送数据到leader写入成功后再 等follower也写入成功后返回ack给生产者，这个里面有一个问题就是数据重复，什么样的时候会造成数据重复呢就是follower写入成功后刚要回传ack leader就挂了，这个时候生产者认为数据并没有发送成功，kafka这面会把follower晋升为leader 这个时候的leader已经同步完了但是生产者不知道，又重新发送一个一遍数据，就重复了。
    + 故障处理：  
    A 为leader 数据为1-10  
    B 为follower 已同步数据为 1-6  
    C 为follower 已同步数据为 1-8  
    假如leader挂掉了，B选举为leader了 c再去同步发现数据不一样一个是1-8 一个 是1-6 假如这个时候A恢复了 为1-10，这个时候他们的消息不等 消费者消费消息就会出现问题，kefka是用两个标识来处理的一个是HW（在所有消息队列里面最短的一个LEO）为HW，LEO（log end office）最后一个消息的偏移量，HW在选举为B的时候 他为1-6 他会发送命令 所有人给我截取到6 其余的数据都扔掉，消费者来消费消息也只会消费到HW所在对应的那一个偏移量，假如这个时候选取的是C同步数据为1-8，他的HW为6的数据上面 会发送指定 都给我截取到6 卡卡卡都截取了，再给我A和B再去询问是否有新数据 发现有 7-8 就会同步上去
    
  + 消费者
    + 常用参数：  
    
        | 参数 | 简介 | 
        | ---- | ---- | 
        | fetch.min.bytes | consumer一次拉取中拉取的最小数据量，默认值为1B  |
        | fetch.max.bytes | 一次最大拉的数据量默认50mb |
        | fetch.max.wait.ms | 等待时间，默认值为500ms，如果消息不够多满足不了最小的拉取量，则等待该时间 |
        | max.partition.fetch.bytes | 配置从每个分区返回给消费者最大数据量 |
        | max.poll.records | 最大拉去数据条数默认500 | 
        | connections.max.idle.ms | 空链接超时限制 | 
        | reconnect.backoff.ms | 尝试连接主机之间的等待时间 | 
        | isolation.level | 事务隔离级别，默认读未提交，可以消费到hw | 
    + 消费者和消费者组：一个topic的消息分为几个分区，每个消费者组之间消费topic是互不干扰，都是完整的数据，比如A组消费到10，B组可以从0开始消费；消费者组可以有多个消费者，多个消费者共同消费一个完整的topic消息，每个消费者是一个线程，连接一个分区进行消费，消费者多于分区，则多的消费者是没用的。
  + 数据存储机制  
  同一个topic有多个不同的分区，每个分区是一个目录，文件夹是topic名_分区序号，从0开始；  
  每个分区目录中文件被平均分割为大小相等（默认500mb）的数据文件  
  每个数据文件被称为一个段（segment），每段消息数量不一定相等，主要是方便旧数据清除，默认7天清除旧的。  
  Segment文件命名的规则：partition全局的第一个segment从0（20个0）开始，后续的每一个segment文件名是上一个segment文件中最后一条消息的offset值。
  段文件包含索引（*.index）和数据文件（*.log），索引文件保存消息在数据文件中编号（第几条），编号不一定连续，因为没必要为每条数据都建立索引，隔一定量数据建立避免索引太大。
  如：*.index：1，0 表在数据文件中从上倒下第一条消息，0表示消息的物理偏移地址（前面所有消息总长度）。