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
其中问题放在question目录，sql代码放在code目录，参考资料放在reference目录。

##
Q1: oracle不像其他sql语言一样，可以用drop table if exists table_name这种方式删表。
那么在oracle中如何实现类似的功能呢？<br />
Code:
```oracle-plsql
begin 
execute immediate 'drop table table_name';
exception 
when others then 
    if sqlcode <> -942 then 
        raise; 
        end if; 
end;
```
Reference:<br />
https://stackoverflow.com/questions/1799128/oracle-if-table-exists



## License
[MIT](https://choosealicense.com/licenses/mit/)