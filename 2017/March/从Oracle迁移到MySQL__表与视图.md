# 从Oracle迁移到MySQL__表与视图

> 最近刚巧经历了将实验室的工程后台数据库从Oracle迁移到MySQL数据库的过程，遇到了很多的坑，总结出了一系列问题

## 表和视图的迁移

### 1. 表的迁移

>实验室的数据库有几十张表，最大的表有27万条数据，手工迁移并不现实，本次使用软件迁移，软件名称为MySQL Migration Toolkit，版本号为1.0

1. 无论是本地连接还是远程连接，首先要确保Oracle和MySQL都可以使用ODBC正常连接，绝对大多数市面上常用的数据库迁移软件都使用ODBC进行数据库连接控制。

1. 在迁移表的时候，无论是使用软件还是手动迁移，要注意数据库字符集的匹配问题，如果字符集不匹配，可能会出现转完数据库发现是空表的情况（数据插入失败）。

    >实验室由于有大量的中文字符串信息，字符集设置为ZHS16GBK，迁移的时候将MySQL字符集设置为UTF8

1. Oracle中的很多数据类型，在MySQL绝大部分都有对应的数据类型，在迁移到MySQL时要按照对应的数据类型重新设计表格格式。

    >具体参考这篇博客：[MySQL与Oracle 差异比较之一数据类型](http://www.cnblogs.com/HondaHsu/p/3641116.html)。

    注意在Oracle中没有自动增长的数据类型，所以一般的做法是会建立一个自动增长的序列号，插入的时候手动插入。MySQL中是存在自动增长的数据类型的，因此要考虑是否要使用这种自动增长的字段设计。

1. 表的数据迁移完成后要检查一下主键和索引有没有设计，在转移的过程中有没有忘记设置同样的主键和索引。

### 2. 视图的迁移

>MySQL视图不支持子查询，Oracle视图没有这个问题，软件无法只能的解决这个问题，只能选择手动重新设计视图。

子查询问题采用笨办法就可以解决：将子查询语句写成子视图。但是要注意如果子查询套的层数很多的话，可能会出现严重的性能瓶颈，最终的视图可能根本无法使用，这时候只能考虑重新设计视图，或者优化视图的语句。
