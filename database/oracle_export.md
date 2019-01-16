reference  http://wenku.baidu.com/view/2266b7ca05087632311212d0.html

背景
Oracle 11g使用exp时，默认无法导出空表；

导入时也无法指定表空间。

 

导出空表的解决方法
查看是否能导出空表
show parameter deferred_segment_creation

deferred_segment_creation=TRUE表示空表不创建数据段，这是就不能导出表了。

只有在这个参数设置为FALSE后新建的表才能导出。

alter system set deferred_segment_creation=false;

解决方法
给空表增加一条记录，然后再删除，然后就能导出了。

l         导出表

Exp xxx/xxx@dbsid file=xxx.dmp owner=(xxx,xxx);

l         然后将xxx.dmp导入一个临时的用户A中，

l         将临时用户A的表和原用户表比较，查询所有空表，

用sys登录，运行

select object_name,CREATED,STATUS from dba_objects where owner='XXX'  AND OBJECT_TYPE='TABLE' and OBJECT_NAME NOT IN

(select OBJECT_NAME from dba_objects where owner='A'  ) ORDER BY OBJECT_NAME

 

l          禁用所有外键约束

运行SQL,生成禁用所有外键的SQL预计

select 'alter table '||table_name||' disable constraint '||constraint_name||';' from user_constraints where constraint_type='R'

然后将结果复制下来运行即可。

 

l          使用POWERDESIGN等工具生成测试数据或者手工生成测试数据

使用IMP命令生成这些空表的创建语句

Imp xxx/xxx@dbsid show=y tables=(xxx,xxx) log=xxx.log

然后将xxx.log改成xxx.sql导入到powerdesign中，再利用powerdesign的生成测试数据生成测试数据testdata.sql，生成测试数据前要先设置自动填充的值的模式，否则会生成很长的字符串。

 

l         运行测试数据，然后删除测试数据，空表就会有数据段

l         恢复外键约束

运行SQL,生成启动所有外键的SQL预计

select 'alter table '||table_name||' enable constraint '||constraint_name||';' from user_constraints where constraint_type='R'

然后将结果复制下来运行即可。

l          导出所有表，这次导出的文件就包含了空表了

 

 

 

 

将数据导出到指定表空间
 

l          导出数据，最好指定导出某个用户的数据

exp userid/pwd@dbsid file=xxx.dmp owner=(xxx ,xxx)

 

l          创建表空间

create tablespace XXXX

nologging

datafile '+DATA/webdb/datafile/XXX.dbf'

size 1024m

autoextend on

next 100m

maxsize 30810m

extent management local

解释：

nologging 表示不用创建日志，由于新创建的表空间，无需日志

+DATA/webdb/datafile/XXX.dbf' 这个是存储设备路径

 

l          授权用户使用该表空间

alter user XXX quota unlimited on XXXX;

 

l          修改oracle导出文件xxx.dmp中的表空间

Vim xxx.dmp

%s/TABLESPACE "XXX"/XXXX/g

 

l          导入

Imp xxx/xxx@dbsid fromuser=xxx touser=xxx

 

 

l         检查表空间的表

以用户身份登录

select tablespace_name from user_tables where table_name = 'tabname'

 

 

查询空表的另外一种方法
SET SERVEROUTPUT ON;

exec dbms_output.enable(200000);

 

DECLARE

v_table dba_objects.object_name%TYPE;

v_sql VARCHAR2(888);

v_q NUMBER;

CURSOR c1 IS

SELECT object_name tn FROM dba_objects where owner='xxxx' and object_type='TABLE';

TYPE c IS REF CURSOR;

c2 c;

BEGIN

    DBMS_OUTPUT.PUT_LINE('empty table:');

    FOR r1 IN c1 LOOP

        v_table :=r1.tn;

        v_sql :='SELECT count(*) q FROM xxx.'||v_table||' where rownum = 1';

        OPEN c2 FOR v_sql;

        LOOP

            FETCH c2 INTO v_q;

                EXIT WHEN c2%NOTFOUND;

                IF v_q=0 THEN

                     DBMS_OUTPUT.PUT_LINE(v_table);

                END IF;

        END LOOP;

        CLOSE c2;

    END LOOP;

    EXCEPTION

    WHEN OTHERS THEN DBMS_OUTPUT.PUT_LINE('Error occurred');

END;

/

 

解析

Xxx代表用户名。

需要用DBA角色登录。

 

 

 

修改数据库字符集
数据库字符集在创建后原则上不能更改。如果需要修改字符集，通常需要导出数据库数据，重建数据库，再导入数据库数据的方式来转换，或通过ALTER DATABASE CHARACTER SET语句修改字符集，但创建数据库后修改字符集是有限制的，只有新的字符集是当前字符集的超集时才能修改数据库字符集，例如UTF8是US7ASCII的超集，修改数据库字符集可使用ALTER DATABASE CHARACTER SET UTF8。

 

在Export过程中，如果源数据库字符集与Export用户会话字符集不一致，会发生字符集转换，并在导出文件的头部几个字节中存储Export用户会话字符集的ID号。在这个转换过程中可能发生数据的丢失。
例:如果源数据库使用ZHS16GBK，而Export用户会话字符集使用US7ASCII，由于ZHS16GBK是16位字符集,而US7ASCII是7位字符集，这个转换过程中，中文字符在US7ASCII中不能够找到对等的字符，所以所有中文字符都会丢失而变成“?? ”形式，这样转换后生成的Dmp文件已经发生了数据丢失。
因此如果想正确导出源数据库数据，则Export过程中用户会话字符集应等于源数据库字符集或是源数据库字符集的超集

 

l         查询数据库字符集

查看数据字符集

Select * from v$NLS_PARAMETERS;

Select * from nls_database_parameters;

查看session

Select * from nls_session_parameters;

 

 

l         客户端字符集设置

NLS_LANG参数格式

    NLS_LANG=<language>_<territory>.<client character set>

    Language:显示oracle消息,校验，日期命名

    Territory：指定默认日期、数字、货币等格式

    Client character set：指定客户端将使用的字符集

    例如：NLS_LANG=AMERICAN_AMERICA.US7ASCII 

    AMERICAN是语言，AMERICA是地区，US7ASCII是客户端字符集

客户端字符集设置方法

         $NLS_LANG=“simplified chinese”_china.zhs16gbk

         $export NLS_LANG

         编辑oracle用户的profile文件

 

l         导入数据

设置字符集和目标数据库字符集

NLS_LANG=AMERICAN_AMERICA.ZHS16GBK

export NLS_LANG

 

ZHS16GBK到 AL32UTF8 是不用转换的。

修改目标数据库字符集让它和源数据库字符集相同

update props$ set values$='US7ASCII'　where name='NLS_CHARACSET'

Imp xxx

update props$ set values$='US7ASCII'　where name='NLS_CHARACSET'