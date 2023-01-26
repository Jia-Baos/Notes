# MySQL

## MySQL之最基本的命令

| 操作 | Description |
| :---- | :---- |
| 创建数据库 | CREATE DATABASE 数据库名; |
| 查看数据库 | SHOW DATABASES; |
| 指定要操作的数据库 | USE 数据库名; |
| 创建数据表 | CREATE TABLE 数据表名; |
| 查看数据表 | SHOW TABLES; |
| 使用DESCRIBE语句查看数据表 | DESCRIBE 数据表名; |
| 查看数据表所有内容 | SELECT * FROM 数据表名;|
| 为数据表重命名 | ALTER TABLE 数据表名 RENAME 新表名; |
| 修改字段名 | ALTER TABLE 数据表名 CHANGE 旧字段名 新字段名 新数据类型; |
| 修改字段数据类型  | ALTER TABLE 数据表名 MODIFY 字段名 数据类型; |
| 修改字段的排列位置 | ALTER TABLE 数据表名 MODIFY 字段名1 数据类型 AFTER 字段名2; |
| 添加字段 | ALTER TABLE 数据表名 ADD 字段名 数据类型; |
| 删除字段 | ALTER TABLE 数据表名 DROP 字段名; |
| 删除数据表 | DROP TABLE 数据表名; |

```MYSQL
CREATE TABLE student (student_name VARCHAR(255) NOT NULL,student_id INT NOT NULL AUTO_INCREMENT,PRIMARY KEY(student_id));
```

## MySQL数据类型

### 数值类型

1. TINYINT（1 Bytes）
2. SAMLLINT（2 Bytes）
3. MEDIUMINT（3 Bytes）
4. INT或INTEGER（4 Bytes）
5. BIGINT（8 Bytes）
6. FLOAT（4 Bytes）
7. DOUBLE（8 Bytes）
8. DECIMAL（对DECIMAL(M,D) ，如果M>D，为M+2否则为D+2）

### 日期和时间类型

1. DATE（YYYY-MM-DD）
2. TIME（HH:MM:SS）
3. YEAR（YYYY）
4. DATETIME（YYYY-MM-DD hh:mm:ss）
5. TIMESTAMP（YYYY-MM-DD hh:mm:ss）

### 字符串类型

1. CHAR（0-255 bytes）
2. VARCHAR（0-65535 bytes）
3. TINYBLOB（0-255 bytes）
4. TINYTEXT（0-255 bytes）
5. BLOB（0-65 535 bytes，二进制形式的长文本数据）
6. TEXT（0-65 535 bytes，普通长文本数据）
7. MEDIUMBLOB（0-16 777 215 bytes）
8. MEDIUMTEXT（0-16 777 215 bytes）
9. LONGBLOB（0-4 294 967 295 bytes）
10. LONGTEXT（0-4 294 967 295 bytes）

## MySQL约束总结

1. NOT NULL：非空约束，用于约束该字段的值不能为空。比如姓名、学号等。
2. DEFAULT：默认值约束，用于约束该字段有默认值，约束当数据表中某个字段不输入值时，自动为其添加一个已经设置好的值。比如性别。
3. PRIMARY KEY：主键约束，用于约束该字段的值具有唯一性，至多有一个，可以没有，并且非空。比如学号、员工编号等。
4. UNIQUE：唯一约束，用于约束该字段的值具有唯一性，可以有多个，可以没有，可以为空。比如座位号。
5. CHECK：检查约束，用来检查数据表中，字段值是否有效。比如年龄、性别。
6. FOREIGN KEY：外键约束，外键约束经常和主键约束一起使用，用来确保数据的一致性，用于限制两个表的关系，用于保证该字段的值必须来自于主表的关联列的值。在从表添加外键约束，用于引用主表中某列的值。比如学生表的专业编号，员工表的部门编号，员工表的工种编号。

主要归类为列级约束和表级约束

1. 列级约束：NOT NULL、DEFAULT、PRIMARY KEY、 UNIQUE、CHECK
2. 表级约束：PRIMARY KEY、UNIQUE、CHECK、FOREIGN KEY

## 增

1. INSERT 语句中指定所有字段名

    ```MYSQL
    INSERT INTO 数据表名 (字段名1,字段名2,字段名3...) VALUES(值1,值2,值3...);
    ```

2. INSERT 语句中不指定字段名

   若不指定字段名，则添加的值的顺序应和字段在表中的顺序完全一致。

    ```MYSQL
    INSERT INTO 数据表名 VALUES(值1,值2,值3...);
    ```

3. 同时添加多条数据

    ```MYSQL
    INSERT INTO 数据表名 VALUES(值1,值2,值3...),VALUES(值1,值2,值3...),VALUES(值1,值2,值3...);
    ```

4. 为表的指定字段添加数据

    为指定字段添加数据，即只向部分字段添加值，而其他字段的值为表定义时的默认值。

    ```MYSQL
    INSERT INTO 数据表名 (字段名2,字段名3...) VALUES(值2,值3...);
    ```

5. INSERT 语句的其它写法

   ```MYSQL
    INSERT INTO 数据表名 SET 字段名1=值1,字段名2=值2;
    ```

## 删

1. 删除部分数据

    即删除指定的部分数据，需要使用 WHERE 子句来指定删除记录的条件。

    ```MYSQL
    DELETE FROM 数据表名 WHERE 字段名1=值1;
    ```

2. 删除全部数据

    ```MYSQL
    DELETE FROM 数据表名;
    ```

3. 删除全部数据的另一种方法——TRUNCATE

    ```MYSQL
    TRUNCATE FROM 数据表名;
    ```

    注意：
    1. DELETE 后面可以跟 WHERE 子句指定删除部分记录，TRUNCATE 只能删除整个表的所有记录；
    2. 使用 TRUNCAT E语句删除记录后，新添加的记录时，自动增长字段（如本文中 student 表中的 id 字段）会默认从 1 开始，而使用 DELETE 删除记录后，新添加记录时，自动增长字段会从删除时该字段的的最大值加 1 开始计算（即原来的 id 最大为 5，则会从 6 开始计算）。所以如果是想彻底删除一个表的记录而且不会影响到重新添加记录，最好使用 TRUNCATE 来删除整个表的记录。

## 改

1. 更新部分数据

    指更新指定表中的指定记录，使用 WHERE 子句来指定。

    ```MYSQL
    UPDATE 数据表名 SET 字段名1=值1,字段名2=值2 WHERE 字段名3=值3;
    ```

2. 更新全部数据

    在 UPDATE 语句中若不使用 WHERE 子句，则会将表中所有记录的指定字段都进行更新。

    ```MYSQL
    UPDATE 数据表名 SET 字段名1=值1,字段名2=值2;
    ```

## 查

1. 查询所有字段

    ```MYSQL
    SELECT 字段名1,字段名2.. FROM 数据表名;
    ```

2. 带关系运算的符查询

    ```MYSQL
    SELECT 字段名1,字段名2.. FROM 数据表名 WHERE 字段名3=值3;
    ```

3. 带 IN 关键字的查询

    IN 关键字用于判断某个字段的值是否在指定集合中，若在，则该字段所在的记录将会被查询出来。

    ```MYSQL
    SELECT * FROM 数据表名 WHERE id IN (1,2,3);
    ```

4. 带 BETWEEN AND 关键字的查询

    BETWEEN AND 用于判断某个字段的值是否在指定范围之内，若在，则该字段所在的记录会被查询出来，反之不会。

    ```MYSQL
    SELECT 字段名1,字段名2 FROM 数据表名 WHERE id BETWEEN 2 AND 5;
    ```

5. 空值查询

    在数据表中有些值可能为空值（NULL），空值不同于 0，也不同于空字符串，需要使用 IS NULL 来判断字段的值是否为空值。

    ```MYSQL
    SELECT * FROM 数据表名 WHERE id IS NULL;
    ```

6. 带 DISTINCT 关键字的查询

    很多表中某些字段的数据存在重复的值，可以使用 DISTINCT 关键字来过滤重复的值，只保留一个值。

    ```MYSQL
    SELECT DISTINCT 字段名1 FROM 数据表名;
    ```

7. 带 LIKE 关键字的查询

    ```MYSQL
    SELECT 字段名1,字段名2 FROM 数据表名 WHERE 字段名2 LIKE '匹配字符串';   // 可以结合百分号、下划线通配符使用
    ```

8. 带 AND 关键字的多条件查询

    在使用 SELECT 语句查询数据时，优势为了使查询结果更加精确，可以使用多个查询条件，如使用 AND 关键字可以连接两个或多个查询条件。

    ```MYSQL
    SELECT 字段名1,字段名2.. FROM 数据表名 WHERE 字段名3=值3 AND 字段名4=值4;
    ```

9. 带 OR 关键字的多条件查询

    在使用 SELECT 语句查询数据时，优势为了使查询结果更加精确，可以使用多个查询条件，如使用 AND 关键字可以连接两个或多个查询条件。

    ```MYSQL
    SELECT 字段名1,字段名2.. FROM 数据表名 WHERE 字段名3=值3 OR 字段名4=值4;
    ```

## 高级查询

| 函数名称 | 作用 |
| :--- | :--- |
| COUNT() | 返回某列的行数 |
| SUM() | 返回某列值的和 |
| AVG() | 返回某列的平均值 |
| MAX() | 返回某列的最大值 |
| MIN() | 返回某列的最小值 |

1. 统计记录的条数

    ```MYSQL
    SELECT COUNT(*) FROM 数据表名;
    ```

2. 对查询结果进行排序

    在该语法中指定的字段名是对查询结果进行排序的依据，ASC 表示升序排列，DESC 表示降序排列，默认情况是升序排列。

    ```MYSQL
    SELECT * FROM 数据表名 ORDER BY 字段名1;
    ```

3. 使用 LIMIT 限制查询结果的数量

    在此语法中，LIMIT 后面可以跟两个参数，第一个参数“OFFSET”表示偏移量，如果偏移量为0，则从查询结果的第一条记录开始，偏移量为1则从查询结果中的第二条记录开始，以此类推。OFFSET 为可选值，默认值为 0，第二个参数“记录数”表示指定返回查询记录的条数。

    ```MYSQL
    SELECT * FROM 数据表名 LIMIT 4;
    ```
