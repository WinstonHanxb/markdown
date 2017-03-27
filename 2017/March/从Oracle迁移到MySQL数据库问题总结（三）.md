# 从Oracle迁移到MySQL数据库问题总结（一）

> 最近刚巧经历了将实验室的工程后台数据库从Oracle迁移到MySQL数据库的过程，遇到了比较多的坑，总结出了一系列问题

##函数与存储过程的迁移

>实验室Oracle函数与存过全部是使用PL/SQL语句写的，MySQL不支持PL/SQL，也无法使用软件进行转换，因此必须手动更改。

MySQL的基本语法与Oracle有很多不同的地方，具体可以参见下面的连接：

* [MySQL与Oracle 差异比较之二基本语法](http://www.cnblogs.com/HondaHsu/p/3641183.html)

* [MySQL与Oracle 差异比较之三内置函数](http://www.cnblogs.com/HondaHsu/p/3641190.html)

* [MySQL与Oracle 差异比较之四条件循环语句](http://www.cnblogs.com/HondaHsu/p/3641246.html)

* [MySQL与Oracle 差异比较之五存储过程&Function](http://www.cnblogs.com/HondaHsu/p/3641258.html)

其中遇到的比较多的几个问题有：

#### 1.变量的声明和赋值方式不同

  MySQL的声明必须在函数或者过程的BEGIN和END之间，**处于所有其他语句之前**，且MySQL**不支持自定义数据类型**。

    DECLARE li_index INTEGER DEFAULT 0;
    SET Li_index = 1;

  Oracle的声明要在BEGIN之前

    li_index NUMBER :=0;

#### 2.对数组的支持不同

 Oracle支持数组，而MySQL不支持数组，在迁移的过程中遇到很多Oracle存储过程之间传递数组的情况。针对这种情况，只能在改写的时候将数组设计成为临时表，数组的存取过程全部**改写成为临时表**的存取。

 要注意创建临时表的函数或者过程一定要记得配套的在最后写上删除临时表的语句。

#### 3.对时间的处理不同

 Oracle的时间格式如下，获取当前数据库系统时间的函数为**SYSDATE**精确到秒。

    yyyy-MM-dd hh:mi:ss

 MySQL的时间格式如下，获取当前系统时间的函数为**now()**，精确到秒。

    %Y-%m-%d %H:%i:%s

#### 3.字符串的连接方式不同

 Oracle的字符串连接符为"**||**"

    result := v_init1 || v_init2;

 MySQL的字符串连接使用**concat**函数

    set result = concat(v_init1, v_init2);

#### 4.内置的函数对应名称和用法不同

 参见[MySQL与Oracle 差异比较之三内置函数](http://www.cnblogs.com/HondaHsu/p/3641190.html)

#### 5. 对包的支持不同
 
 Oracle支持建包，而MySQL不存在这种功能，在迁移的过程中需要将包拆分成存储过程或函数，为了整理方便，将存储过程或函数的名称改为包名_函数名的形式。

#### 6.对异常的支持不同

 MySQL的内部异常需要先定义，并且定以后要写明异常的行为，MySQL**不支持自定义异常**。MySQL定义异常的格式如下。

    DECLARE EXIT HANDLER FOR  SQLEXCEPTION 
    BEGIN
    ROLLBACK ;
    set ov_rtn_msg = concat(c_sp_name,'(', li_debug_pos ,'):',
        TO_CHAR(SQLCODE),': ',SUBSTR(SQLERRM,1,100));
    END;

Oracle的内部异常不需要定义，在存储过程或函数末尾鞋厂EXCEPTION后，后面的部分即为异常处理的部分。
Oracle支持自定义异常，但是需要使用raise关键字抛出异常才能被EXCEPTION捕获。Oracle定义异常的格式如下。

    EXCEPTION
    WHEN OTHERS THEN
    ROLLBACK ;
    ov_rtn_msg := c_sp_name||'('|| li_debug_pos ||'):'||
        TO_CHAR(SQLCODE)||': '||SUBSTR(SQLERRM,1,100);

#### 7.Oracle和MySQL对函数的声明方式和要求有所不同

 MySQL的函数创建语句如下，同时只允许传入参数不允许Oracle一样可以对传入的参数进行修改，因此遇到Oracle中采用in/out类型参数的函数要考虑改写成为存储过程或者重新设计函数。

    DROP FUNCTION IF EXISTS `SD_ROLE_F_ROLE_FACS_GRP`;
    CREATE  FUNCTION `SD_ROLE_F_ROLE_FACS_GRP`(
     ii_role_int_key INTEGER(10)
    ) RETURNS varchar(1000) 

Oracle的函数创建语句如下，传入的参数允许设置成in、out、in/out三种类型

    CREATE OR REPLACE FUNCTION F_ROLE_FACS_GRP(
     ii_role_int_key IN SD_ROLE.ROLE_INT_KEY%TYPE
    ) RETURN VARCHAR2

 注意到Oracle支持将参数定义为表的字段类型，而MySQL不支持，必须直接写明。

