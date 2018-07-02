# mysql 分组汇总（WITH ROLLUP使用）
### 基础数据：
```sql
SELECT 'A' AS type,'D' AS owner,435 AS num1,234 AS num2 UNION ALL
SELECT 'A' AS type,'E' AS owner,123 AS num1,111 AS num2 UNION ALL
SELECT 'B' AS type,'D' AS owner,873 AS num1,984 AS num2 UNION ALL
SELECT 'B' AS type,'E' AS owner,486 AS num1,435 AS num2 UNION ALL
SELECT 'C' AS type,'D' AS owner,956 AS num1,723 AS num2 UNION ALL
SELECT 'C' AS type,'E' AS owner,765 AS num1,324 AS num2

type  owner num1  num2
A	  D	    435	  234
A	  E	    123	  111
B	  D	    873	  984
B	  E	    486	  435
C	  D	    956	  723
C	  E	    765	  324
```
### 期望加工后的结果：
```sql
type    owner   num1  num2 total
A	  D	    435	  234   0
A	  E	    123	  111   0
A     小计  312  123   435
B	  D	    873	  984   0
B	  E	    486	  435   0
B     Z小计 387  549   936
C	  D	    956	  723   0
C	  E	    765	  324   0
C     Z小计 191  399   590
```
### 查询语句：
```sql
SELECT
  t.type AS type,
  CASE WHEN t.type='A' AND t.owner='Z小计' THEN '小计' ELSE t.owner END AS owner,
  CASE WHEN t.owner='E' THEN -t.num1 ELSE t.num1 END AS num1,
  CASE WHEN t.owner='E' THEN -t.num2 ELSE t.num2 END AS num2,
  SUM(CASE WHEN t.owner='Z小计' THEN t.num1+t.num2 ELSE 0 END) AS total
  FROM
    (SELECT t.type,
    CASE WHEN t.owner <=> NULL THEN 'Z小计' ELSE t.owner END AS owner,
    SUM(CASE WHEN t.owner='E' THEN -t.num1 ELSE t.num1 END) AS num1,
    SUM(CASE WHEN t.owner='E' THEN -t.num2 ELSE t.num2 END) AS num2
    FROM
    (SELECT 'A' AS type,'D' AS owner,435 AS num1,234 AS num2 UNION ALL
    SELECT 'A' AS type,'E' AS owner,123 AS num1,111 AS num2 UNION ALL
    SELECT 'B' AS type,'D' AS owner,873 AS num1,984 AS num2 UNION ALL
    SELECT 'B' AS type,'E' AS owner,486 AS num1,435 AS num2 UNION ALL
    SELECT 'C' AS type,'D' AS owner,956 AS num1,723 AS num2 UNION ALL
    SELECT 'C' AS type,'E' AS owner,765 AS num1,324 AS num2) t-- 基础数据
    GROUP BY t.type,t.owner WITH ROLLUP) t-- 分组汇总
WHERE t.type IS NOT NULL
GROUP BY t.type,t.owner,t.num1,t.num2-- 查询结果修正
```