# MYSQL8.0手册

## 简介
mysql8.0手册阅读

## 连接、关闭Mysql服务器
1. 远程连接
```
shell> mysql -h host -u user -p
Enter password: ********
```
2. 本机连接
```
shell> mysql -u user -p
```

## 骚操作
Here is another query. It demonstrates that you can use mysql as a simple calculator:
```
mysql> SELECT SIN(PI()/4), (4+1)*5;

+------------------+---------+
| SIN(PI()/4)      | (4+1)*5 |
+------------------+---------+
| 0.70710678118655 |      25 |
+------------------+---------+
1 row in set (0.02 sec)
```

## 基本操作

> show databases
```
mysql> SHOW DATABASES;
```

> 选择数据库
```
mysql> USE DATABASE_NAME;
```

> 创建数据库 Under [Unix, database names are case-sensitive (unlike SQL keywords)]
```
mysql> CREATE DATABASE DATABASE_NAME;
```

> 直接选择数据库
```
shell> mysql -h host -u user -p menagerie
Enter password: ********
```

> 查看数据表
```
mysql> SHOW TABLES;
```

> 创建表单
```
mysql> CREATE TABLE pet (name VARCHAR(20), owner VARCHAR(20), species VARCHAR(20), sex CHAR(1), birth DATE, death DATE);
```

> 查看表的结构
```
mysql> DESCRIBE pet;
```

> 导入文件内容到表中(mysql 5.7没成功)
```
mysql> LOAD DATA LOCAL INFILE '/path/pet.txt' INTO TABLE pet;
mysql> LOAD DATA LOCAL INFILE '/path/pet.txt' INTO TABLE pet 
-> LINES TERMINATED BY '\r\n';
```

> 插入数据
```
mysql> INSERT INTO pet VALUES ('Puffball','Diane','hamster','f','1999-03-30',NULL);
```

> 查询数据
```
SELECT what_to_select(*) FROM which_table WHERE conditions_to_satisfy;
```

> 更新数据
```
UPDATE pet SET birth = '1989-08-31' WHERE name = 'Bowser';
```

> 特定数据的查询
```
mysql> SELECT * FROM pet WHERE (species = 'cat' AND sex = 'm') OR (species = 'dog' AND sex = 'f');
```

> 特定列的查询
```
mysql> SELECT name, species, birth FROM pet WHERE species = 'dog' OR species = 'cat';
```

## 参考：
1. [https://dev.mysql.com/doc/refman/8.0/en/](https://dev.mysql.com/doc/refman/8.0/en/)
2. [http://yiyulinfeng.com/MySQL8.0/](http://yiyulinfeng.com/MySQL8.0/)