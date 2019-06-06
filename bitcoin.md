#详细文章和图见myblog
>点击这里直达[makerroot](https://www.makerroot.com/detail/4)

#勒索防不胜防（2019/06/01儿童节）
>儿童节快乐吗？反正我是不快乐，一方面我已经不是儿童了，另一个方面辛苦写的文章被一个叫做‘勒索’的东西全部搞掉，写到此处的时候很难受，所以不管什么东西，安全性是真的很重要，所以呀经常备份就很重要了，养成良好的备份习惯从我做起。
##勒索之后的表大家看看，如图所示。


##为了解决这个问题，首先从数据库开始入手。
>1建议关闭所有的外部连接数据库的形式
>`select host,user from user;`
>显示如图所示

>删除host含有%的用户数据
>`delete from user where host='%';`
>删除之后如图所示。

>2开启binlog
>binlog 基本认识
	MySQL的二进制日志可以说是MySQL最重要的日志了，它记录了所有的DDL和DML(除了数据查询语句)语句，以事件形式记录，还包含语句所执行的消耗的时间，MySQL的二进制日志是事务安全型的。
	
	一般来说开启二进制日志大概会有1%的性能损耗(参见MySQL官方中文手册 5.1.24版)。二进制有两个最重要的使用场景: 
	其一：MySQL Replication在Master端开启binlog，Mster把它的二进制日志传递给slaves来达到master-slave数据一致的目的。 
	其二：自然就是数据恢复了，通过使用mysqlbinlog工具来使恢复数据。
	二进制日志包括两类文件：二进制日志索引文件（文件名后缀为.index）用于记录所有的二进制文件，二进制日志文件（文件名后缀为.00000*）记录数据库所有的DDL和DML(除了数据查询语句)语句事件。 
>开启binlog日志的方法：
    vim编辑打开mysql配置文件
    # vim /etc/my.cnf
    在[mysqld] 区块
    设置/添加 log-bin=mysql-bin
	确认是打开状态(值 mysql-bin 是日志的基本名或前缀名)；
	重启mysqld服务使配置生效
	如图所示。



>也可登录mysql服务器，通过mysql的变量配置表，查看二进制日志是否已开启

    登录服务器
    # mysql -u root -p密码

    mysql> show variables like 'log_%'; 
   

>3常用binlog日志操作命令
    1.查看所有binlog日志列表
      mysql> show master logs;

    2.查看master状态，即最后(最新)一个binlog日志的编号名称，及其最后一个操作事件pos结束点(Position)值
      mysql> show master status;

    3.刷新log日志，自此刻开始产生一个新编号的binlog日志文件
      mysql> flush logs;
      注：每当mysqld服务重启时，会自动执行此命令，刷新binlog日志；在mysqldump备份数据时加 -F 选项也会刷新binlog日志；

    4.重置(清空)所有binlog日志
      mysql> reset master;


>4查看某个binlog日志内容

         mysql> show binlog events [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count];

             选项解析：
               IN 'log_name'   指定要查询的binlog文件名(不指定就是第一个binlog文件)
               FROM pos        指定从哪个pos起始点开始查起(不指定就是从整个文件首个pos点开始算)
               LIMIT [offset,] 偏移量(不指定就是0)
               row_count       查询总条数(不指定就是所有行)

             
      
      A.查询第一个(最早)的binlog日志：
        mysql> show binlog events\G; 
    
      B.指定查询 mysql-bin.000021 这个文件：
        mysql> show binlog events in 'mysql-bin.000021'\G;

      C.指定查询 mysql-bin.000021 这个文件，从pos点:8224开始查起：
        mysql> show binlog events in 'mysql-bin.000021' from 8224\G;

      D.指定查询 mysql-bin.000021 这个文件，从pos点:8224开始查起，查询10条
        mysql> show binlog events in 'mysql-bin.000021' from 8224 limit 10\G;

      E.指定查询 mysql-bin.000021 这个文件，从pos点:8224开始查起，偏移2行，查询10条
        mysql> show binlog events in 'mysql-bin.000021' from 8224 limit 2,10\G;

      总结：所谓恢复，就是让mysql将保存在binlog日志中指定段落区间的sql语句逐个重新执行一次而已。