# having和where的区别

**1.where和having的区别**</br>

where和having都是用于筛选过滤, 但是使用场景不同:

(1)where是对分组之前的数据进行筛选过滤

(2)having是对分组之后的数据进行筛选过滤

(3)where中不能使用列别名和聚合函数(max/min/sum/avg/count)

(4)having中可以使用列别名和聚合函数

(5)如果where和having不是同时出现, 大多数情况下, having可以替代where, 但是反过来不行!!


**where**:</br>
- where是一个约束声明,使用where来约束来自数据库的数据;</br>
- where是在结果返回之前起作用的;</br>
- where中不能使用聚合函数。

**having**:</br>
- having是一个过滤声明;</br>
- 在查询返回结果集以后，对查询结果进行的过滤操作;</br>
- 在having中可以使用聚合函数。

**2.聚合函数和group by**</br>
聚合函数就是例如`SUM, COUNT, MAX, AVG`等对一组(多条)数据操作的函数，需要配合group by 来使用。
```sql
#如：
SELECT SUM(population),region FROM T01_Beijing GROUP BY region; //计算北京每个分区的人数
```

**3.where和having的执行顺序**</br>
- where 早于 group by   早于 having
- where子句在聚合前先筛选记录，也就是说作用在group by 子句和having子句前，而 having子句在聚合后对组记录进行筛选

**4.where不能使用聚合函数、having中可以使用聚合函数**
```sql
#筛选出北京西城、东城、海淀及各区学校数量
SELECT region,count(school) 
FROM T02_Bejing_school 
WHERE region IN ('海淀' , '西城' , '东城') GROUP BY region；
```

```sql
#筛选出北京西城、东城、海淀三个区中学校数量超过10所的区及各区学校数量。
SELECT region,count(school) 
FROM T02_Bejing_school 
WHERE region IN ('海淀' , '西城' , '东城') 
GROUP BY region HAVING count(school) > 10;
```
注意！我们不能用where来筛选超过学校数量超过10的区，因为表中不存在这样一条记录。而HAVING子句可以让我们筛选成组后的各组数据．