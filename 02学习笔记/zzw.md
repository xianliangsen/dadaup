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
    
    
### 1.6 SQL语句
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
+ 各种查询关键字区别
+ 倒排索引
## 4. redis
## 5. kafka