
# Sugo PlyQL

 - [简介](#intro)
 - [示例](#example)
 - [操作符](#operators)
 - [函数](#functions)
 - [聚合](#aggregations)
 - [JDBC 驱动文档](/developer/interfaces/jdbc.md)

---

<a id="intro" href="intro"></a>
plyql是类似SQL的语言，可以解析成sugo-plywood的表达并执行。

plyql目前支持`SELECT`，`DESCRIBE`，和`SHOW TABLES`查询。

---

### 启用服务方式

  - [终端命令模式](#dos)
  - [HTTP REST API模式](#rest)
  - [JDBC驱动模式](/developer/interfaces/jdbc.md)

---

### <a id="example" href="#example"></a> 示例
---

  > 以下使用实例以`终端命令模式`为例子，其他模式的SQL调用一样

  > 对于这些示例，我们查询一个Druid代理节点IP为`192.168.60.100`，端口为`8082`

  > 我们将地址（192.168.60.100:8082）传递给`--host`（`-h`）选项。

  > 将所有的SQL比如`SHOW TABLES`，`DESCRIBE TABLE_NAME`，`SELECT COUNT(*) FROM TABLE_NAME`传递给`--query`（`-q`）选项。

  - [查询所有数据源列表](#show-tables)
  - [查询数据源列定义结构](#desc)
  - [SQL查询调用](#query)
    - [获取最大时间](#query-max)
    - [综合使用SQL查询](#query-group-by)
    - [QUANTILE函数](#QUANTILE)
    - [高级查询](#adv-query)

### <a id="dos" href="#dos"></a> 终端命令模式

#### <a id="show-tables" href="show-tables"></a> 查询所有数据源列表

> 查询当前节点所有数据源列表的例子：


```sql
plyql -h 192.168.60.100:8082 -q 'SHOW TABLES'
```

返回

```
┌────────────────────────────────────┐
│ Tables_in_database                 │
├────────────────────────────────────┤
│ COLUMNS                            │
│ SCHEMATA                           │
│ TABLES                             │
│ sugo_test1                         │
│ sugo_test2                         │
│ sugo                               │
└────────────────────────────────────┘
```

#### <a id="desc" href="desc"></a> 查看数据源列定义结构

```sql
plyql -h 192.168.60.100:8082 -q 'DESCRIBE sugo'
```

返回数据源的列定义：

```
┌──────────────┬────────┬──────┬─────┬─────────┬───────┐
│ Field        │ Type   │ Null │ Key │ Default │ Extra │
├──────────────┼────────┼──────┼─────┼─────────┼───────┤
│ HEAD_ARGS    │ STRING │ YES  │     │ NULL    │       │
│ HEAD_IP      │ STRING │ YES  │     │ NULL    │       │
│ __time       │ TIME   │ YES  │     │ NULL    │       │
│ app_id       │ STRING │ YES  │     │ NULL    │       │
│ duration     │ NUMBER │ YES  │     │ NULL    │       │
│ is_active    │ STRING │ YES  │     │ NULL    │       │
│ is_installed │ STRING │ YES  │     │ NULL    │       │
│ client_time  │ TIME   │ YES  │     │ NULL    │       │
└──────────────┴────────┴──────┴─────┴─────────┴───────┘
```

<a id="query" href="query"></a>

#### <a id="query-max" href="query-max"></a> 获取最大时间

这里是一个简单的查询，获取最大的` __time `信息。此查询显示数据库中最新事件的时间。

```sql
plyql -h 192.168.60.100:8082 -q 'SELECT MAX(__time) AS maxTime FROM sugo'
```

返回:

```
┌──────────────────────┐
│ maxTime              │
├──────────────────────┤
│ 2016-09-19T06:36:43Z │
└──────────────────────┘
```

#### <a id="query-group-by" href="query-group-by"></a> 综合使用SQL查询

```sql
plyql -h 192.168.60.100:8082 -q '
SELECT page as pg, 
COUNT() as cnt 
FROM sugo 
GROUP BY page 
ORDER BY cnt DESC 
LIMIT 5;
'
```

上面的查询会存在一个问题，因为没有指定plyql查询的时间过滤范围。

这种行为可以禁用使用`--allow eternity`，但不推荐这样做，这样做时，当数据量过大时，它可以发出计算禁止查询。

再试一次，用一个时间过滤器：
  
```sql
plyql -h 192.168.60.100:8082 -q '
SELECT page as pg, 
COUNT() as cnt 
FROM wikipedia 
WHERE "2015-09-12T00:00:00" <= __time AND __time < "2015-09-13T00:00:00"
GROUP BY page 
ORDER BY cnt DESC 
LIMIT 5;
'
```

结果集:
  
```
┌─────────────────────┬─────┐
│ pg                  │ cnt │
├─────────────────────┼─────┤
│ Jeremy Corbyn       │ 2   │
│ Jeremy 111111       │ 2   │
│ Jeremy 222222       │ 2   │
│ Jeremy 333333       │ 2   │
└─────────────────────┴─────┘
```

##### --interval 参数

Plyql有一个选项 `--interval` (`-i`) 自动过滤器加上`interval`间隔时间。

如果您不想键入时间筛选器，则这样设置非常有用。

```sql
plyql -h 192.168.60.100:8082 -i P1Y -q '
SELECT page as pg, 
COUNT() as cnt 
FROM wikipedia 
GROUP BY page 
ORDER BY cnt DESC 
LIMIT 5;
'
```

##### 时间分组

通过`TIME_BUCKET`函数可以对时间分解

```sql
plyql -h 192.168.60.100:8082 -i P1Y -q '
SELECT SUM(added) as TotalAdded 
FROM wikipedia 
GROUP BY TIME_BUCKET(__time, PT6H, "Asia/Shanghai");
'
```

返回:

```json
[
  {
    "TotalAdded": 15426936,
    "split0": {
      "start": "2015-09-12T00:00:00.000Z",
      "end": "2015-09-12T06:00:00.000Z",
      "type": "TIME_RANGE"
    }
  },
  {
    "TotalAdded": 25996165,
    "split0": {
      "start": "2015-09-12T06:00:00.000Z",
      "end": "2015-09-12T12:00:00.000Z",
      "type": "TIME_RANGE"
    }
  },
  "... 结果省略了 ..."
]
```

注意分组列如果没有选择但仍有返回, 列如`TIME_BUCKET(__time, PT1H, 'Asia/Shanghai') as 'split'`
是查询列中的一个。
时间分割也支持，这里是一个例子：
```sql
plyql -h 192.168.60.100:8082 -i P1Y -q '
SELECT TIME_PART(__time, HOUR_OF_DAY, "Asia/Shanghai") as HourOfDay, 
SUM(added) as TotalAdded 
FROM wikipedia 
GROUP BY 1 
ORDER BY TotalAdded DESC LIMIT 3;
'
```
注意，这个 `GROUP BY` 指的是选择中的第一列。
这将返回：
```json
[
  {
    "TotalAdded": 8077302,
    "HourOfDay": 10
  },
  {
    "TotalAdded": 5998730,
    "HourOfDay": 17
  },
  {
    "TotalAdded": 5210222,
    "HourOfDay": 18
  }
]
```
#### QUANTILE函数

支持对直方图的分位数。

假设您想要使用直方图计算 0.95 分位数的三角洲筛选的城市是旧金山。

```sql
plyql -h 192.168.60.100:8082 -i P1Y -q '
SELECT 
QUANTILE(delta_hist WHERE cityName = "San Francisco", 0.95) as P95 
FROM wikipedia;
'
```

它也是可能做到多维度分组查询的

```sql
plyql -h 192.168.60.100:8082 -i P1Y -q '
SELECT TIME_BUCKET(__time, PT1H, "Asia/Shanghai") as Hour, 
page as PageName, 
SUM(added) as TotalAdded 
FROM wikipedia 
GROUP BY 1, 2 
ORDER BY TotalAdded DESC 
LIMIT 3;
'
```

返回:

```json
[
  {
    "TotalAdded": 242211,
    "PageName": "Wikipedia‐ノート:即時削除の方針/過去ログ16",
    "Hour": {
      "start": "2015-09-12T15:00:00.000Z",
      "end": "2015-09-12T16:00:00.000Z",
      "type": "TIME_RANGE"
    }
  },
  {
    "TotalAdded": 232941,
    "PageName": "Користувач:SuomynonA666/Заготовка",
    "Hour": {
      "start": "2015-09-12T14:00:00.000Z",
      "end": "2015-09-12T15:00:00.000Z",
      "type": "TIME_RANGE"
    }
  },
  {
    "TotalAdded": 214017,
    "PageName": "User talk:Estela.rs",
    "Hour": {
      "start": "2015-09-12T12:00:00.000Z",
      "end": "2015-09-12T13:00:00.000Z",
      "type": "TIME_RANGE"
    }
  }
]
```
#### <a id="adv-query" href="adv-query"></a> 高级查询

这里是一个高级的示例，获取前 5 页编辑时间。
PlyQL 的一个显著特征是它对待 （aka 表） 的数据集作为可以嵌套在另一个表的只是另一种数据类型。
这使我们能够嵌套查询数据像这样︰
```sql
plyql -h 192.168.60.100:8082 -i P1Y -q '
SELECT page as Page, 
COUNT() as cnt, 
(
  SELECT 
  SUM(added) as TotalAdded 
  GROUP BY TIME_BUCKET(__time, PT1H, "Asia/Shanghai") 
  LIMIT 3 -- only get the first 3 hours to keep this example output small
) as "ByTime" 
FROM wikipedia 
GROUP BY page 
ORDER BY cnt DESC 
LIMIT 5;
'
```

返回:

```json
[
  {
    "cnt": 314,
    "Page": "Jeremy Corbyn",
    "ByTime": [
      {
        "TotalAdded": 1075,
        "split0": {
          "start": "2015-09-12T01:00:00.000Z",
          "end": "2015-09-12T02:00:00.000Z",
          "type": "TIME_RANGE"
        }
      },
      {
        "TotalAdded": 0,
        "split0": {
          "start": "2015-09-12T07:00:00.000Z",
          "end": "2015-09-12T08:00:00.000Z",
          "type": "TIME_RANGE"
        }
      },
      {
        "TotalAdded": 10553,
        "split0": {
          "start": "2015-09-12T08:00:00.000Z",
          "end": "2015-09-12T09:00:00.000Z",
          "type": "TIME_RANGE"
        }
      }
    ]
  },
  {
    "cnt": 255,
    "Page": "User:Cyde/List of candidates for speedy deletion/Subpage",
    "ByTime": [
      {
        "TotalAdded": 73,
        "split0": {
          "start": "2015-09-12T00:00:00.000Z",
          "end": "2015-09-12T01:00:00.000Z",
          "type": "TIME_RANGE"
        }
      },
      {
        "TotalAdded": 3363,
        "split0": {
          "start": "2015-09-12T01:00:00.000Z",
          "end": "2015-09-12T02:00:00.000Z",
          "type": "TIME_RANGE"
        }
      },
      {
        "TotalAdded": 336,
        "split0": {
          "start": "2015-09-12T02:00:00.000Z",
          "end": "2015-09-12T03:00:00.000Z",
          "type": "TIME_RANGE"
        }
      }
    ]
  },
  "... results omitted ..."
]
```

## <a id="operators" href=“operators”></a> 运算符

名称                    | 描述
------------------------|-------------------------------------
+                       | 加法运算符
-                       | 减号运算符 / 一元求反
*                       | 乘法运算符
/                       | 除法运算符
=, IS                   | 等于运算符
!=, <>, IS NOT          | 不等于运算符
>=                      | 大于或等于运算符
>                       | 大于运算符
<=                      | 小于或等于运算符
<                       | 小于运算符
BETWEEN ... AND ...     | 检查一个值是否在值范围内
NOT BETWEEN ... AND ... | 检查一个值是否不在值范围内
LIKE, CONTAINS          | 简单的模式(模糊)匹配
NOT LIKE, CONTAINS      | 否定的简单模式(模糊)匹配
REGEXP                  | 模式匹配使用正则表达式
NOT REGEXP              | 正则表达式的否定
NOT, !                  | 取否
AND                     | 逻辑并
OR                      | 逻辑或

## <a id="functions" href="functions"></a> 函数

<a id="TIME_BUCKET" href="#TIME_BUCKET">#</a>
**TIME_BUCKET**(operand, duration, timezone)

将时间在给定的`timezone(时区)`中按给定的`duration(粒度)`查询

示例: `TIME_BUCKET(time, 'P1D', 'Asia/Shanghai')`

这将把`time`变量装入日期块，其中按`Asia/Shanghai`时区的天粒度定义。

<a id="TIME_PART" href="#TIME_PART">#</a>
**TIME_PART**(operand, part, timezone)

按指定的时区获取对应参数的时间参数值

例如: `TIME_PART(time, 'DAY_OF_YEAR', 'Asia/Shanghai')`
这将把'time`变量分成（整数）数字，表示一年中的哪一天。
可能的部分值是:

* `SECOND_OF_MINUTE`, `SECOND_OF_HOUR`, `SECOND_OF_DAY`, `SECOND_OF_WEEK`, `SECOND_OF_MONTH`, `SECOND_OF_YEAR`
* `MINUTE_OF_HOUR`, `MINUTE_OF_DAY`, `MINUTE_OF_WEEK`, `MINUTE_OF_MONTH`, `MINUTE_OF_YEAR`
* `HOUR_OF_DAY`, `HOUR_OF_WEEK`, `HOUR_OF_MONTH`, `HOUR_OF_YEAR`
* `DAY_OF_WEEK`, `DAY_OF_MONTH`, `DAY_OF_YEAR`
* `WEEK_OF_MONTH`, `WEEK_OF_YEAR`
* `MONTH_OF_YEAR`
* `YEAR`


<a id="TIME_FLOOR" href="#TIME_FLOOR">#</a>
**TIME_FLOOR**(operand, duration, timezone)

在给定的`timezone(时区)`中将时间置于最近的`duration(粒度)`。
示例: `TIME_FLOOR(time, 'P1D', 'Asia/Shanghai')`

这将把`time`变量置于一天的开始，其中天在`Asia/Shanghai`时区中定义。

<a id="TIME_SHIFT" href="#TIME_SHIFT">#</a>
**TIME_SHIFT**(operand, duration, step, timezone)

在给定的`timezone(时区)`中，通过`duration(粒度)` * `step`向前移动时间。
`step`可能是负数。

示例: `TIME_SHIFT(time, 'P1D', -2, 'Asia/Shanghai')`

这将在两天后移动`time`变量，其中天在`Asia/Shanghai`时区中定义。

<a id="TIME_RANGE" href="#TIME_RANGE">#</a>
**TIME_RANGE**(operand, duration, step, timezone)

在给定的`timezone(时区)`中创建一个范围形式`time`和一个`duration` *`step`远离`time`的点。
`step`可能是负数。

示例: `TIME_RANGE(time, 'P1D', -2, 'Asia/Shanghai')`

这将在两天后移动`time`变量，其中天在`Asia/Shanghai`时区中定义，并创建一个时间-2 * P1D - >时间范围。

<a id="SUBSTR" href="#SUBSTR">#</a>
**SUBSTR**(*str*, *pos*, *len*)

从字符串*str*返回一个子串*len*个字符，从位置*pos*开始

<a id="CONCAT" href="#CONCAT">#</a>
**CONCAT**(*str1*, *str2*, ...)

返回由连接参数产生的字符串。 可以有一个或多个参数

<a id="EXTRACT" href="#EXTRACT">#</a>
**EXTRACT**(*str*, *regexp*)

返回第一个匹配的组，结果窗体匹配 *regexp* 到 *str*

<a id="LOOKUP" href="#LOOKUP">#</a>
**LOOKUP**(*str*, *lookup-namespace*)

返回键的值 *str* 内 *查找-命名空间*

<a id="IFNULL" href="#IFNULL">#</a>
**IFNULL**(*expr1*, *expr2*)

返回 *expr* 如果不为空，否则返回 *expr2*

<a id="FALLBACK" href="#FALLBACK">#</a>
**FALLBACK**(*expr1*, *expr2*)

这个等同于 **IFNULL**(*expr1*, *expr2*)

<a id="OVERLAP" href="#OVERLAP">#</a>
**OVERLAP**(*expr1*, *expr2*)

检查 *expr1* 和 *expr2* 是否重叠

### 数学函数

<a id="ABS" href="#ABS">#</a>
**ABS**(*expr*)

返回*expr*的绝对值。

<a id="POW" href="#POW">#</a>
**POW**(*expr1*, *expr2*)

返回 *expr1* 的幂 *expr2*。

<a id="POWER" href="#POWER">#</a>
**POWER**(*expr1*, *expr2*)

这个等同于 **POW**(*expr1*, *expr2*)

<a id="SQRT" href="#SQRT">#</a>
**SQRT**(*expr*)

返回*expr*的平方根

<a id="EXP" href="#EXP">#</a>
**EXP**(*expr*)

返回 e（自然对数的基数） 的值*expr*的幂。



## <a id="aggregations" href="aggregations"></a> 聚合函数

<a id="COUNT" href="#COUNT">#</a>
**COUNT**(*expr?*)

如果在没有表达式的情况下使用（或者作为`COUNT(*)`）返回行数的计数。
当提供表达式时，返回*expr*不为null的行的计数。

<a id="COUNT" href="#COUNT">#</a>
**COUNT**(**DISTINCT** *expr*), **COUNT_DISTINCT**(*expr*) 

返回具有不同*expr*值的行数的计数。

<a id="SUM" href="#SUM">#</a>
**SUM**(*expr*) 

返回所有*expr*值的总和。

<a id="MIN" href="#MIN">#</a>
**MIN**(*expr*) 

返回所有*expr*值的最小值。

<a id="MAX" href="#MAX">#</a>
**MAX**(*expr*) 

返回所有*expr*值的最大值。

<a id="AVG" href="#AVG">#</a>
**AVG**(*expr*)

返回所有*expr*值的平均值。

<a id="QUANTILE" href="#QUANTILE">#</a>
**QUANTILE**(*expr*, *quantile*)

返回所有*expr*值的上层*分位数*。

<a id="CUSTOM" href="#CUSTOM">#</a>
**CUSTOM**(*custom_name*)

返回名为*custom_name*的用户定义聚合。

## <a id="rest" href="#rest"></a> HTTP REST API模式

#### 启动HTTP REST APi

```
plyql -h 192.168.60.100  --json-server  8001
```

HTTP接口请求
```http
http://192.168.60.100:8001/plyql/
post请求: application/json 参数
{
  "sql":
  "SHOW TABLES"
}
```