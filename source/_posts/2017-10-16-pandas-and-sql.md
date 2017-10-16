---
title: pandas与sql的对应关系
date: 2017-10-16 16:02:04
keywords: pandas, sql, python
tags:
- python
- pandas
- sql
---

# 参考:
- [Comparison with SQL](https://pandas.pydata.org/pandas-docs/stable/comparison_with_sql.html)
- [Indexing and Selecting Data](https://pandas.pydata.org/pandas-docs/stable/indexing.html)

# pandas与sql

pandas和sql在很多操作上都是类似的，如果熟悉sql操作，也可以很方便地使用pandas

首先需要引用两个库

```
>>> import pandas as pd
>>> import numpy as np
```

本文使用的数据来源是[tips](https://raw.github.com/pandas-dev/pandas/master/pandas/tests/data/tips.csv), 假设有与之同名同结构的数据表

```
>>> url = 'https://raw.github.com/pandas-dev/pandas/master/pandas/tests/data/tips.csv'
>>> tips = pdf.read_csv(url)
```

tips的数据是这样的
```
>>> tips.head()
   total_bill   tip     sex smoker  day    time  size
0       16.99  1.01  Female     No  Sun  Dinner     2
1       10.34  1.66    Male     No  Sun  Dinner     3
2       21.01  3.50    Male     No  Sun  Dinner     3
3       23.68  3.31    Male     No  Sun  Dinner     2
4       24.59  3.61  Female     No  Sun  Dinner     4
```

下面将介绍常见sql操作与pandas操作的对应关系

<!-- more -->

# select

sql

```
select total_bill, tip, smoker, time from tips limit 10;
```

pandas

```
>>> tips[['total_bill', 'tip', 'smoker', 'time']].head(10)
   total_bill   tip     sex smoker  day    time  size
0       16.99  1.01  Female     No  Sun  Dinner     2
1       10.34  1.66    Male     No  Sun  Dinner     3
2       21.01  3.50    Male     No  Sun  Dinner     3
3       23.68  3.31    Male     No  Sun  Dinner     2
4       24.59  3.61  Female     No  Sun  Dinner     4
5       25.29  4.71    Male     No  Sun  Dinner     4
6        8.77  2.00    Male     No  Sun  Dinner     2
7       26.88  3.12    Male     No  Sun  Dinner     4
8       15.04  1.96    Male     No  Sun  Dinner     2
9       14.78  3.23    Male     No  Sun  Dinner     2
```

如果要`select * from tips`, 则对应的pandas操作就是

```
>>> tips
```

# where

## select * where

sql

```
select * from tips where time = 'Dinner' limit 10;
```

pandas

```
>>> tips[tips['time'] == 'Dinner'].head(10)
```

或者

```
>>> tips.loc[tips['time'] == 'Dinner'].head(10)
```

## select (columns) where

sql

```
select total_bill, tip, smoker, time from tips where time = 'Dinner' limit 10;
```

pandas

```
>>> tips.loc[tips['time'] == 'Dinner', ['total_bill', 'tip', 'smoker', 'time']].limit(10)
```

## select where and/or

sql
```
select * from tips where time = 'Dinner' and tip > 5.0 limit 10;
```

pandas

```
>>> tips[(tips['time'] == 'Dinner') & (tips['tip'] > 5.0)].head(10)
```

或者

```
>>> tips.loc[(tips['time'] == 'Dinner') & (tips['tip'] > 5.0)].head(10)
```

sql

```
select total_bill, tip, smoker, time from tips where time = 'Dinner' and tip > 5.0 limit 10;
```

pandas

```
>>> tips.loc[(tips['time'] == 'Dinner') & (tips['tip'] > 5.0), ['total_bill', 'tip', 'smoker', 'time']].head(10)
```

sql

```
select total_bill, tip, smoker, time from tips where time = 'Dinner' or tip > 5.0 limit 10;
```

pandas

```
>>> tips.loc[(tips['time'] == 'Dinner') | (tips['tip'] > 5.0), ['total_bill', 'tip', 'smoker', 'time']].head(10)
```

## 判断是否为null

两个函数, isnull()和notnull()

```
select total_bill, tip, smoker, time from tips where time is null and tip is not null limit 10;
```

```
>>> tips.loc[tips['time'].isnull() & tips['sex'].notnull(), ['total_bill', 'tip', 'smoker', 'time']].head(10)
```

# group by

## group by count

sql

```
select sex, time, count(*) from tips group by sex, time;
```

pandas

```
>>> tips.groupby(['sex', 'time']).size()
```

注意pandas里也有一个`count`函数，与sql中的`count`不一样。pandas的`count`函数会作用到每一列，返回每列的非空数量。

```
>>> tips.groupby(['sex', 'time']).count()
               total_bill  tip  smoker  day  size
sex    time
Female Dinner          52   52      52   52    52
       Lunch           35   35      35   35    35
Male   Dinner         124  124     124  124   124
       Lunch           33   33      33   33    33
```

## 使用多个聚合函数

```
select smoker, day, count(*), avg(tip) from tips group by smoker, day;
```

pandas

```
>>> tips.groupby(['smoker', 'day']).agg({'tip': [np.size, np.mean]})
              tip
             size      mean
smoker day
No     Fri    4.0  2.812500
       Sat   45.0  3.102889
       Sun   57.0  3.167895
       Thur  45.0  2.673778
Yes    Fri   15.0  2.714000
       Sat   42.0  2.875476
       Sun   19.0  3.516842
       Thur  17.0  3.030000
```
