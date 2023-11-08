# SQL优化技巧及原理:see_no_evil:

<a name="3060-1621846615933"></a>目录

[一.sql优化原则：	2](#_toc23358)

[explain	3](#_toc22680)

[适合建立索引的情景	3](#_toc23034)

[不建或者少建索引	3](#_toc1268)

[对千万级Mysql 数据库建立索引的事项 及 提高性能的 手段	4](#_toc25688)

[二.常用sql技巧	6](#_toc7208)

[2.1.窗口函数语法	6](#_toc19953)

[2.2 连续时间问题	6](#_toc23387)

[tips	6](#_toc32637)

[1.执行顺序	6](#_toc20455)

[2.子查询位置	7](#_toc616)

[3.简洁多表连接	7](#_toc12570)

[4.union与union all	7](#_toc32272)

[5.repalec into 与 insert into	7](#_toc18686)

[6. truncate 与 delete alter	7](#_toc23539)

[7.count(email)= count(*) email is not null	8](#_toc27721)

[MySQL	8](#_toc12889)

[DCL	8](#_toc17125)

[-用户管理	8](#_toc20795)

[-权限控制	9](#_toc28086)

[事务	10](#_toc27514)

[-事务操作	10](#_toc15236)

[1.  手动事务操作-开启提交回滚	10](#_toc26265)

[2.  自动事务	10](#_toc28205)

[-ACID	10](#_toc28068)

[-并发事务问题	11](#_toc23192)

[-隔离级别	11](#_toc2020)

[存储引擎	12](#_toc16631)

[-存储引擎特点	13](#_toc14986)

[-存储引擎选择	15](#_toc26713)

[索引	15](#_toc13447)

[-索引结构	15](#_toc8097)

[B树	16](#_toc15894)

[B+数	16](#_toc5978)

[Hash	17](#_toc12758)

[-索引分类	17](#_toc25343)

[-索引创建	18](#_toc13147)

[-SQL性能分析	19](#_toc9648)

[-索引使用规则	20](#_toc14242)

[视图	24](#_toc19823)

[锁	25](#_toc21964)

[-全局锁	25](#_toc6486)

[-表级锁	26](#_toc19210)

[-行级锁：基于索引项	28](#_toc19042)

[InnoDB引擎架构	30](#_toc12971)

[-逻辑存储结构	30](#_toc7120)

[-内存架构(内存+磁盘)	30](#_toc15253)

[- 磁盘结构	32](#_toc18201)

[-后台线程（内存磁盘交互）	33](#_toc11972)

[InnoDB事务原理	34](#_toc14607)

[-redo log -持久性>WAL先写日志刷磁盘	34](#_toc30334)

[-undo log -原子性-回滚-MVCC	35](#_toc2221)

[MVCC-快照读历史版本	35](#_toc5210)

[-基本概念	35](#_toc1429)

[-隐藏字段	36](#_toc9295)

[-undolog版本链	36](#_toc9042)

[-readview读视图	37](#_toc15714)

[-原理分析(RC级别)	38](#_toc28633)

[-原理分析(RR级别)	38](#_toc17585)

[管理	38](#_toc30428)

[-系统数据库-自带库	38](#_toc11759)

[-常用工具1-客户端工具	39](#_toc16621)

[-常用工具2	40](#_toc15491)

[日志	41](#_toc21635)

[-错误日志-启动停止错误排查	41](#_toc13038)

[-二进制日志-数据恢复(记录DDL+DML)	42](#_toc4190)

[-查询日志(二进制无查询)	43](#_toc2601)

[-慢查询日志(默认不记录非索引查询)	43](#_toc12771)


# <a name="vc0n-1647601801878"></a><a name="_toc23358"></a>**一.sql优化原则：**
<a name="9291-1595213622686"></a>1.与数据库交互是比较重的操作，尽可能一次查询获取所有结果，不行就单次查出所有关联结果，用map关联。

<a name="8081-1595213727499"></a>2.查询条件的质量：比如inner jo in 会比 left join 性能好很多，尽可能根据逻辑使用好join

<a name="4480-1595213840419"></a>3.结果集大小。数据量大的时候，尽可能精简结果集大小。

<a name="4324-1595852444880"></a><a name="atwv-1616513147299"></a>=================SQL语言艺术====================

<a name="1yxx-1620285567295"></a>IO   查询数量、查询次数

<a name="exzz-1620285570717"></a>数据结构  B+树

<a name="1912-1595249961796"></a>总结：影响性能的重要因素

<a name="6255-1595249984508"></a>表的数据量（300万级别性能开始下降）

<a name="2130-1595249989316"></a>表有哪些索引 （子查询没有索引）提高查找效率的排序的数据结构（bTree）b数，3层b数 百万级别只用3次 IO访问磁盘，全表扫描则需要百万次io

<a name="2720-1595249997947"></a>存储特性（例如分区），和索引同样重要

<a name="9812-1595250024727"></a>查询条件的质量

<a name="2711-1595250030204"></a>结果集的大小
## <a name="4162-1587888154083"></a><a name="7993-1595249948969"></a><a name="_toc22680"></a>**explain** 
<a name="4027-1596112331386"></a>     # type  效率由高到低   system(相当于系统表)>const(常量)>eq\_ref( 只有一条记录与之匹配--ceo)>ref(多条记录与之匹配)>range(in、between、<> 非全表扫描)>index(索引扫描)>all(全表扫描) 

<a name="5049-1596112963788"></a>百万级别一般需要all优化到range以上

<a name="8429-1596113008509"></a>possible key---可能用到的索引（仅推测）

<a name="2520-1596113226001"></a>key --实际使用的索引 （可能索引失效）

<a name="p69r-1698087433975"></a>key\_len 索引长度（不失精度下越短越好）

<a name="8555-1597061912200"></a>ref -- 可能使用到索引列当中的值

<a name="5587-1597061945213"></a>#rows -- 需要检索的列（越少越好） ，加索引就变少。（已排序的B+树）

<a name="ghsa-1698087508571"></a>filtered 读取行数百分比，值越大越好

<a name="5792-1597062007399"></a>#extra -- using filesort 没有使用索引中的列(复合索引，多字段)排序 ，(慢)            ？复合索引最左原则

<a name="3758-1597062093776"></a>         -- using temp 使用临时表 ，非常慢 （解决和上面一样，如有group by）

<a name="0045-1597062449405"></a>         -- using index   发财

<a name="lhrv-1698262052975"></a>-1.using index condition : 使用了索引，但需要回表查询;

<a name="fv4c-1698262084857"></a>-2.using where:using index : 使用了索引，但需要的数据都在索引列中能找到，所以不需要回表查询,意味比1快；

<a name="xj35-1626320498319"></a><a name="uysj-1648624140014"></a>B树节点和叶子节点存储的是key value

<a name="fyid-1626331941842"></a>innodb 聚集(聚簇)索引  叶子节点包含 索引key 和 完整的数据，节点存放的是key

<a name="3inb-1626332059793"></a>非聚集索引/二级索引/辅助索引    叶子节点包含 索引key 和 数据地址 （优点：status占用空间小，保证数据一致性）

<a name="x22t-1642563633561"></a>非聚簇索引先查一次b+tree查出key ,在用key去查聚簇索引的 b+tree, 这个过程叫“回表";

## <a name="s4vk-1626331942008"></a><a name="a8li-1647601801879"></a><a name="_toc23034"></a>**适合建立索引的情景**
<a name="bx9r-1626331451957"></a>1. **表的主关键字**  自动建立唯一索引

<a name="08gq-1626331522394"></a>2. **表的字段唯一约束**  （联合索引）

<a name="mgzg-1626331583644"></a>3. **直接条件查询的字段**   where 后面的条件

<a name="l2bl-1626331622173"></a>4. **查询中与其它表关联的字段**  外键

<a name="jknl-1626331656495"></a>5. **查询中排序的字段**  排序的字段如果通过索引去访问那将大大提高排序速度

<a name="bwy1-1626331690457"></a>6. **查询中统计或分组统计的字段**  count() 、group by

## <a name="cr9t-1626331787638"></a><a name="tzpf-1647601801880"></a><a name="_toc1268"></a>**不建或者少建索引**
<a name="mjkv-1626331886984"></a>1.表记录太少

<a name="4s9u-1626331909730"></a>2.经常插入、删除、修改的表 （相对索引的开销）

<a name="dk9c-1626332543234"></a>3.数据重复且分布平均的表字段   例如 0 or 1 true or false

## <a name="depu-1626332833278"></a><a name="ib4d-1647601801881"></a><a name="_toc25688"></a>**对千万级Mysql 数据库建立索引的事项 及 提高性能的 手段**
<a name="ss38-1626331309779"></a>一、注意事项：

<a name="tlaf-1626332976484"></a>    建立索引的时候也会占用大量表空间， 首先考虑空间容量问题。

<a name="stzi-1626333013845"></a>    其次，在对建立索引的时候要对表进行加锁，应在业务空闲的时候进行。

<a name="vhtw-1626333051096"></a>二、建立索引优化需要注意的问题：

<a name="sas4-1626333333510"></a>  1.创建索引，不加索引哪怕查找一条特定的数据都会进行全表扫描。引起致命的性能下降。

<a name="wvib-1626333419103"></a>   2.复合索引  mysql查询每次只能使用一个索引。（area、age、salary）的复合索引，其实相当创建了（area、age、salary)、（area、age）、（area）三个索引，也称为 最佳左前缀特性。因此我们在创建 复合索引时应该将最常用作限制条件的放最左边，依次递减。

<a name="mwwm-1626333658633"></a>  3.  索引不会包含有NULL值的列。只要列中包含NULL值都将不会被包含在索引中，复合索引中只要有一列含有NULL值，那么这一列对于此复合索引都是无效的，所以我们在设计数据库时不要让字段的默认值为NULL

<a name="3lqk-1626338252309"></a>  4. 使用短索引  对串列进行索引，在前10个或 20个字符内进行索引

<a name="9jrk-1626338313382"></a>  5. 排序索引的问题。mysql查询只使用一个索引，因此如果where子句中已经使用了索引的话，那么order by中的列是不会使用索引的。因此数据库默认排序可以符合要求的情况下不要使用排序索引；尽量不要包含多个列的排序，如果需要最好给这些列创建复合索引

<a name="val6-1626338517759"></a>  6. like 语句操作。一般情况下不鼓励使用like 操作，如果非使用不可，如何使用也是一个问题， like "%aaa%" 不会使用索引。而"aaa%"可以使用索引。左模糊查询不会使用索引

<a name="kwff-1626340575592"></a>  7. 不要在列上进行运算 select \* from user where YEAR(adddate);

<a name="oc6z-1626342289547"></a>  8. 不使用NOT IN 操作。 not in操作都不会使用索引将进行全表扫描。

<a name="ncgn-1626320519358"></a><a name="xmfq-1631180283612"></a>with as 关键字 片段

<a name="3jhc-1647601801882"></a><a name="2jld-1647601801883"></a>**执行一条sql语句的过程**

<a name="4nqi-1621693845080"></a>池化思想、消耗资源的链接创建一个连接池来维护

<a name="ctz1-1647776806706"></a>局部性原理  将数据相邻的数据也会读到内存当中。

<a name="gvpa-1621694106869"></a>mysql由c++编写的一个应用

<a name="8c1e-1621694087191"></a>消耗资源的操作：

<a name="th4v-1621694191553"></a>        I/O ：数据库连接 （线程的创建与销毁） 、磁盘读写（磁盘读写慢，）

<a name="emp6-1621694326676"></a> 解决方案：数据库连接池维护一定数量的线程、磁盘预读写，缓存在innnoDB buffer pool内存 中

<a name="1evs-1621694476090"></a>1.通过JDBC驱动连接到 mysql ,调用 mysql应用的接口，sql语句经过分析器词法分析（AST抽象语法树），成mysql执行语句

<a name="va5u-1621694591081"></a>2.innoDB数据库存储引擎，执行器分析出优化的方案执行。

<a name="yrby-1621694652054"></a>3.先从buffer  pool中获取数据，没有就从磁盘中读取，然后将数据缓存到buffer中。类似redis缓存。同时iinnodb级别的undo.log记录操作前数据（事务回滚），redo.log 记录操作后数据（断电恢复）。数据库级别bin.log记录所有操作（主从同步）。

<a name="9t7e-1621695058134"></a>事务提交（两阶段提交），

<a name="zvzv-1621695656987"></a>将 redo log buffer 数据刷入到 redo log.file (WAL write ahead log)文件中，（断电可恢复，数据一致性）

<a name="zyvq-1621695651711"></a>       将本次操作记录写到bin.log文件中，

<a name="uguj-1621695692818"></a>将bin。log文件名字和更新内容在bin log 中的位置(LSN)记录到 redo log中，同时在 redo log最后添加commit标记 

<a name="z1rz-1621695598617"></a>4.buffer定期刷磁盘同步数据（更新操作默认立即刷入）中间还有一层OS cache.如果数据还在OS cache未刷入磁盘时断电，数据也会丢失

<a name="t0ug-1648713164223"></a><a name="enmw-1648713164378"></a>**2.采用二阶段提交的原因**

<a name="zqjy-1648713174043"></a>（只单独写 redolog（prepare,commit） 或者binlog（有无记录）,会存在问题，只有两个执行完后，其中一个再做commit标记，才能标记 回滚，继续提交，以及已完成提交3种状态 ）

<a name="xu9d-1648713164854"></a>先写redolog再写binlog

<a name="z1ux-1648713164856"></a>如果在一条语句redolog之后崩溃了，binlog则没有记录这条语句。系统在crash recovery时重新执行了一遍binlog便会少了这一次的修改。恢复的数据库少了这条更新。

<a name="gn9c-1648713164858"></a>先写binlog再写redolog

<a name="3ijm-1648713164860"></a>如果在一条语句binlog之后崩溃了，redolog则没有记录这条语句（数据库物理层面并没有执行这条语句）。系统在crash recovery时重新执行了一遍binlog便会多了这一次的修改。恢复的数据库便多了这条更新。

<a name="xfjo-1648713501496"></a>redo和binlog这两种日志有以下三点不同：

<a name="dspi-1648713502180"></a>1、 redo log 是 InnoDB 引擎特有的；binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。

<a name="nwhn-1648713502183"></a>2、 redo log 是物理日志，记录的是“在某个数据页上做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。

<a name="mxjg-1648713502185"></a>3、redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。

<a name="hfbv-1648713502187"></a><a name="y8wk-1648713393914"></a>**Binlog**是数据库级别**，真正确定是否有该条记录，**而**redolog**是**Innodb**用来做**内存数据操作的(快速恢复)，**crud优先记载再内存中，后刷磁盘，避免大量磁盘IO,耗性能

<a name="f9ot-1621694746777"></a>**Crash recovery**

<a name="odza-1648713141758"></a>在做Crash recovery时，分为以下3种情况：

<a name="h1qp-1648713141760"></a>binlog有记录，redolog状态commit：正常完成的事务，不需要恢复；

<a name="jpsc-1648713141763"></a>binlog有记录，redolog状态prepare：在binlog写完提交事务之前的crash，恢复操作：提交事务。（因为之前没有提交）

<a name="eoe6-1648713141765"></a>binlog无记录，redolog状态prepare：在binlog写完之前的crash，恢复操作：回滚事务（因为crash时并没有成功写入数据库）


## <a name="riff-1648713141767"></a><a name="6994-1595249949003"></a><a name="hekl-1647601801884"></a><a name="_toc7208"></a>**二.常用sql技巧**
### <a name="i9j9-1647601801885"></a><a name="_toc19953"></a>**2.1.窗口函数语法**
<窗口函数> over (partition by <用于分组的列名>

<a name="6qo6-1610438422238"></a>                order by <用于排序的列名>)

<a name="hztt-1610438422238"></a><窗口函数>的位置，可以放以下两种函数：

<a name="v6uf-1610438422238"></a><a name="u52d-1610438422238"></a>1） 专用窗口函数，比如rank, dense\_rank, row\_number等

<a name="sua3-1610438422239"></a>2） 聚合函数，如sum. avg, count, max, min等

<a name="ymz1-1610438422239"></a><a name="onjw-1610438422239"></a>**2.窗口函数有以下功能：**

<a name="hc71-1610438422239"></a>1）同时具有分组（partition by）和排序（order by）的功能

<a name="dwue-1610438422239"></a>2）不减少原表的行数，所以经常用来在每组内排名

<a name="c6rg-1610438422239"></a>**3.注意事项**

<a name="8jca-1610438422239"></a>窗口函数原则上只能写在select子句中

<a name="wfxw-1610438422239"></a>**4.窗口函数使用场景**

<a name="kh6h-1610438422239"></a>1）业务需求“**在每组内排名”**，比如：

<a name="p8ph-1610438422239"></a>排名问题：每个部门按业绩来排名topN问题：找出每个部门排名前N的员工进行奖励

### <a name="7083-1595249949111"></a><a name="4n1t-1647601801886"></a><a name="_toc23387"></a>**2.2 连续时间问题**

# <a name="koke-1696863905445"></a><a name="iim8-1696863845970"></a><a name="_toc32637"></a>**tips**
## <a name="fv2q-1698086159246"></a><a name="_toc20455"></a>**1.执行顺序**
1\.执行顺序 ：from > on > where > group by > having > select > distinct > order by > top

但是，MY SQL 中HAVING是可以使用SELECT中定义的别名。HAVING、GROUP BY、ORDER BY都可以使用SELECT中定义的别名，

这在MY SQL使用文档中有表述“A select\_expr can be given an alias using AS alias\_name. The alias is used as the expression's

` `column name and can be used in GROUP BY, ORDER BY, or HAVING clauses.It is not permissible to refer to a column alias

`  `in a WHERE clause, because the column value might not yet be determined when the WHERE clause is executed. 

`  `See Section B.5.4.4, “Problems with Column Aliases”



`  `在执行 SELECT 子句时，系统先计算 SELECT 中的列表达式和函数等，然后为这些列赋予一个列别名，

`  `并生成一个虚拟的查询结果表。接着，系统按照 GROUP BY 子句中的指定字段分组，并对每个分组进行计算，生成虚拟的分组结果表。

`  `最后，HAVING 子句基于这个虚拟的分组结果表进行筛选操作。

由于在执行 HAVING 子句时已经生成了虚拟的查询结果表和虚拟的分组结果表，因此 HAVING 子句可以直接引用 SELECT 列别名，

<a name="ghgu-1696863854941"></a>而不需要再次计算或声明
## <a name="omve-1698086204572"></a><a name="_toc616"></a>**2.子查询位置**
<a name="vdum-1698086204571"></a>2. 有在select做子查询，也有在from做子查询   
## <a name="ym2d-1696735265316"></a><a name="_toc12570"></a>**3.简洁多表连接**
<a name="fgy5-1698086273074"></a>3.连接用  USING() 简洁一点  
## <a name="6dnv-1698086245009"></a><a name="_toc32272"></a>**4.union与union all**
<a name="bbz9-1698086354506"></a>4. union 在前面的列名会被保留。 union all 不去重 
## <a name="j1na-1698086318830"></a><a name="_toc18686"></a>**5.repalec into 与 insert into**
5\.关键字NULL可以用DEFAULT替代。

掌握replace into···values的用法

replace into 跟 insert into功能类似，不同点在于：replace into 首先尝试插入数据到表中，

如果发现表中已经有此行数据（根据主键或者唯一索引判断）则先删除此行数据，然后插入新的数据；

否则，直接插入新数据。

<a name="dg87-1698086430406"></a>要注意的是：插入数据的表必须有主键或者是唯一索引！否则的话，replace into 会直接插入数据，这将导致表中出现重复的数据。
## <a name="qhl3-1698086420800"></a><a name="_toc23539"></a>**6. truncate 与 delete alter**
细节剖析：

删除exam\_record表中所有记录；

并重置自增主键;

思路实现：本题采用第二种删除方式，满足条件1或条件2就删除，但只删除3条记录:

TRUNCATE exam\_record; --全部删除（表清空，包含自增计数器重置）

也可采用第一种，不过需要手动重置自增ID，不过效率角度考虑，还是第二种方式效率更高：

DELETE FROM exam\_record;

<a name="jkiv-1698086497338"></a>ALTER TABLE exam\_record auto\_increment=1;
## <a name="drx4-1698086420953"></a><a name="_toc27721"></a>**7.count(email)= count(\*) email is not null**
聚合函数里面还能写条件

select (select count(1) from exam\_record) total\_pv, 

(select count(1) from exam\_record where score is not null) complete\_pv ,(select count(distinct(exam\_id)) from exam\_record where score is not null ) complete\_exam\_cnt;

#优美

select count(\*) as total\_pv,

count(score) as complete\_pv,

count(distinct exam\_id,score IS NOT NULL or null) as complete\_exam\_cnt

<a name="x8jn-1698355930626"></a>from exam\_record

# <a name="sgmf-1698086363498"></a><a name="cgnw-1698086318976"></a><a name="_toc12889"></a>**MySQL**
## <a name="x1t2-1696735285622"></a><a name="_toc17125"></a>**DCL**
### <a name="6q0c-1696736466072"></a><a name="_toc20795"></a>**-用户管理**
#### <a name="uj4y-1698934973382"></a>**1．查询用户**
#### <a name="wspi-1698934977505"></a>**2．创建用户**
#### <a name="7nyp-1698934982483"></a>**3．修改用户密码**
#### <a name="um15-1698934986587"></a>**4．删除用户**
1．查询用户

USE mysql;

SELECT \* FROM user;

2．创建用户


CREATE USER ‘用户名'@'主机名’IDENTIFIED BY‘密码';


3．修改用户密码


ALTER USER ‘用户名'@'主机名’IDENTIFIED WITH mysql\_native\_password BY‘新密码;




4．删除用户

DROP USER‘用户名'@'主机名';

注意:

.主机名可以使用%通配。

<a name="bwhz-1696735302829"></a>·这类SQL开发人员操作的比较少，主要是DBA （Database Administrator 数据库管理员）使用。
### <a name="3nyz-1625216658971"></a><a name="_toc28086"></a>**-权限控制**
#### <a name="jbks-1698934999767"></a>**1．查询权限**
#### <a name="diuh-1698935003460"></a>**2.   授予权限**
#### <a name="ylog-1698935006832"></a>**3．撤销权限**
1．查询权限

SHOW GRANTS FOR‘用户名'@'主机名';

2\.授予权限

GRANT权限列表ON数据库名.表名TO‘'用户名'@'主机名';

3．撤销权限

REVOKE 权限列表ON数据库名.表名FROM‘用户名'@'主机名';

注意:

·多个权限之间，使用逗号分隔

<a name="zatv-1696736510154"></a>·授权时，数据库名和表名可以使用\*进行通配，代表所有。

## <a name="5525-1595249949179"></a><a name="sjar-1636598197682"></a><a name="_toc27514"></a>**事务**
### <a name="wri9-1696837069470"></a><a name="_toc15236"></a>**-事务操作**
### <a name="acec-1698935098479"></a><a name="_toc26265"></a>**1.  手动事务操作-开启提交回滚**
### <a name="46eo-1698935204248"></a><a name="_toc28205"></a>**2.  自动事务**
Transaction：一组操作的集合，要么全部成功，要么全部失败。

-开启事务，提交事务（抛异常，回滚事务）

-事务操作

·开启事务    START TRANSACTION或 BEGIN;

·提交事务    COMMIT;

·回滚事务    ROLLBACK ;

手动事务： 灵活和可控的事务管理方式

<a name="krl9-1696758438821"></a>自动事务： 减少开发人员的工作量
### <a name="vqlu-1696758438822"></a><a name="_toc28068"></a>**-ACID**
#### <a name="x6u4-1698935230037"></a>**1.  原子性**
#### <a name="b0em-1698935233295"></a>**2.  一致性**
#### <a name="zspy-1698935235755"></a>**3.  隔离性**
#### <a name="zyul-1698935238660"></a>**4.  持久性**
·原子性(Atomicity):事务是不可分割的最小操作单元，要么全部成功，要么全部失败。

.一致性（Consistency):事务完成时，必须使所有的数据都保持一致状态。

·隔离性（Ilsolation)︰数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下运行。

<a name="hdkj-1696837126784"></a>·持久性（Durability):事务一旦提交或回滚，它对数据库中的数据的改变就是永久的。
### <a name="nokq-1696837105831"></a><a name="_toc23192"></a>**-并发事务问题**
#### <a name="mrdk-1698935252808"></a>**1.  脏读**
#### <a name="ewhq-1698935269469"></a>**2.  不可重复读**
#### <a name="aujz-1698935274920"></a>**3.  幻读**
.脏读（A可读B未提交）:一个事务读到另外一个事务还没有提交的数据。(读未提交-事务操作 未提交却允许读取)--有变没有（B事务回滚）

.不可重复读（A读B未提交和已提交不一致）:一个事务先后读取同一条记录，但两次读取的数据不同，称之为不可重复读。

.幻读（A事务未提交，可重复读B已提交新增，A新增失败）  --没有变有 (读的历史版本)

<a name="pyv7-1696837399325"></a>一个事务按照条件查询数据时，没有对应的数据行，但是在插入数据时，又发现这行数据已经存在，好像出现了"幻影”。
### <a name="yhxf-1696837399326"></a><a name="_toc2020"></a>**-隔离级别**
#### <a name="omjq-1698935288286"></a>**1.  读未提交**
#### <a name="lssb-1698935297866"></a>**2.  读已提交**
#### <a name="kwca-1698935302839"></a>**3.  可重复读**
#### <a name="tmef-1698935310086"></a>**4.  串行化**
隔离级别                赃读    不可重复读    幻读

Read uncommitted        √(会出现)  √           √

Read committed                   √           √

Repeatable Read(默认)                        √

Serializable

双终端模拟两个事务（手动开启事务，事务没提交时的中间态情况分析）

读未提交-事务操作 A(开事务)事务可读B未提交事务  未提交却允许读取 （事务中间态-类似缓存 ）

读已提交-A(开事务)事务读取B事务未提交和提交后 数据不一致； 但会不可重复读

可重复读-即读事务开启前的数据，A（未提交）读B commit (未提交和提交后都)是未修改前数据; A commit后才会读B commit的数据； 但会幻读

串行化-顺序执行B需要在A-commit后才能update数据; 

一个事务按照条件查询数据时，没有对应的数据行，但是在插入数据时，又发现这行数据已经存在，好像出现了"幻影”。

--B先insert7-commit,Aselect7出null,insert7却失败（告知已有数据）

可重复读并发事务时一般用间隙锁（仅使用RR）解决幻读；--注意：但是需要基于索引项，非索引列就不会有间隙锁，升级为表锁

--A先加间隙锁insert id=5(无该行记录)，锁3~8间隙, Binsert7阻塞，避免A幻读。

--查看事务隔离级别

SELECT @@TRANSACTION\_ISOLATION;

--设置事务隔离级别

SET [ SESSION| GLOBAL ] TRANSACTON SOLATON LEVEL {READUNCOMMTTED | READCOMMITED │ REPEATABLEREAD |SERIALIZABLE}

<a name="zrrk-1696838153952"></a>注意:事务隔离级别越高，数据越安全，但是性能越低。
## <a name="bht1-1696838153953"></a><a name="_toc16631"></a>**存储引擎**
#### <a name="femh-1698935487485"></a>**1.  在创建表时，指定存储引擎**
#### <a name="tqfb-1698935493528"></a>**2.  查看当前数据库支持的存储引擎**
存储引擎简介-属于表级别

1\.在创建表时，指定存储引擎

CREATE TABLE表名(

`    `字段1字段1类型[ COMMENT字段1注释]，

`    `字段n字段n类型[COMMENT字段n注释]

`  `)ENGINE = INNODB [ COMMENT 表注释];

2\.查看当前数据库支持的存储引擎

<a name="gbam-1696848529134"></a>SHOW ENGINES;
### <a name="oglk-1696848529135"></a><a name="_toc14986"></a>**-存储引擎特点**
#### <a name="6fp3-1696848857758"></a>**InnoDB**
##### <a name="ox4a-1698935565602"></a>**1.  介绍**
##### <a name="onm5-1698935566490"></a>**2.  特点**
##### <a name="gazm-1698935567481"></a>**3.  文件**
##### <a name="8h9j-1698935571871"></a>**4.  逻辑存储结构**
\>介绍

InnoDB是一种兼顾高可靠性和高性能的通用存储引擎，在MySQL5.5之后，InnoDB是默认的MySQL存储引擎。

\>特点

DML操作遵循ACID模型，支持事务;行级锁，提高并发访问性能;

支持外键FOREIGN KEY约束，保证数据的完整性和正确性;

\>文件

xx.ibd:xo代表的是表名，innoDB引擎的每张表都会对应这样一个表空间文件，存储该表的表结构（frm, esdi）)、数据和索引。

参数: innodb\_file\_per\_table

\>逻辑存储结构

.TableSpece:表空间

.segment:段

.Extent:区    1M  =64页

.Page:页      16K

<a name="jz77-1696848676612"></a>·Row:行
#### <a name="pe25-1696848676613"></a>**MyISAM**
##### <a name="lrec-1698935670082"></a>**1.  介绍**
##### <a name="pbvq-1698935670649"></a>**2.  特点**
##### <a name="yqhc-1698935670650"></a>**3.  文件**
MylSAM>介绍

MylSAM是MySQL早期的默认存储引擎。

\>特点

不支持事务，

不支持外键支持表锁，

不支持行锁访问速度快

\>文件

XXx.sdi:存储表结构信息

XXx.MYD:存储数据心

<a name="kzmo-1696849511270"></a>XXx.MYI:存储索引
#### <a name="secf-1696849511272"></a>**Memory**
##### <a name="nelb-1698935673103"></a>**1.  介绍**
##### <a name="dxmb-1698935673584"></a>**2.  特点**
##### <a name="qgqo-1698935673585"></a>**3.  文件**
Memory

\>介绍

Memory引擎的表数据时存储在内存中的，由于受到硬件问题、或断电问题的影响，只能将这些表作为临时表或缓存使用。

\>特点

内存存放

hash索引（默认)

\>文件

<a name="mfpx-1696849718445"></a>Xxx.sdi:存储表结构信息
### <a name="qiek-1696849718446"></a><a name="_toc26713"></a>**-存储引擎选择**
在选择存储引擎时，应该根据应用系统的特点选择合适的存储引擎。对于复杂的应用系统，还可以根据实际情况选择多种存储引擎进行组合。

\>InnoDB:是Mysql的默认存储引擎，支持事务、外键。如果应用对事务的完整性有比较高的要求，在并发条件下要求数据的一致

性，数据操作除了插入和查询之外，还包含很多的更新、删除操作，那么InnoDB存储引擎是比较合适的选择。

\>MyISAM如果应用是以读操作和插入操作为主，只有很少的更新和删除操作，并且对事务的完整性、并发性要求不是很高，那

么选择这个存储引擎是非常合适的。  MongoDB平替

\>MEMORY:将所有数据保存在内存中，访问速度快，通常用于临时表及缓存。MEMORY的缺陷就是对表的大小有限制，太大的表

<a name="uv2i-1696850044598"></a>无法缓存在内存中，而且无法保障数据的安全性。  Redis平替
## <a name="jrcz-1696850044599"></a><a name="_toc13447"></a>**索引**
#### <a name="z7mq-1698935695310"></a>**1.  介绍**
#### <a name="vjro-1698935699363"></a>**2.  优缺点**
\>介绍

`  `高效查找的数据结构（有序）

\>优缺点

优势:

提高数据检索的效率，降低数据库的IO成本

通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗

劣势：

索引列也是要占用空间的。

<a name="hj5b-1696858679769"></a>索引大大提高了查询效率，同时却也降低更新表的速度，如对表进行INSERT、UPDATE、DELETE时，效率降低。
### <a name="p6r3-1696858679770"></a><a name="_toc8097"></a>**-索引结构**
MySQL的索引是在存储引擎层实现的，不同的存储引擎有不同的结构，主要包含以下几种:

B+Tree索引：最常见的索引类型，大部分引擎都支持B+树索引

Hash索引：底层数据结构是用哈希表实现的,只有精确匹配索引列的查询才有效,不支持范围查询

R-tree(空间索引)：空间索引是MylSAM引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少

<a name="lzy4-1696860843490"></a>Full-text(全文索引)：是一种通过建立倒排索引,快速匹配文档的方式。类似于Lucene,Solr,ES
### <a name="jfeb-1696860843491"></a><a name="_toc15894"></a>**B树**
### <a name="9rrd-1697524570983"></a><a name="_toc5978"></a>**B+数**
#### <a name="4tqk-1698935751853"></a>**1.  与B数区别**
#### <a name="pzp7-1698935768059"></a>**2.  InnoDB为什么选择B+树**
相对于B-Tree区别:

①.所有的数据都会出现在叶子节点 

②.叶子节点形成一个单向链表

MySQL索引数据结构对经典的B+Tree进行了优化。在原B+Tree的基础上，增加一个指向相邻叶子节点的链表指针（双向链表），

就形成了带有顺序指针的B+Tree，提高区间访问的性能。这样查“数据块（区间范围）”的时候比B数快很多

为什么InnoDB存储引擎选择使用B+tree索引结构?

\>相对于二叉树，层级更少，搜索效率高;

对于B-tree，无论是叶子节点还是非叶子节点，,都会保存数据，这样导致一

页（16KB）中存储的键值减少，指针跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低;

<a name="guzj-1697524584074"></a>4阶/度/叉 最多存3个数
### <a name="snxg-1697524584075"></a><a name="_toc12758"></a>**Hash**
#### <a name="lu1h-1698935823436"></a>**1.  特点**
#### <a name="peli-1698935826938"></a>**2.  存储引擎支持**
#### <a name="xhlq-1698935848943"></a>**3.  哈希碰撞问题解决-HashMap结构**
\>Hash索引特点

1\. Hash索引只能用于对等比较(=，in)，不支持范围查询(between，>，<，..)2.无法利用索引完成排序操作

3\.查询效率高，通常只需要一次检索就可以了，效率通常要高于B+tree索引

\>存储引擎支持

在MySQL中，支持hash索引的是Memory引擎，而innoDB中具有自适应hash功能，hash索引是存储引擎根据B+Tree索引在指定条件下自动构建的。

哈希碰撞解决

<a name="bb7q-1697525798383"></a>HashMap hash槽后面增加一个链表 -详见数据结构
### <a name="76bw-1697525778738"></a><a name="_toc25343"></a>**-索引分类**
#### <a name="fbur-1698935901887"></a>**1.  主键索引**
#### <a name="nzzv-1698935906644"></a>**2.  唯一索引**
#### <a name="upau-1698935909493"></a>**3.  常规索引**
#### <a name="rjnv-1698935919240"></a>**4.  全文索引**
#### <a name="rmvu-1698935983754"></a>**5.  聚集与非聚集**
分类        含义        特点        关键字

主键索引    针对于表中主键创建的索引    默认自动创建,只能有一个    PRIMARY

唯一索引    避免同一个表中某数据列中的值重复    可以有多个    UNIQUE

常规索引    快速定位特定数据    可以有多个    

全文索引    全文索引查找的是文本中的关键词，而不是比较索引中的值    可以有多个    FULLTEXT

聚集索引(Clustered Index)  将数据存储与索引放到了一块，索引结构的叶子节点保存了行数据  必须有,而且只有一个(也只需有一个)

非聚集索引/二级索引(Secondary lndex)  将数据与索引分开存储，索引结构的叶子节点关联的是对应的主键  可以存在多个公

聚集索引选取规则:

\>如果存在主键，主键索引就是聚集索引。

如果不存在主键，将使用第一个唯一(UNIQUE）索引作为聚集索引。

如果表没有主键，或没有合适的唯一索引，则InnoDB会自动生成一个rowid作为隐藏的聚集索引。

<a name="kbxn-1697526587316"></a>非主键查找逻 辑(回表)- 通过非聚集索引查找到(叶子结点)主键，再通过聚集(主键)索引来定位到(叶子结点)行数据
#### <a name="t1u1-1697527701468"></a>**-3层B+树主键容量**
两阶3

假设:

一行数据大小为1k，一页中可以存储16行这样的数据。InnoDB的指针占用6

个字节的空间，主键即使为bigint，占用字节数为8。

高度为2:

n\*8+(n + 1)\*6=16\*1024,算出n约为11701171\* 16= 18736

高度为3:

<a name="uqnt-1697527701467"></a>1771\* 1771\*76= 21,939,856
### <a name="ju0c-1697527701470"></a><a name="_toc13147"></a>**-索引创建**
#### <a name="a8zv-1698936024125"></a>**1.  创建create**
#### <a name="k5x4-1698936028474"></a>**2.  查看show**
#### <a name="bqtf-1698936031012"></a>**3.  删除drop**
1\. name字段为姓名字段，该字段的值可能会重复，为该字段创建索引。 

2\. phone手机号字段的值，是非空，且唯一的，为该字段创建唯一索引。

3\. 为profession、age、status创建联合索引。

4．为email建立合适的索引来提升查询效率。

CREATE INDEX idx\_user\_name ON tb\_user(name); 

CREATE UNIQUE INDEX idx\_user\_phone ON tb\_user(phone);

CREATE INDEX idx\_user\_pro\_ale\_sta ON tb\_user(profession,age,status);CREATE INDEX idx\_email ON tb\_user(email);

<a name="iryx-1697531629869"></a>查看 Show , 删除 Drop
### <a name="rsrr-1697532072447"></a><a name="_toc9648"></a>**-SQL性能分析**
#### <a name="9ps1-1697532346408"></a>**1. SQL执行频率**
MySQL客户端连接成功后，通过show[sessionlglobal status 命令可以提供服务器状态信息。通过如下指令，

可以查看当前数据库的INSERT、UPDATE、DELETE、SELECT的访问频次:

SHOW GLOBAL STATUS LIKE 'Com\_';

<a name="fu19-1697532103702"></a>-索引主要优化的时查询效率
#### <a name="epoc-1697532103703"></a>**2.  慢查询日志**
慢查询日志记录了所有执行时间超过指定参数(long\_query\_time，单位:秒，默认10秒〉的所有SQL语句的日志。

MySQL的慢查询日志默认没有开启，需要在MySQL的配置文件(/etc/my.cnf）中配置如下信息:

#开启MySQL慢日志查询开关

slow\_query\_log=1

#设置慢日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，记录慢查询日志long\_query\_time=2

<a name="ejlo-1697532453860"></a>配置完毕之后，通过以下指令重新启动MySQL服务器进行测试，查看慢日志文件中记录的信息/var/lib/mysql/localhost-slow.log.
#### <a name="rtzu-1698086024361"></a>**3.  profile详情-补充未记录到慢查询**
profile详情（概述）-未记录到慢查询日志的sql执行分析补充-

开发仅做了解就行

show profiles能够在做SQL优化时帮助我们了解时间都耗费到哪里去了。通过have\_profiling参数，能够看到当前MySQL是否支持profile操作:

`    `SELECT @@have\_profiling ;

默认profiling是关闭的，可以通过set语句在session/global级别开启profiling:

`    `SET profiling = 1;

执行一系列的业务SQL的操作，然后通过如下指令查看指令的执行耗时:

#查看当前会话中每一条SQL的耗时基本情况

`    `show profiles;

#查看指定query\_id的5QL语句各个阶段的耗时情况

`    `show profile for query query\_id;

#查看指定query\_id的SQL语句CPU的使用情况

<a name="t4id-1697532924762"></a>    show profile cpu for duery query\_id;
#### <a name="fixj-1697532924763"></a>**4.  explain执行计划**
EXPLAIN或者DESC命令获取MySQL如何执行SELECT语句的信息，包括在SELECT语句执行过程中表如何连接和连接的顺序。

语法:

#直接在select语句之前加上关键字explain / desc

`    `EXPLAIN LSELECT字段列表 FROM表名 WHERE条件;

<a name="8bs8-1698086053391"></a>详见置顶的优化
### <a name="hsyt-1698086053392"></a><a name="_toc14242"></a>**-索引使用规则**
#### <a name="wfva-1698258549870"></a>**1.  最左前缀法则**
如果索引了多列（联合索引)，要遵守最左前缀法则。最左前缀法则指的是查询从索引的最左列开始，并且不跳过索引中的列。

如果跳跃某一列，索引将部分失效(后面的字段索引失效)。-类似级联效果

index(profession\_age\_status) -三条索引

explain select \* from tb\_user where profession='软件工程' and age = 31 and status = '0';✔ key\_len=54

explain select \* from tb\_user where profession= '软件工程' and age = 31;✔  --key\_len=49

explain select \* from tb\_user where profession= '软件工程' and status = '0'; ✔✘ profession生效，跳过age,级联status失效

explain select\* from tb\_user where profession= '软件工程';✔

explain select \* from tb\_user where age = 31 and profession='软件工程' and status = '0'; ✔ -无关and顺序,优化器优化执行

explain select \* from tb\_user where age = 31 and status = 'O';✘

<a name="jzos-1698258295801"></a>explain select \* from tb\_user where status = 'O';✘
#### <a name="rhqp-1698258215261"></a>**2.  范围查询**
联合索引中，出现范围查询(>,<)，范围查询右侧的列索引失效

<a name="d7jm-1698259719246"></a>explain select\*from tb\_user where profession= '软件工程' and age > 30 and status ='O'; --status将失效
#### <a name="noqp-1698259719247"></a>**3.  索引失效情况1**
##### <a name="73m3-1698259958067"></a>**.索引列运算**
不要在索引列上进行运算操作,索引将失效。

<a name="xod2-1698259915148"></a>explain select \* from tb\_user where substring(phone,10,2) = '15';
##### <a name="jfiy-1698259897450"></a>**.字符串不加引号**
字符串类型字段使用时，不加引号,索引将失效。

select \* from tb\_user where phone = 17799990015; --隐式类型转换

<a name="9bq2-1698260050291"></a>select \* from tb\_user where phone = '17799990015'; --优化后走查询
##### <a name="ys0a-1698260050292"></a>**.模糊查询**
模糊查询是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效。

select \*from tb\_user where profession like '%软件'; -- 索引失效（大数据量避免头部模糊，会全表扫描）

<a name="pzdz-1698260181917"></a>select \*from tb\_user where profession like '软件%';  -- 索引有效
#### <a name="smyy-1698260181918"></a>**4.  索引失效情况2**
##### <a name="e8xu-1698260409971"></a>**.or连接的条件**
如果or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到。

explain select \* from tb\_user where id=10 or age = 23; --不回用到索引

#解决针对age建立索引

<a name="deyi-1698260481407"></a>explain select \* from tb\_user where id=10 or age = 23; --用到两条索引
##### <a name="4ymt-1698260481408"></a>**.数据分布影响**
如果mysql评估使用索引比全表更慢，则不使用索引。--查大部分数据（临界点是？），则不会走索引

select \*from tb\_user where phone >=17799990005'; --走索引，要查绝大部分数据

select \*from tb\_user where phone >='17799990015'; --不走索引，不需要查绝大部分数据

<a name="bg17-1698260635692"></a>phone is null 和phone is not null 是否走索引也是受数据分布影响，mysql评估
#### <a name="lm6a-1698260620466"></a>**5.  SQL提示-指定使用哪个索引**
SQL提示，是优化数据库的一个重要手段，简单来说，就是在SQL语句中加入一些人为的提示来达到优化操作的目的。

use index: -建议

explain select \* from tb\_user use index(idx\_user\_pro) where profession= '软件工程';

ignore index: -忽略

explain select \* from tb\_user ignore index(idx\_user\_pro) where profession= '软件工程';

force index: -强制

<a name="lss6-1698261564300"></a>explain select \*from tb\_user force index(idx\_user\_pro) where profession= '软件工程';
#### <a name="7kuw-1698261564301"></a>**6.  索引覆盖&回表查询**
覆盖索引

尽量使用覆盖索引（查询使用了索引，并且需要返回的列，在该索引中已经全部能够找到)，减少select \*

-不回，叶子节点就是id (using where;using index )

explain select id, profession from tb\_user where profession= '软件工程’ and age=31 and status = 'O' ;

-不回，叶子节点就是id (using where;using index )

explain select id,profession,age,status from tb\_user where profession='软件工程' and age=31 and status = '0' ;

-回表查主键索引拿叶子节点的name列 (using index condition )

explain select id,profession,age ,status, name from tb\_user where profession ='软件工程' and age=31 and status = '0';

-回表

explain select \* from  tb\_user where profession= '软件工程' and age = 31 and status= 'O' ;

select \* from tb\_user where id = 2; -主键索引不用回表

select \* from tb\_user where name = 'James'; -二级索引，非索引列字段需要回表查询

知识小贴士:

using index condition :查找使用了索引，但是需要回表查询数据

using where;using index :查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据；根据id回表查主键索引获取其他列

##################

一张表,有四个字段(id, username, password , status),由于数据量大,需要对以下SQL语句进行优化,该如何进行才是最优方案:

select id,username,password from tb\_user where username = 'itcast';

#1. 建username索引，先查id，再根据id查字段

<a name="yctg-1698261747664"></a>#2. 最优解，建立username和password联合索引，无需回表查询
#### <a name="lloi-1698261747665"></a>**7.  前缀索引-缩减大文本索引**
当字段类型为字符串(varchar, text等）时，有时候需要索引很长的字符串，这会让索引变得很大，查询时，浪费大量的磁盘l0，影响查询效率。

此时可以只将字符串的一部分前缀，建立索引，这样可以大大节约索引空间，从而提高索引效率。

create index idx\_xxxx on table\_name(column(n));

\>前缀长度

可以根据索引的选择性来决定，而选择性是指不重复的索引值〈基数）和数据表的记录总数的比值，索引选择性越高则查询效率越高，

唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的。

select count(distinct email) / count(\*) from tb\_user ; 结果=1 //count(email)-不为空条数

select count(distinct substring(email,1,5)) / count(\*) from tb\_user ; 结果是0.958

create index idx\_email\_5 on tb\_user (email(5));

<a name="akw2-1698263059797"></a>流程，回表后找到列数据，再全字符串对比email字段，正确再返回
#### <a name="hbtk-1698263059798"></a>**8.  单列索引&联合索引**
单列索引与联合索引

单列索引:即一个索引只包含单个列。联合索引:即一个索引包含了多个列。（第一个列必须存在，注意性能）

在业务场景中，如果存在多个查询条件，考虑针对于查询字段建立索引时，建议建立联合索引（覆盖索引），而非单列索引（可能回表变慢）。

单列索引情况:

<a name="lwzp-1698350476889"></a>多条件联合查询时，MySQL优化器会评估哪个字段的索引效率更高，会选择该索引完成本次查询。
#### <a name="xqe0-1698350463296"></a>**9.  设计原则**
1\.针对于数据量较大，且查询比较频繁的表建立索引。

2．针对于常作为查询条件(where)、排序（order by)、分组(group by)操作的字段建立索引。

3\.尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。

4．如果是字符串类型的字段，字段的长度较长，可以针对于子段的特扁，建工刖袋系引。

5．尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率。

6\.要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大,会影响增删改的效率。

7．如果索引列不能存储NULL值，请在创建表时使用NOT NULL约束它。当优化器知道每列是否包含NULL值时，它可以更好地确定哪个

<a name="xk0f-1698351177865"></a>索引最有效地用于查询。
## <a name="6tgn-1698351177866"></a><a name="_toc19823"></a>**视图**
#### <a name="9jqw-1698936202186"></a>**1.  介绍**
#### <a name="pgws-1698936204054"></a>**2.  作用**
#### <a name="bmjn-1698936206457"></a>**3.  创建查看删除**
·介绍

视图（View)是一种虚拟存在的表(代理罗？)。视图中的数据并不在数据库中实际存在，行和列数据来自定义视图的查询中使用的表，并且是在使用视图时动态生成的。

通俗的讲，视图只保存了查询的SQL逻辑，不保存查询结果。所以我们在创建视图的时候，主要的工作就落在创建这条SQL查询语句上。

作用：提供数据给其他部门

简单：视图不仅可以简化用户对数据的理解，也可以简化他们的操作。那些被经常使用的查询可以被定义为视图，从而使得用户不必为以后的操作每次指定全部的条件。

安全：数据库可以授权，但不能授权到数据库特定行和特定的列上。通过视图用户只能查询和修改他们所能见到的数据

数据独立：视图可帮助用户屏蔽真实表结构变化带来的影响。

创建CREATE[OR REPLACE] VIEW视图名称[(列名列表)]ASSELECT语句[WITH[ CASCADED | LOCAL ] CHECK OPTION]J

查看创建视图语句:SHOW CREATE VIEW视图名称;

查看视图数据:SELECT \*FROM视图名称....... ;

修改方式一:CREATE [OR REPLACE] VEW视图名称[(列名列表)]AS SELECT语句[WTH [ CASCADED | LOCAL] CHECK OPTION]

方式二:ALTER VIEW 视图名称[(列名列表)]AS SELECT语句[ WTH [ CASCADED | LOCAL ] CHECK OPTION]

删除DROP VIEW [IF EXISTS]视图名称[视图名称]....

案例：（代理）

1\. 为了保证数据库表的安全性，开发人员在操作tb\_user表时，只能看到的用户的基本字段，屏蔽手机号和邮箱两个字段。

<a name="vq4q-1698501711460"></a>2．查询每个学生所选修的课程（三张表联查)，这个功能在很多的业务中都有使用到，为了简化操作，定义一个视图。
## <a name="aozc-1698501711461"></a><a name="_toc21964"></a>**锁**
●介绍

锁是计算机协调多个进程或线程并发访问某一资源的机制。在数据库中，除传统的计算资源（CPU、RAM、I/O)的争用以外，数据也是

一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的

一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。

●分类

MySQL中的锁，按照锁的粒度分，分为以下三类:

1\.全局锁:锁定数据库中的所有表。-锁数据库实例

2\.表级锁:每次操作锁住整张表。

<a name="nwub-1698630097401"></a>3.行级锁:每次操作锁住对应的行数据。
### <a name="34q7-1698630097402"></a><a name="_toc6486"></a>**-全局锁**
#### <a name="9wq1-1698630617019"></a>**1.  数据库逻辑备份**
●介绍

全局锁就是对整个数据库实例加锁，加锁后整个实例就处于只读状态，后续的DML的写语句，DDL语句，已经更新操作的事务提交语句都

将被阻塞。

其典型的使用场景是做全库的逻辑备份，对所有的表进行锁定，从而获取一致性视图，保证数据的完整性。

电商商品 库存-订单-订单日志 表备份(mysqldump) 顺序执行，未加锁可能会导致数据不一致

1\.flush tables with read lock ;  --全局锁

2\.mysqldump -uropt -p1234 itcast > itcast.sql  --指定itcast库

3\.unlock tables ; --释放锁

特点

数据库中加全局锁，是一个比较重的操作，存在以下问题:

1\.如果在主库上备份，那么在备份期间都不能执行更新，业务基本上就得停摆。

<a name="cdwr-1698630272961"></a>2．如果在从库上备份，那么在备份期间从库不能执行主库同步过来的二进制日志(binlog)，会导致主从延迟。
#### <a name="1uni-1698630145465"></a>**2.  InnoDB不加锁一致性备份**
\# 优化全局锁(太重)

在InnoDB引擎中，我们可以在备份时加上参数--single-transaction参数来完成不加锁的一致性数据备份。

<a name="fznq-1698632304931"></a>mysqldump --single-transaction -uroot -p123456 itcast > itcast.sql --快照读
### <a name="hzlu-1698632267315"></a><a name="_toc19210"></a>**-表级锁**
●介绍

表级锁，每次操作锁住整张表。锁定粒度大，发生锁冲突的概率最高，并发度最低。应用在MyISAM、InnoDB、BDB等存储引擎中。

对于表级锁，主要分为以下三类:

1\.表锁

2．元数据锁（ meta data lock，MDL)

<a name="z0gx-1698631445867"></a>3．意向锁
#### <a name="oonk-1698631410993"></a>**1.  表锁-共享读独占写**
##### <a name="rgnc-1698949909492"></a>**.表共享读锁**
##### <a name="flt3-1698949914801"></a>**.表独占写锁**
对于表锁，分为两类:

1\.表共享读锁( read lock ) --共享读，皆不可写 Table 'score ' was locked with a READ lock and can't be upddated

2\.表独占写锁（ write lock) --读写独占，其他客户端读写阻塞知道释放。

语法:

1\.加锁:lock tables表名... read/write。

<a name="srzl-1698631479405"></a>2.释放锁:unlock tables /客户端断开连接。
#### <a name="mbvi-1698631471760"></a>**2.  元数据锁-指表结构**
##### <a name="vgvr-1698949948193"></a>**.MDL共享读-curd操作**
##### <a name="m0yt-1698949965870"></a>**.MDL排他写-DDL操作**
●元数据锁( meta data lock,MDL) 为了避免DML与DDL冲突，保证读写的正确性。

MDL加锁过程是系统自动控制，无需显式使用，在访问一张表的时候会自动加上。MDL锁主要作用是维护表元数据的数据一致性，

在表上有活动事务的时候，不可以对元数据进行写入操作。

在My5QL5.5中引入了MDL，当对一张表进行增删改查的时候，加MDL读锁(共享);当对表结构进行变更操作的时候，加MDL写锁(排他)。

lock tables xxx read / write      SHARED\_READ\_ONLY / SHARED\_NO\_READ\_WRITE

select . select ... lock in share mode   SHARED\_READ                //与SHARED\_READ、SHARED\_WRITE兼容，与EXCLUSIVE互斥

insert . update、 delete、 select ... for update    SHARED\_WRITE    //与SSHARED\_READ、SHARED\_WRITE兼容,与EXCLUSIVE互斥

alter table ...    EXCLUSIVE排他       // 与其他的MDL都互斥

示例：又tm手动开事务 

1\. A-begin A-select（MDL-SHARED\_READ） B-alter table(阻塞) A-commit B-alter执行

查看元数据锁:

select object type,object\_schema,object\_name,lock\_type,lock\_duration from performa

<a name="6ap7-1698631434004"></a>nce\_schema.metadata\_locks ; //手动开事务，然后select查会有MDL锁
#### <a name="uldg-1698632501087"></a>**3.  意向锁-行锁表锁冲突提效**
##### <a name="lihy-1698950039794"></a>**.意向共享（IS）**
##### <a name="vokq-1698950064112"></a>**.意向排他（IX）**
●意向锁

为了避免DML在执行时，加的行锁与表锁的冲突，在InnoDB中引入了意向锁，使得表锁不用检查每行数据是台加锁，使用急问锁米减少表锁的检查。

1\.意向共享锁（ IS):由语句select ... lock in share mode添加。

2．意向排他锁（IX)︰由insert、update、delete、select ... for update添加。

1．意向共享锁（IS)︰与表锁共享锁( read）兼容，与表锁排它锁(write)互斥。

2．意向排他锁（IX)∶与表锁共享锁(read)及排它锁(write)都互斥。意向锁之间不会互斥。

可以通过以下SQL，查看意向锁及行锁的加锁情况:

select object\_schema,object\_name,index\_name,lock\_type,ock\_mode,ock\_data from perfarmance\_schema.data\_locks;

示例：

A-begin A-update(行锁，意向锁) B-lock table xxx read/wirte(阻塞，无意向锁会逐行检查有无行锁-性能低) 

<a name="3xwl-1698631438115"></a>A-commit B-lock执行
### <a name="lhar-1698630153890"></a><a name="_toc19042"></a>**-行级锁：基于索引项**
●介绍

行级锁，每次操作锁住对应的行数据。锁定粒度最小，发生锁冲突的概率最低，并发度最高。应用在InnoDB存储引擎中。

InnoDB的数据是基于索引组织的，行锁是通过对索引上的索引项加锁来实现的，而不是对记录加的锁。对于行级锁，主要分为以下三类:

1\.行锁(Record Lock):锁定单个行记录的锁，防止其他事务对此行进行update和delete。在RC、RR隔离级别下都支持。

2．间隙锁(Gap Lock):锁定索引记录间隙(不含该记录），确保索引记录间隙不变，防止其他事务在这个间隙进行insert，产生幻读。

在RR隔离级别下都支持。

3．临键锁(Next-Key Lock)∶行锁和间隙锁组合，同时锁住数据，并锁住数据前面的间隙Gap。在RR隔离级别下支持。

查看锁表

<a name="ckgn-1698785009241"></a>record  s 7,7 临键锁 锁id=7行,锁7与之前行的间隙
#### <a name="ihem-1698785009242"></a>**1.  行锁/记录锁**
##### <a name="fqsj-1698950110507"></a>**.共享锁（S）**
##### <a name="frfu-1698950116296"></a>**.排他锁（X）**
InnoDB实现了以下两种类型的行锁:

1．共享锁(S）∶允许一个事务去读一行，阻止其他事务获得相同数据集的排它锁。

2．排他锁（X)︰允许获取排他锁的事务更新数据，阻止其他事务获得相同数据集的共享锁和排他锁。

请求锁类型    S(共享锁)    X(排他锁)

当前锁类型

s (共享锁)    兼容        冲突

×(排他锁)    冲突          冲突

INSERT ...排他锁    自动加锁   -----兼容性其实就是自动事务表锁共享读独占写锁（排他写锁可读不阻塞是因为读的快照）实现的体现？

UPDATE ...排他锁    自动加锁

DELETE ...排他锁    自动加锁

SELECT（正常)不加任何锁

SELECT ... LOCK IN SHARE MODE共享锁    需要手动在SELECT之后加LOCK IN SHARE MODE

SELECT ... FOR UPDATE排他锁    需要手动在SELECT之后加FOR UPDATE

行锁-演示

默认情况下，InnoDB在REPEATABLE READ事务隔离级别运行，InnoDB使用next-key锁（临键锁）进行搜索和索引扫描，以防止幻读。

1．针对唯一索引进行检索时，对已存在的记录进行等值匹配时，将会自动优化为行锁。

2\. InnoDB的行锁是针对于索引加的锁，不通过索引条件检索数据，那么InnoDB将对表中的所有记录加锁，此时就会升级为表锁。

手动开启事务

A-update stu set name='Lei' where name='lilu';--id=4

B-udpate stu set name='Peter' where id=3 ; --阻塞，虽然id不同(不同行)，但是name查询不走索引，升级表锁；

A-commit;

<a name="uvdc-1698785564250"></a>B-query OK;
#### <a name="092u-1698785365284"></a>**2.  间隙锁：RR下解决幻读**
●间隙锁/临键锁-演示

默认情况下，InnoDB在REPEATABLE READ事务隔离级别运行，InnoDB使用next-key锁（临建锁）进行搜索和索引扫描，以防止幻读。

1\.索引上的等值查询(唯一索引)，给不存在的记录加锁时,优化为间隙锁。

2．索引上的等值查询(普通索引)，向右遍历时最后一个值不满足查询需求时，next-key lock退化为间隙锁。

3．索引上的范围查询(唯一索引)--会访问到不满足条件的第一个值为止。

注意:间隙锁唯一目的是防止其他事务插入间隙。间隙锁可以共存，一个事务采用的间隙锁不会阻止另一个事务在同一间隙上采用间隙锁。

演示1 手动开启事务

A-update stu set age = 10 where id = 5; --无id=5的记录，锁id 3^8之间gap

B-insert into stu values (7,'Ruby',7); --阻塞，有gap锁

演示2

<a name="lvdd-1698786993879"></a>A-select \* from stu where id >=19  lock in share mode;  -- 19>25>+ s,rec\_not\_gap 19 //行锁,  s 25 //临间锁, s pre+ //25+  
#### <a name="2cip-1698785368743"></a>**3.  临键 锁**
<a name="ld4a-1698934283059"></a>s 行锁+间隙锁；  s-reco\_not\_gap 行锁； s gap 间隙锁
## <a name="wv5u-1698934641347"></a><a name="_toc12971"></a>**InnoDB引擎架构**
### <a name="xxte-1698934785966"></a><a name="_toc7120"></a>**-逻辑存储结构**
#### <a name="rijc-1698936536741"></a>**1.  表空间（ibd文件）**
#### <a name="hcj0-1698936614757"></a>**2.  段（索引段）**
#### <a name="trpj-1698936634864"></a>**3.  区（1M,64页 ）**
#### <a name="pnjg-1698936652874"></a>**4.  页 （16K-磁盘管理的最小单元，申请4-5区 ）**
#### <a name="o35i-1698936658205"></a>**5.  行 （Trx\_id-隐藏列，Rall\_pointer-Undolog指针）**
表空间（ibd文件)，一个mysql实例可以对应多个表空间，用于存储记录、索引等数据。

段，分为数据段（Leaf node segment)、索引段（Non-leaf node segment)、回滚段(Rollback segment)，InnoDB是索引组织表，

`    `数据段就是B+树的叶子节点，索引段即为B+树的非叶子节点。段用来管理多个Extent(区)

区，表空间的单元结构，每个区的大小为1M。默认情况下，InnoDB存储引擎页大小为16K，即一个区中一共有64个连续的页。

页，是InnoDB存储引擎磁盘管理的最小单元，每个页的大小默认为16KB。为了保证页的连续性，InnoDB存储引擎每次从磁盘申请4-5个区。

行，InnoDB存储引擎数据是按行进行存放的。

Trx\_id:每次对某条记录进行改动时，都会把对应的事务id赋值给trx\_id隐藏列。

Rall\_pointer:每次对某条引记录进行改动时，都会把旧的版本写入到undo日志中，然后这个隐藏列就相当于一个指针，

可以通过它来找到该记录修改前的信息。

<a name="b5hk-1698936466080"></a>row\_id:无主键时才会生成隐式主键
### <a name="cns1-1698936283052"></a><a name="_toc15253"></a>**-内存架构(内存+磁盘)**
MySQL5.5版本开始，默认使用InnoDB存储引擎，它擅长事务处理，具有崩溃恢复特性，在日常开发中使用非常广泛。

下面是InnoDB架构图，左侧为内存结构，右侧为磁盘结构。

<a name="news-1698937385247"></a>缓存加载数据进行CRUD操作后(脏页即修改页)刷入磁盘，undo redo log保存部分数据用于恢复
#### <a name="cbz5-1698936905672"></a>**1.  Buffer Pool(减少磁盘IO）**
#### <a name="zlsn-1698937679189"></a>**2.  Change Buffer(针对于非唯一二级索引页，合并处理减少磁盘IO）**
#### <a name="tt1u-1698937908124"></a>**3.   Adaptive Hash lndex(自动优化对Buffer Pool的查询)**
#### <a name="qkiu-1698938073529"></a>**4.  Log Buffer（undo,redo）**

<a name="62h7-1698938041675"></a>Buffer Pool:缓冲池是主内存中的一个区域，里面可以缓存磁盘上经常操作的真实数据，在执行增删改查操作时，（80%内存分配）

先操作缓冲池中的数据（若缓冲池没有数据，则从磁盘加载并缓存），然后再以一定频率刷新到磁盘，从而减少磁盘IO，加快处理速度。

缓冲池以Page页为单位，底层采用链表数据结构管理Page。根据状态，将Page分为三种类型:

. free page:空闲page，未被使用。

. clean page:被使用page，数据没有被修改过。

. dirty page:脏页，被使用page，数据被修改过，页中数据与磁盘的数据产生了不一致。

Change Buffer:更改缓冲区（针对于非唯一二级索引页），在执行DML语句时，如果这些数据Page没有在Buffer Pool中，不会直接操作磁盘，

而会将数据变更存在更改缓冲区Change Buffer中，在未来数据被读取时，再将数据合并恢复到Buffer Pool中，再将合并后的数据刷新到磁盘中。

Change Buffer的意义是什么?

与聚集索引不同，二级索引通常是非唯一的，并且以相对随机的顺序插入二级索引。同样，删除和更新可能会影响索引树中不相邻的二级索引页，

如果每一次都操作磁盘，会造成大量的磁盘IO。有了ChangeBuffer之后，我们可以在缓冲池中进行合并处理，减少磁盘IO。

Adaptive Hash lndex:自适应hash索引，用于优化对Buffer Pool数据的查询。InnoDB存储引擎会监控对表上各索引页的查询，

如果观察到hash索引可以提升速度，则建立hash索引，称之为自适应hash索引。

自适应哈希索引，无需人工干预，是系统根据情况自动完成。参数: adaptive\_hash\_index

Log Buffer:日志缓冲区，用来保存要写入到磁盘中的log日志数据（ redo log . undo log)，默认大小为16MB，日志缓冲区的日志会定期刷新到

磁盘中。如果需要更新、插入或删除许多行的事务，增加日志缓冲区的大小可以节省磁盘l/O。

参数:

innodb\_log\_buffer\_size:缓冲区大小

innodb\_flush\_log\_at\_trx\_commit:日志刷新到磁盘时机

1:日志在每次事务提交时写入并刷新到磁盘

0:每秒将日志写入并刷新到磁盘一次。

<a name="srie-1698937436652"></a>2:日志在每次事务提交后写入,并每秒刷新到磁盘一次。
### <a name="exmr-1698937415872"></a><a name="_toc18201"></a>**- 磁盘结构**
#### <a name="0snm-1698939323164"></a>**1.  系统表空间（change buffer存储区域）**
#### <a name="nryg-1698939327127"></a>**2.  表空间（idb文件）**
#### <a name="juax-1698939336996"></a>**3.  通用表空间（指定表存放位置）**
#### <a name="rbyy-1698942461411"></a>**4.  undo表空间（撤销表空间）**
#### <a name="yffc-1698942468968"></a>**5.  临时表空间 （临时表数据）**
#### <a name="rubq-1698942871370"></a>**6.  双写缓冲区（双写缓冲区文件，异常恢复）**
#### <a name="ehmy-1698942904011"></a>**7. redo log （脏页刷磁盘错误时恢复数据）**
System Tablespace:系统表空间是更改缓冲区的存储区域。如果表是在系统表空间而不是每个表文件或通用表空间中创建的，它也可能包含表和

索引数据。(在MySQL5.x版本中还包含InnoDB数据字典、undolog等)

参数:innodb\_data\_file\_path  sql:show variables like ' %data\_file path%';  ibdata1文件(可能包含表和索引数据)

File-Per-Table Tablespaces:每个表的文件表空间包含单个InnoDB表的数据和索引，并存储在文件系统上的单个数据文件中。

参数: innodb\_file\_per\_table    ibd文件（表结构，索引，数据）

General Tablespaces:通用表空间，需要通过CREATE TABLESPACE语法创建通用表空间，在创建表时，可以指定该表空间。

CREATE TABLESPACE xxxx ADDDATAFILE 'file\_name' ENGINE=engine\_name;  --生成ibd

CREATE TABLE xxx ... TABLESPACE ts\_ame;

Undo Tablespaces:撤销表空间，MySQL实例在初始化时会自动创建两个默认的undo表空间(初始大小16M)，用于存储undo log日志。

undo\_001  undo\_002

Temporary Tablespaces: InnoDB使用会话临时表空间和全局临时表空间。存储用户创建的临时表等数据。

Doublewrite Buffer Files:双写缓冲区，innoDB引擎将数据页从Buffer Pool刷新到磁盘前，先将数据页写入双写缓冲区文件中，

便于系统异常时恢复数据。#ib\_16384\_0.dblwr #ib\_16384\_1.dblv

Redo Log:重做日志，是用来实现事务的持久性。该日志文件由两部分组成:重做日志缓冲(redo log buffer)以及重做日志文件(redo log),

前者是在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都会存到该日志中,用于在刷新脏页到磁盘时,发生错误时,进行数据恢复使用。

<a name="1vs7-1698937441985"></a>以循环方式(顺序io?)写入重做日志文件，涉及两个文件:ib\_logfile0    ib\_logfile1
### <a name="53jq-1698937441986"></a><a name="_toc11972"></a>**-后台线程（内存磁盘交互）**
#### <a name="kknd-1698947048446"></a>**1. Master Thread**
#### <a name="5vjm-1698947052890"></a>**2. IO Thread（AIO异步IO）**
#### <a name="xqib-1698947176665"></a>**3.   Purge Thread**
#### <a name="mdxm-1698947181806"></a>**4.  Page Cleaner Thread**
1\.Master Thread

核心后台线程，负责调度其他线程，还负责将缓冲池中的数据异步刷新到磁盘中,保持数据的一致性，还包括脏页的刷新、合并插入缓存、

undo页的回收。

2\.IO Thread

在InnoDB存储引擎中大量使用了AIO来处理IO请求,这样可以极大地提高数据库的性能，而IOThread主要负责这些IO请求的回调。

Read thread    4    负责读操作

Write thread    4    负责写操作

Log thread    1    负责将日志缓冲区刷新到磁盘

lnsert buffer thread    1    负责将写缓冲区内容刷新到磁盘

3\. Purge Thread

主要用于回收事务已经提交了的undo log，在事务提交之后，undo log可能不用了，就用它来回收。

4\.Page Cleaner Thread

<a name="v1x1-1698943342220"></a>协助Master Thread刷新脏页到磁盘的线程，它可以减轻Master Thread 的工作压力，减少阻塞。
## <a name="oqzx-1698943342221"></a><a name="_toc14607"></a>**InnoDB事务原理**
#### <a name="3c9f-1699135695351"></a>**1.原子性 undolog**
#### <a name="0fpr-1699135700379"></a>**2.一致性 undolog redolog**
#### <a name="v24b-1699135704557"></a>**3.持久性 redolog**
#### <a name="lhxo-1699135708824"></a>**4.隔离性 锁 MVCC(隐藏字段+undolog版本链+readView)** 
原子性、一致性、持久性：undo log ，redo log

<a name="d7h6-1698948119438"></a>隔离性：锁、MVCC
### <a name="3xv9-1698948119439"></a> **<a name="_toc30334"></a>-redo log -持久性>WAL先写日志刷磁盘**
redo log -持久性

重做日志，记录的是事务提交时数据页的物理修改，是用来实现事务的持久性。

该日志文件由两部分组成:重做日志缓冲(redo log buffer)以及重做日志文件（(redo log file) ,前者是在内存中，后者在磁盘中。

当事务提交之后会把所有修改信息都存到该日志文件中，用于在刷新脏页到磁盘,发生错误时,进行数据恢复使用。

作用：DML-commit 到Buffer pool脏页,会先不刷到磁盘ibd文件中（所有数据，随机IO,性能低）

`     `Redolog buffer 会先刷到磁盘ib\_logflie0/1(redo log)文件中（数据页变化，顺序io,重复利用，头尾覆盖，性能高），保证脏页刷磁盘

`           `发生错误时，能够进行数据恢复使用；WAL(Write-Ahead Logging) 先写日志；

by the way-CopyOnWrite(写时复制)：JDK并发读写问题经常用该方案解决脏读，多线程操作一份实例，写时先拷贝一份，修改后再覆盖原实例。与

`    `数据库解决脏读隔离级别‘读已提交’有异曲同工之妙！

<a name="f9jx-1698948472858"></a>事务这种类似缓存一样的中间副本及并发事务问题解决方案（锁、mvcc实现的隔离性），是否是其他系统解决并发读写的方案。
### <a name="eiip-1698948472859"></a><a name="_toc2221"></a>**-undo log -原子性-回滚-MVCC**
回滚日志，用于记录数据被修改前的信息，作用包含两个:提供回滚和MVCC(多版本并发控制)。undo log和redo log记录物理日志不一样，

它是逻辑日志。可以认为当delete一条记录时undo log中会记录一条对应的insert记录，

反之亦然，当update一条记录时，它记录一条对应相反的update记录。当执行rollback时，就可以从undo log中的逻辑记录读取到相应的内容

`    `并进行回滚。

Undo log销毁: undo log在事务执行时产生，事务提交时，并不会立即删除undo log，因为这些日志可能还用于MVCC。

<a name="oema-1698951285219"></a>Undo log存储: undo log采用段的方式进行管理和记录，存放在前面介绍的 rollback segment 回滚段中，内部包含1024个undo logsegment。
## <a name="fj9w-1698951285220"></a><a name="_toc5210"></a>**MVCC-快照读历史版本**
### <a name="bu2g-1698952158239"></a><a name="_toc1429"></a>**-基本概念**
#### <a name="acm7-1698953042131"></a>**1.  当前读（最新版本-加锁的都是）**
#### <a name="dtns-1698955322144"></a>**2.  快照读 （历史版本-RR下开启事务的第一个Select）**
#### <a name="mi8v-1698956161619"></a>**3. MVCC（维护一个数据的多个版本，提供非阻塞读）**
当前读：读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。对于我们日常的操作，

如:select ..lock in share mode(共享锁)，select... for update、update、insert、delete(排他锁)都是一种当前读。

示例：

A-select id=1 ===> java  --开启事务后第一个select语句才是快照读的地方。

B- update stu set name = 'Jsp' where id = 1; //当前锁为recor锁排他锁，其他事务不能读取到当前数据？

A- select id=1 = ==> java  --快照读，读取的时历史版本，非阻塞读

B- commit

A- select id=1 = ==> java  //默认RR隔离级别，可重复读-即读另一个事务开启前的数据， A-commit 后再 select

解决： select.... lock in share mode; --当前读：最新数据

快照读

简单的select (不加锁）就是快照读，快照读，读取的是记录数据的可见版本，有可能是历史数据，不加锁，是非阻塞读。

. Read Committed:每次select，都生成一个快照读。

. Repeatable Read:开启事务后第一个select语句才是快照读的地方。

. Serializable:快照读会退化为当前读。

MVCC

全称 Multi-version Concurrency Control，多版本并发控制。指维护一个数据的多个版本，使得读写操作没有冲突，快照读为MySQL实现

<a name="thwv-1698952270152"></a>MVCC提供了一个非阻塞读功能。MVCC的具体实现，还需要依赖于数据库记录中的三个隐式字段、undo log日志、readView.
### <a name="p3na-1698952163363"></a><a name="_toc9295"></a>**-隐藏字段**
#### <a name="uwuz-1699301956379"></a>**1.DB\_TRX\_ID事务id**
#### <a name="mqaz-1699301965028"></a>**2.DB\_ROLL\_PTR行指针**
#### <a name="thdk-1699301971563"></a>**3.DB\_ROw\_ID隐藏主键**
DB\_TRX\_ID  :  最近修改事务ID，记录插入这条记录或最后一次修改该记录的事务ID。

DB\_ROLL\_PTR //Pointer:回滚指针，指向这条记录的上一个版本，用于配合undo log，指向上一个版本

<a name="e8jj-1699121892418"></a>DB\_ROw\_ID: 隐藏主键，如果表结构没有指定主键，将会生成该隐藏字段。
### <a name="e2dp-1698952167493"></a><a name="_toc9042"></a>**-undolog版本链**
回滚日志，在insert、update、delete的时候产生的便于数据回滚的日志。

当insert的时候，产生的undo log日志只在回滚时需要，在事务提交后，可被立即删除。

而update、delete的时候，产生的undo log日志不仅在回滚时需要，在快照读时也需要，不会立即被删除。

事务2                            事务3                    事务4                事务5

开始事务                        开始事务                  开始事务            开始事务

修改id为30记录. age改为3                                查询id为30的记录

提交事务                修改id为30记录. name改为A3                            查询id为30的记录 

`                                `提交事务

`                                                    `修改id为30记录. age改为10

`                                                    `查询id为30的记录

`                                                                            `查询id为30的记录

`                                                           `提交事务





记录：         id    age    name    DB\_TRX\_ID     DB\_ROLL\_PTR

`              `30     30      A30       事务1        NULL

`              `30     3      A30       事务2         Ox00001

`              `30     3      A3       事务3         Ox00002

`              `30     10      A3       事务4         Ox00003

undo log:

`    `Ox00003    30     3      A3       事务3         Ox00002

`    `Ox00002   30     3      A30       事务2         Ox00001                    

`    `0x00001   30    30      A30        事务1        null

先生成undo log, 再表记录

不同事务或相同事务对同一条记录进行修改，会导致该记录的undolog生成一条记录版本链表，链表的头部是最新的旧记录，链表尾部是最

<a name="h7kk-1699122395876"></a>早的旧记录。
### <a name="qc3g-1698952179087"></a><a name="_toc15714"></a>**-readview读视图**
ReadView (读视图)是快照读SQL执行时MVCC提取数据的依据，记录并维护系统当前活跃的事务（未提交的) id。ReadView中包含了四个核心字段:

m\_ids    当前活跃的事务ID集合

min\_trx\_id    最小活跃事务ID

max\_trx\_id    预分配事务ID,当前最大事务ID+1（因为事务ID是自增的>

creator\_trx\_id    ReadView创建者的事务ID

访问规则:感觉是根据时序图+隔离级别倒推出规律规则-满足任意一条

trx\_id:代表是当前事务ID。

1\. trx\_id == creator\_trx\_id ?可以访问该版本成立，说明数据是当前这个事务更改的。

2\.trx\_id < min\_trx\_id ?可以访问该版本成立，说明数据已经提交了。

3\.trx\_id > max\_trx\_id ?不可以访问该版本成立，说明该事务是在ReadView生成后才开启。

4\.min\_trx\_id <=trx\_id <= max\_trx\_id ?如果trx\_id不在m\_ids中是可以访问该版本的成立，说明数据已经提交。

不同的隔离级别，生成ReadView的时机不同:

READ COMMITTED︰在事务中每一次执行快照读时生成ReadView.

<a name="igth-1699124847824"></a>REPEATABLE READ:仅在事务中第一次执行快照读时生成ReadView，后续复用该ReadView.
### <a name="zysq-1698952216492"></a><a name="_toc28633"></a>**-原理分析(RC级别)**
事务5第一次查询id为30的记录 ReadView1--  m\_ids: {3,4,5} min\_trx\_id: 3 max\_trx\_\_id: 6creator \_trx\_id: 5

1\. trx\_id == creator\_trx\_id ? 5

2\.trx\_id < min\_trx\_id ? 3

3\.trx\_id > max\_trx\_id ? 6

4\.min\_trx\_id <=trx\_id <= max\_trx\_id ?  m\_ids {3,4,5}

根据规则查版本链确定快照读版本 

当前最新记录 事务为4（未提交）不成立 > undolog版本链 事务3（未提交）不成立  > 事务2（已提交）成立（返回该版本记录）

事务5第二次查询id为30的记录ReadView2 -- m\_ids: {4,5}.min\_trx\_id: 4max\_trx\_id: 6 creator\_trx\_id: 5 m\_ids {4,5}

<a name="bngd-1699126608355"></a>当前最新记录 事务为4（未提交）不成立 > undolog版本链 事务3（已提交）不成立。
### <a name="ebqu-1698952246400"></a><a name="_toc17585"></a>**-原理分析(RR级别)**
RR隔离级别下，仅在事务中第一次执行快照读时生成ReadView，后续复用该ReadView.

事务5第一次查询id为30的记录 ReadView1--  m\_ids: {3,4,5} min\_trx\_id: 3 max\_trx\_\_id: 6creator \_trx\_id: 5

1\. trx\_id == creator\_trx\_id ? 5

2\.trx\_id < min\_trx\_id ? 3

3\.trx\_id > max\_trx\_id ? 6

4\.min\_trx\_id <=trx\_id <= max\_trx\_id ?  m\_ids {3,4,5}

根据规则查版本链确定快照读版本 

当前最新记录 事务为4（未提交）不成立 > undolog版本链 事务3（未提交）不成立  > 事务2（已提交）成立（返回该版本记录）

<a name="z3gv-1699135234616"></a>事务5第二次查询id为30的记录ReadView(复用ReadView1)
## <a name="9uz0-1699135234617"></a><a name="_toc30428"></a>**管理**
### <a name="wzzy-1699200494327"></a><a name="_toc11759"></a>**-系统数据库-自带库**
Mysql数据库安装完成后，自带了一下四个数据库，具体作用如下:

mysql：存储MysQL服务器正常运行所需要的各种信息(时区、主从、用户、权限等)

information\_schema：提供了访问数据库元数据的各种表和视图，包含数据库、表、字段类型及访问权限等

performance\_schema：为MySQL服务器运行时状态提供了一个底层监控功能，主要用于收集数据库服务器性能参数 //锁 data\_locks metadata\_lk

<a name="pgfk-1699200544301"></a>sys:包含了一系列方便DBA和开发人员利用performance\_schema性能数据库进行性能调优和诊断的视图
### <a name="msjq-1699200469340"></a><a name="_toc16621"></a>**-常用工具1-客户端工具**
mysql

该mysql不是指mysql服务，而是指mysql的客户端工具。

语法: mysql [options] [database]

选项:

`    `-U, --user=name #指定用户名

`    `-p,--password[=name]#指定密码

`    `-h, --host=name#指定服务器IP或域名

`    `-P,--port=port#指定连接端口

`    `-e, --execute=name#执行SQL语句并退出



-e选项可以在Mysql客户端执行sQL语句，而不用连接到MysQL数据库再执行，对于一些批处理脚本，这种方式尤其方便。

示例:mysql -uroot -p123456 db01 -e "select \* from stu";

mysqladmin

mysqladmin是一个执行管理操作的客户端程序。可以用它来检查服务器的配置和当前状态、创建并删除数据库等。

通过帮助文档查看选项: mysqladmin --help

示例:mysqladmin -uroot -p123456 drop 'testO1'; mysqladmin -uroot -p123456 version;

mysqlbinlog—查看二进制文件

由于服务器生成的二进制日志文件以二进制格式保存，所以如果想要检查这些文本的文本格式，就会使用到mysqlbinlog日志管理工具。

语法:mysqlbinlog [options] log-files1 log-files2 ...

选项:

`    `-d , --database=name指定数据库名称，只列出指定的数据库相关操作。

`    `-O, --offset=t忽略掉日志中的前n行命令。

`    `-r,--result-file=name将输出的文本格式日志输出到指定文件。

`    `-s, --short-form显示简单格式，省略掉一些信息。

`    `--start-datatime=date1 --stop-datetime=date2指定日期间隔内的所有日志。

`    `--start-position=pos1 --stop-position=pos2指定位置间隔内的所有日志。

mysqlshow

mysqlshow客户端对象查找工具，用来很快地查找存在哪些数据库、数据库中的表表中的列或者索引。

语法:mysqlshow [options][db\_name [table\_name [col\_name]ll

选项:

`    `--count显示数据库及表的统计信息（数据库,表均可以不指定)

`    `-i显示指定数据库或者指定表的状态信息

示例:

`    `#查询每个数据库的表的数量及表中记录的数量    mysqlshow -uroot -p2143 --count

`    `#查询test库中每个表中的字段书，及行数    mysqlshow -uroot -p2143 test --count

<a name="gahk-1699205011846"></a>    #查询test库中book表的详细情况     mysqlshow -uroot -p2143 test book --count
### <a name="mjum-1699200526630"></a><a name="_toc15491"></a>**-常用工具2**
mysqldump

mysqldump客户端工具用求备份数据库或在不同数据库之间进行数据迁移。备份内容包含创建表，及插入表的sQL语句。

语法:

`    `mysqldump [options] db\_name [tables]

`    `mysqldump [options] --database/-B db1 [db2 db3...]mysqldump [options] --all-databases/-A

连接选项:

`    `-U,--user=name 指定用户名

`    `-p,--password[=name] 指定密码

`    `-h, --host=name 指定服务器ip或域名

`    `-P, --port一# 指定连接端口

输出选项:

`    `--add-drop-database 在每个数据库创建语句前加上drop database语句

`    `--add-drop-table 在每个表创建语句前加上drop table语句，默认开启;不开启(--skip-add-drop-table)

`    `-n,--no-create-db 不包含数据库的创建语句

`    `-t, --no-create-info 不包含数据表的创建语句

`    `-d --no-data 不包含数据

`    `-T, --tab=name 自动生成两个文件:一个.sql文件，创建表结构的语句;一个.txt文件，数据文件

mysqlimport/source

mysqlimport是客户端数据导入工具，用来导入mysqldump加-T参数后导出的文本文件。

语法:mysqlimport[options] db\_name textfile1 [textfile2...]

示例:mysqlimport-uroot-p2143 test /tmp/city.txt

如果需要导入sql文件,可以使用mysql中的source指令:

语法: source /root/xxxxx.sql

导入大量文件网上推荐方案 LOAD DATA INFILE   -导入3百万700M数据22s, 客户端导入则10分钟+  整型报错-将文件编码为UTF-8

LOAD DATA INFILE '/var/lib/mysql-files/fdlj\_20210810\_20211110\_03.csv' 

INTO TABLE cmd\_fdlj

FIELDS TERMINATED BY ',' 

LINES TERMINATED BY '\n'

(id,od,dep\_date\_str,dep\_date,airline,flt,@dep\_time,equipment)

SET dep\_time = IF(@dep\_time = 'NULL', '', @dep\_time);

Bypassing Transaction Overhead（绕过事务开销）：默认情况下，LOAD DATA INFILE 操作通常是非事务性的，这意味着它不会创建事务记录，

不进行日志记录和回滚，因此可以避免大量的事务管理开销。这使得数据的加载速度非常快，因为数据直接进入表而不需要额外的事务操作。

Direct Data Loading（直接数据加载）：LOAD DATA INFILE 将数据直接加载到表中，而不需要通过SQL语句一行一行地插入数据，这比逐行插入

数据要快得多。

Optimized File I/O（优化的文件输入/输出）：MySQL的实现对文件的读取和写入进行了优化，以加速数据加载过程。这包括使用适当的

缓冲和IO优化策略。

Concurrency Control（并发控制）：LOAD DATA INFILE 可以利用并发性，允许多个客户端同时加载数据，从而提高了效率。

Minimized Logging（最小化日志记录）：由于非事务性操作，LOAD DATA INFILE 不需要大量的写入和重做日志，因此减少了磁盘写入的开销。

No Index Maintenance（不进行索引维护）：通常，LOAD DATA INFILE 不会在加载数据时维护表的索引，这可以显著加快加载速度。

索引的重建通常在加载完成后进行。

总之，LOAD DATA INFILE 是一种高效的数据加载方法，特别适用于大量数据的批量加载。它通过绕过事务和使用优化的文件I/O策略，

<a name="gqas-1699215225233"></a>以及减少日志记录来实现快速加载。这是 MySQL 中常用的数据导入工具之一，可以提高数据导入的性能和效率。
## <a name="vl7r-1699215225234"></a> **<a name="_toc21635"></a>日志**
### <a name="v5kp-1699299216424"></a><a name="_toc13038"></a>**-错误日志-启动停止错误排查**
·错误日志

错误日志是MySQL中最重要的日志之一，它记录了当mysqld启动和停止时，以及服务器在运行过程中发生任何严重错误时的相关信息。当数据库

出现任何故障导致无法正常使用时，建议首先查看此日志。该日志是默认开启的，默认存放目录/var/log/，默认的日志文件名为 mysqld.log。

查看日志位置: show variables like '%log\_error%';   

示例： mysqld启动和停止 :services restart mysqld 报错

<a name="kliq-1699300738254"></a>$ tail [-f] -20 /var/log/mysqld.log // 查看尾部20行 -f:开启实时监控
### <a name="diuw-1699299216782"></a><a name="_toc4190"></a>**-二进制日志-数据恢复(记录DDL+DML)**
#### <a name="v10r-1699300840079"></a>**1.STATEMENT(sql)**
#### <a name="cjge-1699301022236"></a>**2.ROW(每一行变更前后数据)**
#### <a name="vuwo-1699301026582"></a>**3.MIXED(混合)**
·二进制日志

二进制日志〈BINLOG)记录了所有的DDL(数据定义语言）语句和DML（(数据操纵语言）语句，但不包括数据查询(SELECT、SHOW)语句。

作用:①.灾难时的数据恢复;②.MySQL的主从复制。在MySQL8版本中，默认二进制日志是开启着的，涉及到的参数如下;

show variables like '%log \_bin%'

示例：binlog.0001  binlog0002

MySQL服务器中提供了多种格式来记录二进制日志，具体格式及特点如下:

STATEMENT-基于sQL语句的日志记录，记录的是sQL语句，对数据进行修改的sQL都会记录在日志文件中。

ROW-基于行的日志记录，记录的是每一行的数据变更。（默认）// -v 重构为sql语句

MIXED-混合了STATEMENT和ROW两种格式，默认采用STATEMENT，在某些特殊情况下会自动切换为ROW进行记录。

show variables like '%binlog\_format%';

日志查看

由于日志是以二进制方式存储的，不能直接读取，需要通过二进制日志查询工具mysqlbinlog 来查看，具体语法:

mysqlbinlog[参数选项]logfilename

参数选项:

`    `-d 指定数据库名称，只列出指定的数据库相关操作。

`    `-o 忽略掉日志中的前n行命令。

`    `-V 将行事件(数据变更)重构为SQL语句

`    `-W 将行事件(数据变更)重构为SQL语句，并输出注释信息

日志删除

对于比较繁忙的业务系统，每天生成的binlog数据巨大，如果长时间不清除，将会占用大量磁盘空间。可以通过以下几种方式清理日志:

reset master 删除全部binlog日志，删除之后，日志编号，将从binlog.000001重新开始

purge master loas to 'binlon\*\*\*\*\*\*l 删除\*\*\*\*\*\*编号之前的所有日志

purge master logs before 'yyyy-mm-dd hh24:mi:ss' 删除日志为"yyyy-mm-dd hh24:mi:ss"之前产生的所有日志

也可以在mysql的配置文件中配置二进制日志的过期时间，设置了之后，二进制日志过期会自动删除。

<a name="btnu-1699300763098"></a>show variables like '%binlog\_expire\_logs\_seconds%'; --默认存放2592000=30天
### <a name="txwu-1699299221578"></a><a name="_toc2601"></a>**-查询日志(二进制无查询)**
查询日志中记录了客户端的所有操作语句，而二进制日志术包含查询数据的SQL语句。默认情况下，查询日志是未开启的。如果需要开启查询日志，

可以设置以下配置∶show variables like '%general%';

修改MySQL的配置文件/etc/my.cnf文件，添加如下内容:

#该选项用来开启查询日志，可选值:0或者1;0代表关闭，1代表开启general\_log=1

<a name="8fd3-1699302083336"></a>#设置日志的文件名，如果没有指定，默认的文件名为host\_name.loggeneral\_log\_file=mysql\_query.log
### <a name="hg9x-1699299225236"></a><a name="_toc12771"></a>**-慢查询日志(默认不记录非索引查询)**
慢查询日志记录了所有执行时间超过参数long\_query\_time设置值并且扫描记录数不小于min\_examined\_row\_limit的所有的SQL语句的日志，

默认未开启。long\_query\_time默认为10秒，最小为0，精度可以到微秒。

#慢查询日志 slow\_query\_log=1   -- 1=开启

#执行时间参数 long\_query\_time=2   -- 2s

默认情况下，不会记录管理语句，也不会记录不使用索引进行查找的查询。可以使用log\_slow\_admin\_statements和更改此行为

log\_queries\_not\_using\_indexes，如下所述。

#记录执行较慢的管理语句 log\_slow\_admin\_statements =1

<a name="qitj-1699299211824"></a>#记录执行较慢的未使用索引的语句 log\_queries\_not\_using \_indexes = 1

<a name="qhvr-1699299111244"></a>**主从复制**

<a name="weey-1699299116447"></a>**分库分表**

<a name="n1dc-1699299121860"></a>**读写分离**
