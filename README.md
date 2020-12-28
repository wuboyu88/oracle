# Oracle使用指南

此github建立的意义并非是讲oracle细枝末节的各种语法，而是旨在解决项目实践中遇到的一些共性问题。
鉴于本人精力有限，总结的难免有纰漏之处。如果有任何建议，还望各位读者指出。如果有所补充，也欢迎
新建一个branch或者邮箱联系我予以添加。本人邮箱wuboyu88@163.com

![oracle_logo](./image/oracle_logo.png)

## 展示方式说明
所有的展示都是通过以下三个部分组成：<br />
   1、问题 <br />
   2、代码 <br />
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

## License
[MIT](https://choosealicense.com/licenses/mit/)