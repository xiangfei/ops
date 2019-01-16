oracle 更新字段名、添加字段、删除字段

对字段操作	操作方法
更新字段名	alter table TABLE_NAME rename column column_old to column_new;
添加字段	alter table TABLE_NAME add COLUMN_NAME varchar(10);
删除字段	alter table TABLE_NAME drop column COLUMN_NAME;
添加字段并附值	alter table TABLE_NAME ADD COLUMN_NAME NUMBER(1) DEFAULT 1;
修改字段值	updata TABLE_NAME set filedname=value where filedname=value;
修改字段数据类型	alter table tablename modify filedname varchar2(20);
如果该列有数据就不起作用了（有数据时修改会报ORA-01439: column to be modified must be empty to change datatype）,需要通过另外一种方式：

alter table tablename add filedname_temp varchar2(20)
 
update tablename set filedname_temp = filedname ;
 
alter table tablename  drop column filedname;
 
alter table tablename rename column filedname_temp to filedname;

这种方法能满足需求，但是会使列名发生变化，而且字段顺序有所改变(新增字段默认添加到表末尾，有可能发生行迁移,对应用程序会产生影响）

那么好吧，就说说比较好点的方法吧！

–新增临时列 
alter table tablename add filedname_temp number(2);
–将临时列的值置空

update zyt set id_temp=null; -----#alter table tablename modify filedname null;
–将要更新的字段值挪到临时列，并置空该列
 update tablename set filedname_temp=filedname,filedname=null;
 commit;
 –修改列的数据类型为varchar2
 alter table tablename modify filedname varchar2(20);
 –将要临时列值重新挪到该列，并置空临时列
 update tablename set filedname=filedname_temp,filedname_temp=null;
 commit;
 –删除临时列
 alter table tablename drop column filedname_temp;
 –给该列不能为空
 alter table tablename modify filedname not null;
 –执行查询测试
 select * from tablename ;
 使用这种方式，既不用使列名发生变化，也不会发生表迁移，但有个缺点是表要更新两次，而且当如果数据量较大时，产生的undo和redo也更多，前提也是要停机才进行操作，如果不停机 ,也可以采用在线重定义方式来做。





demo



alter table lsmp_stat_baward  add id_bak  number(18,0);
update lsmp_stat_baward set id_bak = id;
alter table lsmp_stat_baward  drop column id;
alter table lsmp_stat_baward  rename column id_bak to id;
alter table lsmp_stat_baward add primary key (id);