# Oracle使用指南

此github建立的意义并非是讲oracle细枝末节的各种语法，而是旨在解决项目实践中遇到的一些共性问题。
鉴于本人精力有限，总结的难免有纰漏之处。如果有任何建议，还望各位读者指出。如果有所补充，也欢迎
新建一个branch或者邮箱联系我予以添加。本人邮箱wuboyu88@163.com

![oracle_logo](./image/oracle_logo.png)

## 展示方式说明
所有的展示都是通过以下三个部分组成：<br />
   1、问题 <br />
   2、代码or解决方案 <br />
   3、参考资料 <br />
   
##
Q1: oracle不像其他sql语言一样，可以用drop table if exists table_name这种方式删表。
那么在oracle中如何实现类似的功能呢？<br />
Code:
```sql
BEGIN
    EXECUTE IMMEDIATE 'drop table table_name';
EXCEPTION
    WHEN OTHERS THEN
      IF SQLCODE <> -942 THEN
        RAISE;
      END IF;
END; 
```
References:<br />
https://stackoverflow.com/questions/1799128/oracle-if-table-exists

##
Q2: oracle建表和插入数据顺序需保持一致<br />
Code:
```sql
-- 建表(字段顺序为a->b)
CREATE TABLE table1
  (
     a NUMBER,
     b NUMBER
  );

/*错误的插表方式*/
-- 插入数据(字段顺序为b->a,与建表时不一致)
INSERT INTO table1
SELECT b,
       a
FROM   table2;

COMMIT;

/*正确的插表方式*/
-- 插入数据(字段顺序为a->b,与建表时一致)
INSERT INTO table1
SELECT a,
       b
FROM   table2;

COMMIT;    
```
References:<br />
Not found

##
Q3: oracle如何解决plsql中保存的.sql文件中文注释为乱码的情况？<br />
Solution:<br />
configure --> preferences --> files --> format --> encoding --> always UTF8<br />
References:<br />
https://www.jianshu.com/p/1ecb687d1433

##
Q4: oracle报"ORA-00054 资源正忙, 但指定以 NOWAIT 方式获取资源, 或者超时失效"问题如何解决？<br />
Code:
```sql
-- 1.找出引发锁的会话和表名 
SELECT t1.session_id, 
       t2.owner, 
       t2.object_name, 
       t3.username, 
       t3.sid, 
       t3.serial#, 
       t3.logon_time 
FROM   v$locked_object t1 
       join dba_objects t2 
         ON t1.object_id = t2.object_id 
       join v$session t3 
         ON t1.session_id = t3.sid 
ORDER  BY t3.logon_time; 

-- 2.赋予username权限 
GRANT ALTER SYSTEM TO upro_test; --对应上面的username 

-- 3.杀掉会话 
ALTER SYSTEM kill SESSION 'sid,serial#'; --对应上面的sid和serial# 
```
References:<br />
https://blog.csdn.net/deniro_li/article/details/81085758 <br />
https://blog.csdn.net/hehuyi_in/article/details/89669553

##
Q5: oracle表空间查询<br />
Code:
```sql
-- 查看单表占用空间(注意表名需大写) 
SELECT segment_name, 
       bytes / ( 1024 * 1024 * 1024 ) AS space_gb 
FROM   user_segments 
WHERE  segment_name = 'S0CBI_P1_RESULT'; 

-- 查看剩余表空间
SELECT SUM(bytes) / ( 1024 * 1024 * 1024 ) AS space_gb 
FROM   dba_free_space 
WHERE  tablespace_name = 'UPRO';
```
References:<br />
https://www.cnblogs.com/pejsidney/p/8057372.html

## License
[MIT](https://choosealicense.com/licenses/mit/)