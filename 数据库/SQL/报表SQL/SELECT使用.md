# SQL SELECT使用

#### 使用GROUP BY可以实现行分组，SQL中可以使用聚合函数配合CASE WHEN实现将行数据进行分类统计
```sql
SELECT
    v.occurDate AS occurDate,
    COUNT(1) AS 总计,
    SUM(CASE WHEN v.value='value1' THEN 1 ELSE 0 END) AS value1,
    SUM(CASE WHEN v.value='value2' THEN 1 ELSE 0 END) AS value2,
    SUM(CASE WHEN v.value='value3' THEN 1 ELSE 0 END) AS value3,
    SUM(CASE WHEN v.value='value4' THEN 1 ELSE 0 END) AS value4
FROM
(SELECT '2018-04-20' AS occurDate,'value1' AS value
UNION ALL
SELECT '2018-04-20','value2'
UNION ALL
SELECT '2018-04-20','value3'
UNION ALL
SELECT '2018-04-20','value4'
UNION ALL
SELECT '2018-04-19','value2'
UNION ALL
SELECT '2018-04-18','value4'
UNION ALL
SELECT '2018-04-17','value3'
UNION ALL
SELECT '2018-04-19','value2'
UNION ALL
SELECT '2018-04-19','value2'
UNION ALL
SELECT '2018-04-18','value1'
) v
GROUP BY v.occurDate
```

#### 按照名称分组时，由于字符串比较是从第一个字符开始往后比较，所以如果长度不一致，排序结果跟我们期望的不一致，此时可以使用LENGTH函数
```sql
SELECT
  name
FROM
(SELECT '券10' AS name
UNION ALL
SELECT '券20'
UNION ALL
SELECT '券50'
UNION ALL
SELECT '券100'
UNION ALL
SELECT '券130'
UNION ALL
SELECT '券200'
UNION ALL
SELECT '券500'
) v
ORDER BY LENGTH(name),name
```

#### 此处提供一个MySQL的分组函数，主要将数据汇总为按照指定字符分隔的字符串
```sql
SELECT
  GROUP_CONCAT(vv.code SEPARATOR ',') AS value,
  vv.text AS text
FROM
(SELECT v.code AS code,
        CASE WHEN v.code IN('券10','券20','券50')
              THEN '低额券'
             ELSE '高额券'
        END AS text
FROM
(SELECT '券10' AS code,'券10' AS name
UNION ALL
SELECT '券20','券20'
UNION ALL
SELECT '券50','券50'
UNION ALL
SELECT '券100','券100'
UNION ALL
SELECT '券130','券130'
UNION ALL
SELECT '券200','券200'
UNION ALL
SELECT '券500','券500'
)v
) vv
GROUP BY vv.text
```

#### SQL性能对比
```sql
CREATE TABLE cardType(
code varchar(32) not null comment '卡类型代码',
name varchar(100) not null comment '卡类型名称',
primary key(code)
)ENGINE=InnoDB default CHARSET=utf8 COLLATE=utf8_bin

INSERT INTO cardType(code,name)
VALUES('IC卡','IC卡'),
('充值卡10','充值卡10'),
('充值卡20','充值卡20'),
('充值卡30','充值卡30'),
('充值卡50','充值卡50'),
('券100','券100'),
('券200','券200'),
('券500','券500'),
('券800','券800'),
('消费卡50','消费卡50'),
('消费卡80','消费卡80'),
('消费卡100','消费卡100'),
('其他卡类型01','其他卡类型01'),
('其他卡类型02','其他卡类型02'),
('其他卡类型03','其他卡类型03'),
('其他卡类型04','其他卡类型04'),
('其他卡类型05','其他卡类型05'),
('其他卡类型06','其他卡类型06'),
('其他卡类型07','其他卡类型07'),
('其他卡类型08','其他卡类型08'),
('其他卡类型09','其他卡类型09'),
('其他卡类型10','其他卡类型10'),
('其他卡类型11','其他卡类型11'),
('其他卡类型12','其他卡类型12'),
('其他卡类型13','其他卡类型13'),
('其他卡类型14','其他卡类型14'),
('其他卡类型15','其他卡类型15'),
('其他卡类型16','其他卡类型16'),
('其他卡类型17','其他卡类型17'),
('其他卡类型18','其他卡类型18'),
('其他卡类型19','其他卡类型19'),
('其他卡类型20','其他卡类型20'),
('其他卡类型21','其他卡类型21'),
('其他卡类型22','其他卡类型22'),
('其他卡类型23','其他卡类型23'),
('其他卡类型24','其他卡类型24'),
('其他卡类型25','其他卡类型25'),
('其他卡类型26','其他卡类型26'),
('其他卡类型27','其他卡类型27'),
('其他卡类型28','其他卡类型28'),
('其他卡类型29','其他卡类型29'),
('其他卡类型30','其他卡类型30'),
('其他卡类型31','其他卡类型31'),
('其他卡类型32','其他卡类型32'),
('其他卡类型33','其他卡类型33')

-- 性能比较：
-- 30ms+
SELECT code,name FROM cardType
WHERE code LIKE '%充值卡%' OR code LIKE '%券%' OR code LIKE '%消费卡%'

-- 25ms-
SELECT
  v.code,v.name
FROM
(SELECT CASE WHEN code LIKE '%充值卡%' OR code LIKE '%券%' OR code LIKE '%消费卡%' THEN code ELSE '' END AS code,name
FROM cardType
) v
WHERE v.code<>''

-- 分析：
猜测第二种写法磁盘IO次数少，v为中间表，数据全部在内存中，计算较快；第一种写法全表扫描，多次磁盘IO降低了性能。
```

#### MySQL分组汇总函数使用，主要对分组结果差值做统计（将某一行转为负值统计后转回正值）
```sql
SELECT
    l.a,l.b,CASE WHEN l.b='业务' THEN -l.num ELSE l.num END AS num
FROM(
SELECT
    l.a AS a,
    CASE WHEN l.b <=> NULL THEN 'Z小计' ELSE l.b END AS b,
    SUM(CASE WHEN l.b='业务' THEN -l.num ELSE l.num END) AS num
from (
SELECT 'w' AS a,'账务' AS b,123 AS num UNION ALL
SELECT 'w' AS a,'业务' AS b,3245 AS num UNION ALL
SELECT 'w' AS a,'账务' AS b,549 AS num UNION ALL
SELECT 'b' AS a,'账务' AS b,677 AS num UNION ALL
SELECT 'b' AS a,'业务' AS b,846 AS num UNION ALL
SELECT 'k' AS a,'账务' AS b,596 AS num UNION ALL
SELECT 'k' AS a,'业务' AS b,957 AS num UNION ALL
SELECT 'k' AS a,'账务' AS b,457 AS num) l
GROUP BY l.a,l.b WITH ROLLUP
)l
WHERE l.a IS NOT NULL
ORDER BY l.a ASC,l.b DESC
```


#### pasoReport报表参数下拉框(用户选择框)数据设置
在制作报表时，一些入参是提供给用户选择的，pasoReport除了提供用户输入，还提供了SQL查询和树形列表；本文章讲解SQL查询的更高级使用方法。

假设报表有入参卡类型，但是此条件是变相多选，用户有两个选择，“IC卡”和“消费卡”：
IC卡：code LIKE '%IC%'
消费卡：code LIKE '%消费%'
按照pasoReport的要求，我们应该从数据库中按条件查询卡类型表，并且分组为两种类型供用户选择。
这里我就直接贴SQL（cardType表有code,name字段）：
```sql
SELECT GROUP_CONCAT(v.code SEPARATOR ',') AS value,name AS text FROM
(SELECT code,CASE WHEN code LIKE '%IC%' THEN 'IC卡' WHEN code LIKE '%消费%' THEN '消费卡' ELSE '' END AS name FROM cardType)
v WHERE v.name <> ''
GROUP BY v.name
```

最后需要在报表参数设计时，卡类型参数的操作符选择“在之中”

### MySQL 开窗函数（行数据统计）
开窗函数主要功能是实现行数据的统计，但是MySQL本身不支持开窗函数，这里是网上找来的一些替代方法。主要是相邻行间数据的计算。
---------------------------
开窗函数非SQL标准，所以具体使用需要查看使用数据库的手册，找对应的实现方法。 
---------------------------
```sql
-- 假设我们需要查看一个账户的流水，按照时间升序排，有个需求，需要将异常流水标记出来（本条流水中记录的余额与上条流水的交易后余额不一致）。
-- 我们这里用flag字段为1标记异常流水。

-- MySQL查询方法：
-- 不推荐使用，在同一select中使用用户变量，执行顺序并不能保证按照预期执行。
-- 这里还有个问题，@tmp的初始值不定，所以第一条数据flag=1
SELECT
  fildate,
  recordBalance,
  CASE WHEN @tmp <> recordBalance THEN 1 ELSE 0 END AS flag,
  @tmp:=balance as balance
FROM
(SELECT '2018-04-17' AS fildate,123 AS recordBalance,456 AS balance UNION ALL
SELECT '2018-04-18',356,234 UNION ALL -- 异常流水 this.recordBalance <> last.balance
SELECT '2018-04-19',234,666 UNION ALL
SELECT '2018-04-20',666,123
) v
```

### 在查询中使用SELECT查询，SELECT查询多行导致报错

```sql
-- 有些时候，使用where条件把数据过滤好了，在处理返回结果时，需要另外查询一个数据，当然这个查询一般指返回一个字段。
-- 在SQL执行过程中，如果这个“另外一个查询”返回一行数据，那么SQL可以正常执行；如果返回多行，那么SQL会报错，表明查询返回了多行数据，这完全不是我们想要的结果。
-- 那么有什么办法避免返回多行么？当然是有的，两种方案：1、如果关心“另外一个查询”的结果，那么需要再加一个查询；2、不关心结果(即存在性验证)，那么调整SQL即可。

-- 第一种情况：（这里只写如何控制只返回一行数据：）
SELECT * FROM tab t WHERE ... AND t.created = (SELECT MAX(created) FROM tab WHERE ...) -- "..."的条件相同，这里我只要created最新的一条数据。

-- 第二中情况
SELECT COUNT(1)/MAX(1)/MIN(1)/... FROM tab WHERE ...

```

### 对于核销，冲账，再次核销的数据合并展示
```sql
-- 项目中总会遇到一些奇葩需求，比如这次的，一张券，由于前台核销时超时，然后执行了撤销，再次核销，成功了。
-- 那么以上过程，最起码会有三条流水产生，报表要求对于上述记录合并为一条展示，需要对金额字段修改为计算式，类似于“100 = 100 - 100 + 100”。。。
-- 初看这个要求，一般都会很抵触，感觉是不可能做出来的，不过钻研了一下，还是给项目上做出来了，主要使用了MySQL的group_concat函数，这里做下记录，分享出来。
-- 简化数据集：
|ID |券号|   交易时间|    发生金额(撤销交易为负)|
|---:|:---|:---|:--|
|1|1|2018-07-11 09:00:00|100|
|2|1|2018-07-11 09:00:26|-100|
|3|2|2018-07-11 09:01:12|200|
|4|1|2018-07-11 09:01:15|100|
-- 需要将以上三条数据转为两条交易数据：
|券号|交易时间|发生金额|发生金额明细|
|:---|:---|:---|:---|
|1|2018-07-11 09:01:15|100|100 = 100 - 100 + 100|
|2|2018-07-11 09:01:12|200|200|

-- SQL：
SELECT
	vv.voucherid AS 券号,
	vv.fildate AS 交易时间,
	vv.occur AS 发生金额,
	CASE WHEN vv.num = 1 THEN CONCAT(vv.occur,'') ELSE CONCAT(vv.occur,' = ',vv.occurStr) END AS 发生金额明细
FROM (
SELECT v.voucherid,MAX(v.fildate) AS fildate,COUNT(*) AS num,SUM(v.occur) AS occur,REPLACE(GROUP_CONCAT(v.occur ORDER BY v.fildate SEPARATOR ' + '),'+ -','- ') AS occurStr
FROM
(SELECT 1 AS id,'1' AS voucherid, '2018-07-11 09:00:00' AS fildate,100 AS occur UNION ALL
SELECT 2,'1','2018-07-11 09:00:26',-100 UNION ALL
SELECT 3,'2','2018-07-11 09:01:12',200 UNION ALL
SELECT 4,'1','2018-07-11 09:01:15',100
)v
GROUP BY v.voucherid
) vv

-- 扩展：
-- 查询如果需要数据合并，一般都是需要对数据进行分组，将同组数据进行合并，我们需要了解较多的聚合函数，不能局限于常用的count,sum,min,max,avg...
```