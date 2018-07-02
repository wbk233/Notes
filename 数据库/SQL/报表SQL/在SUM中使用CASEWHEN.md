SQL在SUM函数参数中使用CASE WHEN实现伪二维报表
=============================
这里使用伪二维报表是因为报表结果列对应了SELECT中的值，所以实际报表的最终列是固定值
-------------------------
# 目前有流水表（简化）：
```sql
storeId,
cardType,-- 取值一共N种这里假设3种
amount
```
# 数据有：
```sql
storeId        cardType    amount
门店1        卡类型2        100
门店1        卡类型3        133
门店2        卡类型1        666
门店3        卡类型1        222
门店3        卡类型2        222
门店3        卡类型3        444
.            .            .
.            .            .
.            .            .
```
# 需要查询如下格式报表：
```sql
门店    总金额    卡类型1总金额    卡类型2总金额    卡类型3总金额
门店1    233        0                100                133
门店2    666        666                0                0
门店3    888        222                222                444
.        .        .                .                .
.        .        .                .                .
.        .        .                .                .
```
# 具体查询SQL：
```sql
SELECT
    storeId    AS 门店,
    SUM(v.amount) AS 总金额,
    SUM(CASE WHEN v.cardType = '卡类型1' THEN v.amount ELSE 0 END) AS 卡类型1总金额,
    SUM(CASE WHEN v.cardType = '卡类型2' THEN v.amount ELSE 0 END) AS 卡类型2总金额,
    SUM(CASE WHEN v.cardType = '卡类型3' THEN v.amount ELSE 0 END) AS 卡类型3总金额
FROM(
    SELECT storeId,cardType,SUM(amount) AS amount
    FROM hst
    GROUP BY storeId,cardType)v
GROUP BY v.storeId
```
# 另附可以在MYSQL中执行的测试SQL(其中vv视图为上述准备数据)：
```sql
SELECT
    storeId    AS 门店,
    SUM(v.amount) AS 总金额,
    SUM(CASE WHEN v.cardType = '卡类型1' THEN v.amount ELSE 0 END) AS 卡类型1总金额,
    SUM(CASE WHEN v.cardType = '卡类型2' THEN v.amount ELSE 0 END) AS 卡类型2总金额,
    SUM(CASE WHEN v.cardType = '卡类型3' THEN v.amount ELSE 0 END) AS 卡类型3总金额
FROM(
    SELECT vv.storeId,vv.cardType,SUM(vv.amount) AS amount
    FROM(
        SELECT '门店1' AS storeId,'卡类型2' AS cardType,100 AS amount UNION
        SELECT '门店1' AS storeId,'卡类型3' AS cardType,133 AS amount UNION
        SELECT '门店2' AS storeId,'卡类型1' AS cardType,666 AS amount UNION
        SELECT '门店3' AS storeId,'卡类型1' AS cardType,222 AS amount UNION
        SELECT '门店3' AS storeId,'卡类型2' AS cardType,222 AS amount UNION
        SELECT '门店3' AS storeId,'卡类型3' AS cardType,444 AS amount
    )vv
    GROUP BY vv.storeId,vv.cardType)v
GROUP BY v.storeId
```