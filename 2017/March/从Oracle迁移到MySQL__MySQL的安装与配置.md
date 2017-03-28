# 从Oracle迁移到MySQL数据库问题总结（一）

> 最近刚巧经历了将实验室的工程后台数据库从Oracle迁移到MySQL数据库的过程，遇到了很多的坑，总结出了一系列问题。

## MySQL的安装与配置

>MySQL除了安装以外，还需要配置才能使用，本次数据库迁移使用的MySQL版本号为5.7.15（32bit）
 
 * 安装MySQL一路点击下一步就好了。

 * 安装完成后首先需要进入MySQL安装的文件夹下配置一下my.ini文件（要注意使用**无格式**的文本编辑器，否则会出现my.ini无法识别的情况)。
 
        [mysql]
        #设置mysql客户端默认字符集
        default-character-set=utf8 
        [mysqld]
        #设置3306端口
        port = 3306 
        # 设置mysql的安装目录
        basedir=D:\MySql\mysql-5.7.12-winx64
        # 设置mysql数据库的数据的存放目录
        datadir=D:\MySql\mysql-5.7.12-winx64\data
        # 允许最大连接数
        max_connections=200
        # 服务端使用的字符集默认为8比特编码的latin1字符集
        character-set-server=utf8
        # 创建新表时将使用的默认存储引擎
        default-storage-engine=INNODB
        #调整线程栈的大小
        thread_stack = 256K 

    >在后面实际迁移的过程中还遇上了MySQLs线程栈大小不够的问题

    >>ERROR 1436 (HY000): Thread stack overrun:  6860 bytes used of a 131072 byte stack, and 128000 bytes needed.  Use 'mysqld-thread_stack=#' to specify a bigger stack.

    >因此最后补上一个thread_stack将stack大小调整为k

 *  my.ini配置完成后保存在MySQL的安装文件夹，并在安装文件夹下面建立一个空的data文件夹（data必须是空的，否则要使用不同的命令）。
 *  用**管理员模式**打开CMD进入MysQL所在的文件夹，输入下面的命令

        mysqld --initialize-insecure --user=mysql

        mysqld -install

        net start mysql

    如此就能启动本机上MySQL服务，要注意第一句" mysqld --initialize-insecure --user=mysql"非常重要不可省略，
    "mysqld --initilalize" 的作用是在初始化MySQL下的data文件夹并且为root产生一个不随机的密码。

    >MySQL5.7中新增的特性中其中有一个主要的方面就是极大的增强了安全性能，安装完MySQL后默认会为root@localhost用户创建了一个随机密码，这个随机密码在不同的系统上会使用不同的方式查找，否则无法登陆MySQL并且修改初始密码。

    "-user=mysql"这一步指定了mysql用户是运行mysqld进程的用户名。设置这个用户以后，所有mysqld进程创建的文件都会属于mysql用户，在生产环境中便于管理。

*  如果有必要，可以为MySQL的bin文件夹配置环境变量。

*  如果没有配置环境变量，可以在MySQL安装文件夹下路径下执行"mysql -u root -p"进入MySQL，按照上面的初始方式没有密码，提示输入密码时直接回车即可。

*  注意**MySQL对远程连接的账户要求非常严格**，root账户默认是没有开启远程连接的，因此如果是远程服务器上的数据库，切记要配置一个远程连接的账户，并且为远程账户配置权限。