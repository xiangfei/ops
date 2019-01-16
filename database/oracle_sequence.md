    




  下载LOFTER我的照片书  |
reference http://blog.csdn.net/zhoufoxcn/article/details/1762351

先假设有这么一个表：

create table S_Depart  (
   DepartId             INT                             not null,
   DepartName           NVARCHAR2(40)                   not null,
   DepartOrder          INT                            default 0,
   constraint PK_S_DEPART primary key (DepartId)
);
 

在oracle中sequence就是所谓的序列号，每次取的时候它会自动增加，一般用在需要按序列号排序的地方。 
1、Create Sequence 
你首先要有CREATE SEQUENCE或者CREATE ANY SEQUENCE权限， 
CREATE SEQUENCE emp_sequence 
INCREMENT BY 1 -- 每次加几个 
START WITH 1 -- 从1开始计数 
NOMAXvalue -- 不设置最大值 
NOCYCLE -- 一直累加，不循环 
CACHE 10; --设置缓存cache个序列，如果系统down掉了或者其它情况将会导致序列不连续，也可以设置为---------NOCACHE
针对S_Depart创建的sequence如下：

create sequence S_S_DEPART
minvalue 1
maxvalue 999999999999999999999999999
start with 1
increment by 1
nocache;


一旦定义了emp_sequence，你就可以用CURRVAL，NEXTVAL 
CURRVAL=返回 sequence的当前值 
NEXTVAL=增加sequence的值，然后返回 sequence 值 
比如： 
emp_sequence.CURRVAL 
emp_sequence.NEXTVAL 

可以使用sequence的地方： 
- 不包含子查询、snapshot、VIEW的 SELECT 语句 
- INSERT语句的子查询中 
- NSERT语句的valueS中 
- UPDATE 的 SET中 

可以看如下例子： 

insert into S_Depart(departId,Departname,Departorder)values(S_S_Depart.Nextval,'12345',1);


SELECT empseq.currval FROM DUAL; 

但是要注意的是： 
- 第一次NEXTVAL返回的是初始值；随后的NEXTVAL会自动增加你定义的INCREMENT BY值，然后返回增加后的值。CURRVAL 总是返回当前SEQUENCE的值，但是在第一次NEXTVAL初始化之后才能使用CURRVAL，否则会出错。一次NEXTVAL会增加一次 SEQUENCE的值，所以如果你在同一个语句里面使用多个NEXTVAL，其值就是不一样的。明白？ 

- 如果指定CACHE值，ORACLE就可以预先在内存里面放置一些sequence，这样存取的快些。cache里面的取完后，oracle自动再取一组 到cache。 使用cache或许会跳号， 比如数据库突然不正常down掉（shutdown abort),cache中的sequence就会丢失. 所以可以在create sequence的时候用nocache防止这种情况。 

2、Alter Sequence 
你 或者是该sequence的owner，或者有ALTER ANY SEQUENCE 权限才能改动sequence. 可以alter除start至以外的所有sequence参数.如果想要改变start值，必须 drop sequence 再 re-create . 
Alter sequence 的例子 
ALTER SEQUENCE emp_sequence 
INCREMENT BY 10 
MAXvalue 10000 
CYCLE -- 到10000后从头开始 
NOCACHE ; 


影响Sequence的初始化参数： 
SEQUENCE_CACHE_ENTRIES =设置能同时被cache的sequence数目。 

可以很简单的Drop Sequence 
DROP SEQUENCE order_seq; 

一个简单的例子：
create sequence SEQ_ID
minvalue 1
maxvalue 99999999
start with 1
increment by 1
nocache
order;

建解发器代码为：

create or replace trigger tri_test_id
  before insert on S_Depart   --S_Depart 是表名
  for each row
declare
  nextid number;
begin
  IF :new.DepartId IS NULLor :new.DepartId=0 THEN --DepartId是列名
    select SEQ_ID.nextval --SEQ_ID正是刚才创建的
    into nextid
    from sys.dual;
    :new.DepartId:=nextid;
  end if;
end tri_test_id;
OK，上面的代码就可以实现自动递增的功能了。