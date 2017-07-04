---
title: sql笔记
date: 2017-06-24 15:55:11
categories:
- javaee
tags:
- sql
- javaee
---
## sql笔记

### Select---想选择什么，以及从什么地方选
#### 简单例子
* 检索单个列
  ```sql
   SELECT RealName FROM t_user
  ```

* 检索多个列
  ```sql
     SELECT RealName,Gender FROM t_user
  //在选择多个列时，一定要在列名之间加上逗号，但最后一个列名后不加。如果在最后一个列名后加了逗号，将出现错误。
  ```
* 检索所有列
  ```sql
  SELECT * FROM t_user
  ```

* 检索返回不同的值
  ```sql
  SELECT DISTINCT Ctype,Gender FROM t_user  //DISTINCT会将这个字段不重复的值返回,DISTINCT关键字作用于所有的列，不仅仅是跟在其后的那一列。例如，你指定SELECT DISTINCT Ctype, Gender，除非指定的两列完全相同，否则所有的行都会被检索出来
  ```

* 检索限制结果
  ```sql
   SELECT Gender FROM t_user LIMIT 5  OFFSET 5; //各个数据库方式不同mysql中使用方法 限制从第五个开始 长度为5 SELECT Gender FROM t_user LIMIT 5,5;
  ```

#### 排序---关系数据库设计理论认为，如果不明确规定排序顺序，则不应该假定检索出的数据的顺序有任何意义。

* 单列排序
  ```sql
   SELECT * FROM t_user ORDER BY RealName
  //在指定一条ORDER BY子句时，应该保证它是SELECT语句中最后一条子句。如果它不是最后的子句，将会出现错误消息。
  ```
* 多列排序
  ```sql
   SELECT * FROM t_user ORDER BY RealName,Gender //只有在realname有相同值之后才会比较Gender
  ```
* 升降序
  ```sql
   SELECT * FROM t_user ORDER BY RealName DESC,Gender ASC//升降序默认只针对前面的列成立 DESC降序 ASC默认升序可不写
  ```

#### where

* is Null ---在创建表时，表设计人员可以指定其中的列能否不包含值。在一个列不包含值时，称其包含空值NULL。

  ```sql
  SELECT * FROM t_user WHERE t_user.Tel IS NULL //通过过滤选择不包含指定值的所有行时，你可能希望返回含NULL值的行。但是这做不到。因为未知（unknown）有特殊的含义，数据库不知道它们是否匹配，所以在进行匹配过滤或非匹配过滤时，不会返回这些结果。
  ```

* And
  ```sql
  SELECT * FROM t_user WHERE Tel > 15504409406 AND ID < 91000 AND Gender= '男' //并列条件可以使用And连接
  ```

* or or并列时一般满足第一个条件就会返回不需在做其他操作
  ```sql
  SELECT prod_name, prod_price FROM Products WHERE (vend_id = 'DLL01' OR vend_id = 'BRS01') AND prod_price >= 10;//用圆括号括起来不会产生歧义 否则 And 会比 or优先导致结果出错
  ```
* IN 与or具有相同功能 更好的观察性 更好的速度 可以包含select语句
  ```sql
  SELECT prod_name, prod_price FROM Products WHERE vend_id IN('DLL01','BRS01') AND prod_price >= 10
  ```

#### 通配符 LIKE
  * 通配符搜索一般比前面讨论的其他搜索要耗费更长的处理时
  * 不要过度使用通配符。如果其他操作符能达到相同的目的，应该使用其他操作符。
  * 在确实需要使用通配符时，也尽量不要把它们用在搜索模式的开始处。把通配符置于开始处，搜索起来是最慢的。


* % 匹配字符类型 可以在任何位置 可以多个同时使用 但是 通配符%看起来像是可以匹配任何东西，但有个例外，这就是NULL。子句WHERE prod_name LIKE '%'不会匹配产品名称为NULL的行。
  ```sql
  SELECT * FROM t_user WHERE Tel LIKE '1550440%'
  ```

* _ 下划线 一个下划线只能匹配一个字符
  ```sql
  SELECT * FROM t_user WHERE Tel LIKE '1550440940_'
  ```

#### 计算字段

* 拼接字段
```sql
SELECT  CONCAT(Gender,'(',RealName,')') AS GR FROM t_user WHERE Tel = 15504409406
//别名 用as来引用导致输出结果可以直接用此字段使用
```

* 计算字段
```sql
SELECT LoginNum*CType AS la,LoginNum,CType FROM t_user WHERE Tel=15504409406
```

#### 函数

* 文本处理函数

  * 某字段左起几个字符
```sql
SELECT LEFT(RealName,1) AS left_name from t_user WHERE Tel = 18629863729;
```
  * 某字段右起几个字符
```sql
SELECT RIGHT(cname,2) from t_vip WHERE ID=76
```
  * 某个字段的长度
```sql
SELECT LENGTH(Tel) AS left_name from t_user
```
  * 大写变小写
```sql
SELECT LOWER(w_wk.KeyWord) from w_wk
```
  * 小写变大写
```sql
SELECT UPPER(w_wk.KeyWord) from w_wk
```
  * 去空格
```sql
SELECT RTRIM(LTRIM(RIGHT(cname,2))) from t_vip WHERE ID=76
```
  * SOUNDEX 寻找相近读音的单词
```sql
SELECT * from t_vip WHERE SOUNDEX(CName)=SOUNDEX('tu')
```

* 数值日期处理函数太多-----现用现查
* 聚集函数
  * 平均数 AVG()函数忽略列值为NULL的行。
```sql
select AVG(Price) AS avg from t_bk_school
```
  * 计数
```sql
select COUNT(*) AS avg from t_bk_school ; //包含null值的
SELECT COUNT(t_bk_school.FatherID) as o from t_bk_school; //忽略列值为NULL的行
```
  * 最大最小值 忽略null  文本数据也可以取默认取第一个和最后一个
```sql
select MIN(ID) from ac_activity  ORDER BY id DESC ;
```
  * 求和 忽略null
```sql
SELECT SUM(CType) FROM t_user
```

#### 分组数据

* group by

      GROUP BY子句可以包含任意数目的列，因而可以对分组进行嵌套，更细致地进行数据分组。 如果在GROUP BY子句中嵌套了分组，数据将在最后指定的分组上进行汇总。换句话说，在建立分组时，指定的所有列都一起计算（所以 不能从个别的列取回数据）。
      GROUP BY子句中列出的每一列都必须是检索列或有效的表达式（但不能是聚集函数）。
      如果在SELECT中使用表达式，则必须在GROUP BY子句中指定相同的表达式。不能使用别名。
      大多数SQL实现不允许GROUP BY列带有长度可变的数据类型（如文本或备注型字段）。 除聚集计算语句外，SELECT语句中的每一列都必须在GROUP BY子句中给出。
      如果分组列中包含具有NULL值的行，则NULL将作为一个分组返回。如果列中有多行NULL值，它们将分为一组。 GROUP BY子句必须出现在WHERE子句之后，ORDER BY子句之前。
```sql
SELECT CType,COUNT(*) as num from t_user group by CType
SELECT Gender,CType,COUNT(*) as num from t_user group by CType,Gender
```

* Having
```sql
SELECT CType,Gender,COUNT(*) from t_user GROUP BY Gender,CType HAVING CType<2
// HAVING 与 GROUP BY 共同使用时 子句中除了聚集函数外 使用的列必须在group by中使用
// WHERE在数据分组前进行过滤，HAVING在数据分组后进行过滤
```


#### 子查询

```sql
SELECT * FROM t_user_authority WHERE UserID = (SELECT ID FROM t_user WHERE t_user.Tel = 155044094061)
```

注意使用完全限定名
