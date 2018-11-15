set termout off
set echo off
set feedback off
set heading off
set verify off
--pagesize 0用来去除空行
set pagesize 0
set trimspool on
set trimout on
set colsep |
spool user_n.txt

select LUSER_NAME from LSMP_LOTTERY_USER where id>=? and id<?;


spool off