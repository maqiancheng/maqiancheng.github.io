---
layout: post
title: "索引失效"
date: 2017-04-28
tag: java
---   

### SQL优化避免索引失效  
1.对查询进行优化，应尽量避免全表扫描，首先应该考虑在where及order by涉及的列上建立索引。  
2.应该避免在where子句中对字段进行null值判断，否则将导致引擎放弃索引而进行全表扫描，如：  
select id from t where num is null;  
可以在num上面设置默认值是0，确保表中num列没有null值，然后这样查询：  
select id from t where num = 0;  
3.应尽量避免在where子句中使用!=或<>操作符，否则引擎将放弃索引而进行全表扫描。  
4.应尽量避免在where子句中使用or连接条件，否则将导致引擎放弃索引而进行全表扫描，如：  
select id from t where num = 10 or num = 20;  
可以这样查询：  
select id from t where num = 10  
union all  
select id from t where num = 20  
5.in 和 not in 也要慎用，否则会导致全表扫描，如：  
select id from t where num in (1,2,3)  
对于连续的值，能between就不要用in了：  
select id from t where num between 1 and 3  
6.下列查询也将导致全表扫描：  
select id from t where name like '%abc%'  
若要提高效率可以考虑全文检索  
7.如果在where子句中使用参数，也会导致全表扫描。因为sql只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；  
它必须在编译时进行选择。然而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。如果下面语句将进行全表扫描。  
select id from t where num = @num;  
可以改为强制使用索引：  
select id from t with(index(索引名)) where num= @num;  
8.应该尽量避免在where子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。如：  
select id from t where num/2 = 100  
应改为：  
select id from t where num = 100*2  
9.应尽量避免在where子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。  
如：  
select id from t where substring(name, 1, 2) = 'abc';  
select id from t where datadiff(day,createdate,'2016-11-11') = 0;  
应改为：  
select id from t where name like 'abc%';
select id from t where createdate > '2016-11-11' and createdate < '2016-11-13';  
10.不要在where子句“=”的左边进行函数、算术运算或其他表达式运算，否则系统将可能无法使用索引。  
11.在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时，才能保证系统使用该索引，否则该索引将不会被使用，并且尽可能的将字段顺序和索引顺序相一致。  
12.很多时候用exists代替in是一个很好的选择：  
select num from a where num in(select num from b);  
用下面的语句进行替换：  
select num from a where num exists(select 1 from b where num = a.num);  
13.并不是所有查询都是有效的，sql是根据表中数据来进行查询优化的，当索引列有大量数据重复时，sql查询可能不会利用索引，如一张表中有字段sex，male、fmale各占一半，那么即使在sex上建立索引也对查询效率起不了作用。  
14.索引并不是越多越好，索引固然可以提高相应的select效率，但同时降低了insert和update的效率，因为insert和update可能需要重建索引，所以怎样建索引需要慎重考虑，视具体情况而定，一个表的索引数量最好不要超过6个。  
15.尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询的连续性能，并会增加存储开销。这是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了。  
16.尽可能的使用varchar代替char，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。  
17.任何地方都不要使用 select * from t ，用具体的字段列表代替“*”，不要返回用不到的任何字段。   
