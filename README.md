# Oracle使用指南

此github旨在解决项目实施过程中遇到的一些共性问题。总结的难免有纰漏之处。如果有任何建议，还望各位读者指出。如果有所补充，
也欢迎新建一个branch或者邮箱联系我予以添加。本人邮箱wuboyu88@163.com

![oracle_logo](./image/oracle_logo.png)

## 展示方式说明
所有的展示都是通过以下三个部分组成：<br />
   1、问题 <br />
   2、代码or解决方案 <br />
   3、参考资料 <br />
   
##
**Q1**: oracle不像其他sql语言一样，可以用drop table if exists table_name这种方式删表。
那么在oracle中如何实现类似的功能呢？<br />
**Code**:
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
**References**:<br />
https://stackoverflow.com/questions/1799128/oracle-if-table-exists

##
**Q2**: oracle建表和插入数据顺序需保持一致<br />
**Code**:
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
**References**:<br />
Not found

##
**Q3**: oracle如何解决plsql中保存的.sql文件中文注释为乱码的情况？<br />
**Solution**:<br />
configure --> preferences --> files --> format --> encoding --> always UTF8<br />
**References**:<br />
https://www.jianshu.com/p/1ecb687d1433

##
**Q4**: oracle报"ORA-00054 资源正忙，但指定以 NOWAIT 方式获取资源，或者超时失效"问题如何解决？<br />
**Code**:
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
       JOIN dba_objects t2 
         ON t1.object_id = t2.object_id 
       JOIN v$session t3 
         ON t1.session_id = t3.sid 
ORDER  BY t3.logon_time; 

-- 2.赋予username权限 
GRANT ALTER SYSTEM TO upro_test; --对应上面的username 

-- 3.杀掉会话 
ALTER SYSTEM KILL SESSION 'sid,serial#'; --对应上面的sid和serial# 
```
**References**:<br />
https://blog.csdn.net/deniro_li/article/details/81085758 <br />
https://blog.csdn.net/hehuyi_in/article/details/89669553

##
**Q5**: oracle表空间查询<br />
**Code**:
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
**References**:<br />
https://www.cnblogs.com/pejsidney/p/8057372.html

##
**Q6**: oracle数据库无法匹配中文<br />
**Solution**:<br />
添加环境变量<br />
变量名：NLS_LANG<br />
变量值：SIMPLIFIED CHINESE_CHINA.ZHS16GBK<br />
**References**:<br />
https://www.cnblogs.com/1201x/p/6513681.html

##
**Q7**: oracle得到指定日期的上个月最后一天<br />
**Code**:
```sql
SELECT LAST_DAY(ADD_MONTHS(yourdate,-1))
```
**References**:<br />
https://stackoverflow.com/questions/4957224/getting-last-day-of-previous-month-in-oracle-function

##
**Q8**: oracle存储过程默认参数如何设置<br />
**Code**:
```sql
CREATE OR REPLACE PROCEDURE my_procedure(cutoff_date IN VARCHAR2 DEFAULT '2020-05-31',
                                         nb_days     IN NUMBER DEFAULT 7)
```
**References**:<br />
https://oracle-patches.com/en/databases/sql/3777-pl-sql-procedure-setting-default-parameter-values

##
**Q9**: oracle建表之后加主键约束<br />
**Code**:
```sql
ALTER TABLE tablename
  ADD CONSTRAINT pk_id PRIMARY KEY (id); 
```
**References**:<br />
https://blog.csdn.net/yuan_csdn1/article/details/78158247

##
**Q10**: oracle自定义函数性能优化<br />
**Solution**: oracle通过关键字DETERMINISTIC来表明一个函数（Function）是确定性的，确定性函数可以用于创建基于函数的索引。
因此自定义函数加了DETERMINISTIC关键字可以提高执行效率。<br />
**Code**:
```sql
/*自定义get_last_day_n_month获得前n个月的最后一天*/ 
CREATE OR REPLACE FUNCTION get_last_day_n_month(date_str IN VARCHAR2, 
                                                n        IN NUMBER) 
  RETURN DATE DETERMINISTIC IS 
  res DATE; 
BEGIN 
  res := LAST_DAY(ADD_MONTHS(TO_DATE(date_str, 'YYYY-MM-DD'), -n)); 
  RETURN res; 
END;
```
**References**:<br />
https://www.cnblogs.com/kerrycode/p/9099507.html

##
**Q11**: oracle斜杠(/)的使用场景<br />
**Solution**: 我们都知道sql语句可以以分号(;)结尾，通常情况下这么做都没有任何问题。
但是当我们想批量执行代码块时，比如想批量执行100个建表语句或者存储过程时，如果不加
斜杠(/)就会只执行第一个代码块，后续无法执行。因此建议用斜杠(/)对工作单元进行隔离。
<br />
**Code**:
```sql
CREATE OR REPLACE PROCEDURE procedure1 AS 
BEGIN 
... 
END;
/
CREATE OR REPLACE PROCEDURE procedure2 AS 
BEGIN 
... 
END;
/
```
**References**:<br />
https://stackoverflow.com/questions/1079949/when-do-i-need-to-use-a-semicolon-vs-a-slash-in-oracle-sql

##
**Q12**: oracle并行优化之PARALLEL的使用<br />
**Solution**: PARALLEL可以应用在三个场景中(CREATE TABLE、INSERT INTO、SELECT)，
有的时候PARALLEL会以注释的形式出现在代码里，让大家觉得可有可无，请注意，
这不是普通的注释。<br />
**Code**:
```sql
-- 1.CREATE TABLE
CREATE TABLE table1 PARALLEL NOLOGGING AS
SELECT * FROM table2;
/
-- 2.INSERT INTO & SELECT
INSERT INTO /*+ APPEND PARALLEL(table1) NOLOGGING */ table1
  SELECT /*+ PARALLEL NOLOGGING */ a, b FROM table2;
/
```
**References**:<br />
https://docs.oracle.com/database/121/VLDBG/GUID-7A6185D1-AEF3-48C3-A704-8E3D7D3A4AFD.htm#VLDBG1526

##
**Q13**: oracle存储过程中字符串的转化<br />
**Solution**: 简而言之，普通字符常量的单引号(')变成双引号(''),
传入的变量在其两边加上'''|| ||'''。另外要注意，oracle中的单引号(')和双引号('')
意义是不一样的，不像python中两者基本等价。<br />
**Code**:
```sql
CREATE OR REPLACE PROCEDURE procedure1(cutoff_date in VARCHAR2) IS
  sql_str VARCHAR2(20000);
BEGIN
  sql_str := 'CREATE TABLE table1 AS
              SELECT * FROM table2
              WHERE dt = TO_DATE('''|| cutoff_date ||''', ''YYYY-MM-DD'')
                AND a in (''1'', ''2'')';
  EXECUTE IMMEDIATE sql_str;
END;
```
**References**:<br />
https://www.itdaan.com/tw/18d29a2d6d9a17c524919f7821b3254b

##
**Q14**: oracle中哪些语句需要COMMIT<br />
**Solution**: COMMIT操作非常重要，比如UPDATE后，没有及时COMMIT，
很容易导致死锁问题。就好比我们想对一个文件一边写入，一边删除一样。
类似的，python也有个进程锁GIL锁（全局解释器锁）。<br />
<br />
DDL(数据定义语言) - CREATE、ALTER、DROP这些语句自动提交，无需用COMMIT提交。<br />
DQL(数据查询语言) - SELECT查询语句不存在提交问题。<br />
DML(数据操纵语言) - INSERT、UPDATE、DELETE 这些语句需要COMMIT才能提交。<br />
DTL(事务控制语言) - COMMIT、ROLLBACK 事务提交与回滚语句。<br />
DCL(数据控制语言) - GRANT、REVOKE 授予权限与回收权限语句。<br />
**References**:<br />
https://blog.csdn.net/wtl1992/article/details/100553738

##
**Q15**: oracle中如何创建分区表<br />
**Code**:
```sql
-- 例子中是以dt作为分区字段，按天创建分区；将2018-11-01之前的当成一个分区
CREATE TABLE table1
(
id VARCHAR(99),
dt DATE,
a NUMBER,
b NUMBER,
PRIMARY KEY(dt, id))
PARTITION BY RANGE(dt) INTERVAL (NUMTODSINTERVAL(1, 'day')) 
(PARTITION part_t01 VALUES LESS THAN(TO_DATE('2018-11-01', 'YYYY-MM-DD')));
```
**References**:<br />
https://www.cnblogs.com/yuxiaole/p/9809294.html

##
**Q16**: ORA-00600: internal error **code**, arguments: [], [], [], [], []<br />
**Solution**: ORA-00600是oracle的一个尚未解决的bug，通常会出现在建表的时候，
目前查到的解决方法是在最后加上WHERE rownum > -1 <br />
**References**:<br />
https://stackoverflow.com/questions/52565289/execution-28-6-ora-00600-internal-error-**code**-arguments

##
**Q17**: oracle执行存储过程报错ORA-01031:权限不足<br />
**Solution**: 在oracle存储过程中，默认是可以直接执行DML和DQL的，但是执行CREATE TABLE这种的DDL则需要借助EXECUTE IMMEDIATE ···了。
CREATE TABLE想使用CREATE ANY TABLE权限，而CREATE ANY TABLE权限来自DBA角色，默认情况下，虽然在会话环境中可见，
但在存储过程中不可见（无效），需要显式授权<br />
**Code**:
```sql
GRANT CREATE ANY TABLE TO username;
```
**References**:<br />
https://www.cnblogs.com/zjfjava/p/9057779.html

## 
**Q18**: 通过python从oracle下载数据<br />
**Code**:
```python
import cx_Oracle
import csv


def oracle_to_csv(table_name_or_sql_str, output_file_path, username, password, host, port, service_name, 
                  is_sql_str=False):
    """
    oracle下载数据到csv
    :param table_name_or_sql_str: 表名或者sql语句
    :param output_file_path: csv文件名
    :param username: oracle用户名
    :param password: oracle密码
    :param host: oracle主机地址
    :param port: oracle端口号
    :param service_name: oracle服务名
    :param is_sql_str: 是否是sql语句
    :return: 
    """
    db = cx_Oracle.connect(username, password, '{}:{}/{}'.format(host, port, service_name))

    print_header = True
    output_file = open(output_file_path, 'w')
    output = csv.writer(output_file, dialect='excel')
    if not is_sql_str:
        sql = 'select * from {}'.format(table_name_or_sql_str)
    else:
        sql = table_name_or_sql_str

    cr = db.cursor()
    cr.execute(sql)

    if print_header:
        cols = [ele[0] for ele in cr.description]
        output.writerow(cols)

    for row_data in cr:
        output.writerow(row_data)
    output_file.close()

    db.close()
```
**References**:<br />
Not found

##
**Q19**: 通过python上传数据到oracle<br />
**Solution**: dask的to_sql函数，一方面可以实现上传的功能，
另一方面可以看到上传进度条。<br />
**Code**: 
```python
import pandas as pd
import dask.dataframe as dd
from dask.diagnostics import ProgressBar
from sqlalchemy import create_engine
from sqlalchemy.types import VARCHAR


def csv_to_oracle(file_name, table_name, username, password, host, port, service_name, npartitions=10, dtype=None):
    """
    上传csv文件到oracle
    :param file_name: csv文件名称
    :param table_name: oracle表名
    :param username: oracle用户名
    :param password: oracle密码
    :param host: oracle主机地址
    :param port: oracle端口号
    :param service_name: oracle服务名
    :param npartitions: 上传文件的拆分数
    :param dtype: 上传到oracle的数据类型, 例如dtype={'id': VARCHAR(11)}
    :return: 
    """
    engine = create_engine('oracle+cx_oracle://{}:{}@{}:{}/?service_name={}'.format(username, password, host, port,
                                                                                    service_name))
    conn = engine.raw_connection()

    df_result = pd.read_csv(file_name)
    dd_result_all = dd.from_pandas(df_result, npartitions=npartitions)
    
    pbar_rows = 0

    for i in range(dd_result_all.npartitions):
        dd_partition = dd_result_all.get_partition(i)
        with ProgressBar():
            dd_partition.to_sql(table_name, engine, if_exists='append', index=False, dtype=dtype)
        pbar_rows += len(dd_partition)
        all_rows = len(dd_result_all)
        pbar_rate = round(pbar_rows / all_rows, 2) * 100
        print('python logging: [{}%({}/{}) rows insert finished]'.format(pbar_rate, pbar_rows, all_rows))
    conn.close()
```
**References**:<br />
https://stackoverflow.com/questions/62404502/using-dasks-new-to-sql-for-improved-efficiency-memory-speed-or-alternative-to

**Q20**: oracle得到两个日期相差的分钟数<br />
**Solution**: 需要注意的是，两个日期相减默认得到的是天数，因此\*24\*6可以得到分钟数；
其次，24h制的时分秒表达是HH24:MI:SS，而不是HH:MM:SS。<br />
**Code**:
```sql
SELECT ROUND(TO_NUMBER(TO_DATE('2011-10-12 15:10:00','YYYY-MM-DD HH24:MI:SS')-TO_DATE('2011-10-12 14:23:00','YYYY-MM-DD HH24:MI:SS'))*24*60) FROM DUAL;
```
**References**:<br />
https://zhidao.baidu.com/question/1307148614295706059.html

**Q21**: 如何去掉oracle代码里的注释<br />
**Solution**: 这里需要分情况讨论，假如想删掉所有注释，甚至包括不是注释的
/*+ PARALLEL NOLOGGING */，那么可以用sqlparse；如果只是想删掉
某些的字段注释，比如通过--注释的部分，可以用正则表达式替换。<br />
**Code**:
```python
import sqlparse
import re


def rm_sql_comments(file_in, file_out, rm_all_comments=False):
    """

    :param file_in: 带注释的sql文件
    :param file_out: 去除注释的sql文件
    :param rm_all_comments: 是否删掉所有注释
    :return:
    """
    with open(file_out, 'w', encoding='utf-8') as f_out:
        with open(file_in, encoding='utf-8') as f_in:
            sql_codes = f_in.read()
            if rm_all_comments:
                f_out.write(sqlparse.format(sql_codes, strip_comments=True))
            else:
                # 将--到换行符\n之间的所有内容替换成\n，其中.表示任意字符，*表示任意次，?表示非贪婪模式
                f_out.write(re.sub(re.compile(r'--.*?\n'), '\n', sql_codes))
```
**References**:<br />
Not found

##
**Q22**: oracle中connect by level得到连续的月份<br />
**Code**:
```sql
-- 得到从2018年1月31号起两年的月份
SELECT ADD_MONTHS(TO_DATE('2018-01-31', 'YYYY-MM-DD'), level - 1) m
  FROM dual
CONNECT BY LEVEL <=
           MONTHS_BETWEEN(TO_DATE('2018-01-31', 'YYYY-MM-DD'),
                          ADD_MONTHS(TO_DATE('2018-01-31', 'YYYY-MM-DD'), -24))
  ORDER BY m;
```
**References**:<br />
https://www.cnblogs.com/jason2018524/p/10288258.html

##
**Q23**: oracle中delete之后如何恢复数据<br />
**Code**:
```sql
-- 1.假如DELETE之后没有执行COMMIT，则只需要执行ROLLBACK；
-- 2.假如是通过TRUNCATE操作进行删除则无法恢复；
-- 3.假如DELETE之后执行了COMMIT，则可以通过以下方式进行恢复，其中下面的时间点是执行delete之前的
--   某个时间点，如果记不起来可以多试几次看看。
SELECT * FROM table_name AS OF TIMESTAMP TO_TIMESTAMP('2016-08-11 16:12:11', 'yyyy-mm-dd hh24:mi:ss');
ALTER TABLE table_name ENABLE ROW MOVEMENT;
FLASHBACK TABLE table_name TO TIMESTAMP TO_TIMESTAMP('2016-08-11 16:12:11', 'yyyy-mm-dd hh24:mi:ss');
```
**References**:<br />
https://blog.csdn.net/zl834205311/article/details/86605494

##
**Q24**: oracle基于已经存在的表建一个空表<br />
**Code**:
```sql
CREATE TABLE new_table
  AS (SELECT *
      FROM old_table WHERE 1=2);
```
**References**:<br />
https://www.techonthenet.com/oracle/tables/create_table2.php#:~:text=Answer%3A%20To%20do%20this%2C%20the,companies%20WHERE%201%3D2)%3B

##
**Q25**: oracle将一个大表拆分成几个小表<br />
**Code**:
```sql
-- 例如我们需要将一个大表拆分成2个小表便于数据同步，下面通过一个完整的例子来阐述这一过程
-- 1.建立小表
CREATE TABLE small_table1
  AS (SELECT t.*, num
      FROM big_table t WHERE 1=2);

CREATE TABLE small_table2
  AS (SELECT t.*, num
      FROM big_table t WHERE 1=2);

-- 2.插入数据
INSERT ALL
WHEN num=1 THEN INTO small_table1
WHEN num=2 THEN INTO small_table2
SELECT t.*, NTILE OVER (ORDER BY NULL) AS num
FROM big_table t;

-- 3.删除num列
ALTER TABLE small_table1 DROP COLUMN num;
ALTER TABLE small_table2 DROP COLUMN num;
```
**References**:<br />
https://stackoverflow.com/questions/36335406/sql-how-would-you-split-a-100-000-records-from-a-oracle-table-into-5-chunks

##
**Q26**: oracle not in查不出数据的坑<br />
**Code**:
```sql
-- 当B表的id中有null时,此时数据查询不到 
-- 错误使用方法
SELECT *
  FROM A a
  WHERE a.id NOT IN
    (SELECT b.id
          FROM B b);

-- 正确使用方法
SELECT *
  FROM A a
  WHERE a.id NOT IN
    (SELECT b.id
          FROM B b
           WHERE b.id IS NOT NULL
     );
```
**References**:<br />
https://blog.csdn.net/qq_37406548/article/details/90406874

##
**Q27**: oracle将某列的值合并拼接为字符串<br />
**Code**:
```sql
-- 对每个id将name列按照dt的顺序进行合并
SELECT id, LISTAGG(name, ' ') WITHIN GROUP (ORDER BY dt) AS names
FROM table_name GROUP BY id;
```
**References**:<br />
https://stackoverflow.com/questions/4686543/sql-query-to-concatenate-column-values-from-multiple-rows-in-oracle

##
**Q28**: oracle中WM_CONCAT函数的使用<br />
**Code**:
```sql
# 统计每个客户所有购买的产品种类集合
-- 输入:
   cust_no, product_type
   -------  -----------
   1,       b
   1,       a
   1,       a
   2,       c
   2,       c
   2,       a
   2,       b 
-- 输出:
   cust_no, product_type_set
   -------  ---------------
   1,       a,b
   2,       a,b,c

SELECT cust_no, CAST(WM_CONCAT(DISTINCT product_type) AS VARCHAR2(100)) AS product_type_set 
FROM table_name GROUP BY cust_no;
```
**References**:<br />
http://www.dba-oracle.com/t_wm_concat_sql_function.htm

##
**Q29**: 如何恢复oracle中误删除表(drop掉的)？<br />
**Code**:
```sql
FLASHBACK TABLE test_drop TO BEFROE DROP;
```
**References**:<br />
https://blog.csdn.net/zhupanlinch/article/details/103025453

# TO BE CONTINUE
## License
[MIT](https://choosealicense.com/licenses/mit/)