### NOSQL和SQL的区别？

- SQL数据库，指关系型数据库 - 主要代表：SQL Server，Oracle，MySQL(开源)，PostgreSQL(开源)。关系型数据库存储结构化数据。这些数据逻辑上以行列二维表的形式存在，每一列代表数据的一种属性，每一行代表一个数据实体。
- NoSQL指非关系型数据库 ，主要代表：MongoDB，Redis。NoSQL 数据库逻辑上提供了不同于二维表的存储方式，存储方式可以是JSON文档、哈希表或者其他方式。

选择 SQL vs NoSQL，考虑以下因素。

> ACID vs BASE

关系型数据库支持 ACID 即原子性，一致性，隔离性和持续性。相对而言，NoSQL 采用更宽松的模型 BASE ， 即基本可用，软状态和最终一致性。

从实用的角度出发，我们需要考虑对于面对的应用场景，ACID 是否是必须的。比如银行应用就必须保证 ACID，否则一笔钱可能被使用两次；又比如社交软件不必保证 ACID，因为一条状态的更新对于所有用户读取先后时间有数秒不同并不影响使用。

对于需要保证 ACID 的应用，我们可以优先考虑 SQL。反之则可以优先考虑 NoSQL。

> 扩展性对比

NoSQL数据之间无关系，这样就非常容易扩展，也无形之间，在架构的层面上带来了可扩展的能力。比如 redis 自带主从复制模式、哨兵模式、切片集群模式。

相反关系型数据库的数据之间存在关联性，水平扩展较难 ，需要解决跨服务器 JOIN，分布式事务等问题。

### 数据库三大范式是什么？

**第一范式（1NF）：要求数据库表的每一列都是不可分割的原子数据项。**

举例说明：

![img](https://cdn.xiaolincoding.com//picgo/1218459-20180909201651535-1215699096.png)

在上面的表中，“家庭信息”和“学校信息”列均不满足原子性的要求，故不满足第一范式，调整如下：

![img](https://cdn.xiaolincoding.com//picgo/1218459-20180909202243826-1032549277.png)

可见，调整后的每一列都是不可再分的，因此满足第一范式（1NF）；

**第二范式（2NF）：在1NF的基础上，非码属性必须完全依赖于候选码（在1NF基础上消除非主属性对主码的部分函数依赖）**

**第二范式需要确保数据库表中的每一列都和主键相关，而不能只与主键的某一部分相关（主要针对联合主键而言）。**

举例说明：

![img](https://cdn.xiaolincoding.com//picgo/1218459-20180909204750951-639647799.png)

在上图所示的情况中，同一个订单中可能包含不同的产品，因此主键必须是“订单号”和“产品号”联合组成，

但可以发现，产品数量、产品折扣、产品价格与“订单号”和“产品号”都相关，但是订单金额和订单时间仅与“订单号”相关，与“产品号”无关，

这样就不满足第二范式的要求，调整如下，需分成两个表：

![img](https://cdn.xiaolincoding.com//picgo/1218459-20180909210444227-1008056975.png)

![img](https://cdn.xiaolincoding.com//picgo/1218459-20180909210458847-2092897116.png)

**第三范式（3NF）：在2NF基础上，任何非主[属性 (opens new window)](https://baike.baidu.com/item/属性)不依赖于其它非主属性（在2NF基础上消除传递依赖）**

**第三范式需要确保数据表中的每一列数据都和主键直接相关，而不能间接相关。**

举例说明：

![img](https://cdn.xiaolincoding.com//picgo/1218459-20180909211311408-1364899740.png)

上表中，所有属性都完全依赖于学号，所以满足第二范式，但是“班主任性别”和“班主任年龄”直接依赖的是“班主任姓名”，

而不是主键“学号”，所以需做如下调整：

![img](https://cdn.xiaolincoding.com//picgo/1218459-20180909211539242-1391100354.png)

![img](https://cdn.xiaolincoding.com//picgo/1218459-20180909211602202-1069383439.png)

这样以来，就满足了第三范式的要求。

### MySQL 怎么连表查询？

数据库有以下几种联表查询类型：

1. **内连接 (INNER JOIN)**
2. **左外连接 (LEFT JOIN)**
3. **右外连接 (RIGHT JOIN)**
4. **全外连接 (FULL JOIN)**

![img](https://cdn.xiaolincoding.com//picgo/1721710415166-eff24e6c-555c-436c-b1b8-7c6dbb5850d7.webp)

**1. 内连接 (INNER JOIN)**

内连接返回两个表中有匹配关系的行。**示例**:

```sql
SELECT employees.name, departments.name
FROM employees
INNER JOIN departments
ON employees.department_id = departments.id;
```

这个查询返回每个员工及其所在的部门名称。

**2. 左外连接 (LEFT JOIN)**

左外连接返回左表中的所有行，即使在右表中没有匹配的行。未匹配的右表列会包含NULL。**示例**:

```sql
SELECT employees.name, departments.name
FROM employees
LEFT JOIN departments
ON employees.department_id = departments.id;
```

这个查询返回所有员工及其部门名称，包括那些没有分配部门的员工。

**3. 右外连接 (RIGHT JOIN)**

右外连接返回右表中的所有行，即使左表中没有匹配的行。未匹配的左表列会包含NULL。**示例**:

```sql
SELECT employees.name, departments.name
FROM employees
RIGHT JOIN departments
ON employees.department_id = departments.id;
```

这个查询返回所有部门及其员工，包括那些没有分配员工的部门。

**4. 全外连接 (FULL JOIN)**

全外连接返回两个表中所有行，包括非匹配行，在MySQL中，FULL JOIN 需要使用 UNION 来实现，因为 MySQL 不直接支持 FULL JOIN。**示例**:

```sql
SELECT employees.name, departments.name
FROM employees
LEFT JOIN departments
ON employees.department_id = departments.id

UNION

SELECT employees.name, departments.name
FROM employees
RIGHT JOIN departments
ON employees.department_id = departments.id;
```

这个查询返回所有员工和所有部门，包括没有匹配行的记录。

### MySQL如何避免重复插入数据？

**方式一：使用UNIQUE约束**

在表的相关列上添加UNIQUE约束，确保每个值在该列中唯一。例如：

```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) UNIQUE,
    name VARCHAR(255)
);
```

如果尝试插入重复的email，MySQL会返回错误。

**方式二：使用INSERT ... ON DUPLICATE KEY UPDATE**

这种语句允许在插入记录时处理重复键的情况。如果插入的记录与现有记录冲突，可以选择更新现有记录：

```sql
INSERT INTO users (email, name) 
VALUES ('example@example.com', 'John Doe')
ON DUPLICATE KEY UPDATE name = VALUES(name);
```

**方式三：使用INSERT IGNORE**： 该语句会在插入记录时忽略那些因重复键而导致的插入错误。例如：

```sql
INSERT IGNORE INTO users (email, name) 
VALUES ('example@example.com', 'John Doe');
```

如果email已经存在，这条插入语句将被忽略而不会返回错误。

选择哪种方法取决于具体的需求：

- 如果需要保证全局唯一性，使用UNIQUE约束是最佳做法。
- 如果需要插入和更新结合可以使用`ON DUPLICATE KEY UPDATE`。
- 对于快速忽略重复插入，`INSERT IGNORE`是合适的选择。

### CHAR 和 VARCHAR有什么区别？

- CHAR是固定长度的字符串类型，定义时需要指定固定长度，存储时会在末尾补足空格。CHAR适合存储长度固定的数据，如固定长度的代码、状态等，存储空间固定，对于短字符串效率较高。
- VARCHAR是可变长度的字符串类型，定义时需要指定最大长度，实际存储时根据实际长度占用存储空间。VARCHAR适合存储长度可变的数据，如用户输入的文本、备注等，节约存储空间。

###  varchar后面代表字节还是会字符？

`VARCHAR` 后面括号里的数字代表的是字符数，而不是字节数。

比如 `VARCHAR(10)`，这里的 10 表示该字段最多可以存储 10 个字符。字符的字节长度取决于所使用的字符集。

- 如果字符集是 ASCII 字符集：ASCII 字符集每个字符占用 1 个字节，那么 VARCHAR(10) 最多可以存储 10 个 ASCII 字符，同时占用的存储空间最多为 10 个字节（不考虑额外的长度记录开销）。
- 如果字符集是 UTF - 8 字符集，它的每个字符可能占用 1 到 4 个字节，对于 `VARCHAR(10)` 的字段，它最多可以存储 10 个字符，但占用的字节数会根据字符的不同而变化。

### int(1) int(10) 在mysql有什么不同？

`INT(1)` 和 `INT(10)` 的区别主要在于 **显示宽度**，而不是存储范围或数据类型本身的大小。以下是核心区别的总结：

- 本质是显示宽度，不改变存储方式：`INT` 的存储固定为 4 字节，所有 `INT`（无论写成 `INT(1)` 还是 `INT(10)`）占用的存储空间 均为 4 字节。括号内的数值（如 `1` 或 `10`）是显示宽度，用于在 特定场景下 控制数值的展示格式。
- 唯一作用场景：`ZEROFILL` 补零显示，当字段设置 `ZEROFILL` 时：数字显示时会用前导零填充至指定宽度。比如，字段类型为 `INT(4) ZEROFILL`，实际存入 `5` → 显示为 `0005`，实际存入 `12345` → 显示仍为 `12345`（宽度超限时不截断）。

举一个例子

```java
-- 创建一个包含 INT(1) 和 INT(10) 字段的表，并设置 ZEROFILL 属性
CREATE TABLE test_int (
    num1 INT(1) ZEROFILL,
    num2 INT(10) ZEROFILL
);

-- 插入数据
INSERT INTO test_int (num1, num2) VALUES (1, 1);

-- 查询数据
SELECT * FROM test_int;
```

结果分析：

- `num1` 字段由于设置为 `INT(1) ZEROFILL`，其显示宽度为 1，插入数据 `1` 时会显示为 `1`。
- `num2` 字段设置为 `INT(10) ZEROFILL`，显示宽度为 10，插入数据 `1` 时会在前面填充零，显示为 `0000000001`。

### Text数据类型可以无限大吗？

MySQL 3 种text类型的最大长度如下：

- TEXT：65,535 bytes ~64kb
- MEDIUMTEXT：16,777,215 bytes ~16Mb
- LONGTEXT：4,294,967,295 bytes ~4Gb

### IP地址如何在数据库里存储？

IPv4 地址是一个 32 位的二进制数，通常以点分十进制表示法呈现，例如 `192.168.1.1`。

**字符串类型的存储方式**：直接将 IP 地址作为字符串存储在数据库中，比如可以用 `VARCHAR(15)`来存储。

```java
-- 创建一个表，使用VARCHAR类型存储IPv4地址
CREATE TABLE ip_records (
    id INT AUTO_INCREMENT PRIMARY KEY,
    ip_address VARCHAR(15)
);

-- 插入数据
INSERT INTO ip_records (ip_address) VALUES ('192.168.1.1');
```

- **优点**：直观易懂，方便直接进行数据的插入、查询和显示，不需要进行额外的转换操作。
- **缺点**：占用存储空间较大，字符串比较操作的性能相对较低，不利于进行范围查询。

整数类型的存储方式：将 IPv4 地址转换为 32 位无符号整数进行存储，常用的数据类型有 `INT UNSIGNED`。

```java
-- 创建一个表，使用INT UNSIGNED类型存储IPv4地址
CREATE TABLE ip_records (
    id INT AUTO_INCREMENT PRIMARY KEY,
    ip_address INT UNSIGNED
);

-- 插入数据，需要先将IP地址转换为整数
INSERT INTO ip_records (ip_address) VALUES (INET_ATON('192.168.1.1'));

-- 查询时将整数转换回IP地址
SELECT INET_NTOA(ip_address) FROM ip_records;
```

- **优点**：占用存储空间小，整数比较操作的性能较高，便于进行范围查询。
- **缺点**：需要进行额外的转换操作，不够直观，增加了开发的复杂度。

### 说一下外键约束

外键约束的作用是维护表与表之间的关系，确保数据的完整性和一致性。让我们举一个简单的例子：

假设你有两个表，一个是学生表，另一个是课程表，这两个表之间有一个关系，即一个学生可以选修多门课程，而一门课程也可以被多个学生选修。在这种情况下，我们可以在学生表中定义一个指向课程表的外键，如下所示：

```text
CREATE TABLE students (
  id INT PRIMARY KEY,
  name VARCHAR(50),
  course_id INT,
  FOREIGN KEY (course_id) REFERENCES courses(id)
);
```

这里，`students`表中的`course_id`字段是一个外键，它指向`courses`表中的`id`字段。这个外键约束确保了每个学生所选的课程在`courses`表中都存在，从而维护了数据的完整性和一致性。

如果没有定义外键约束，那么就有可能出现学生选了不存在的课程或者删除了一个课程而忘记从学生表中删除选修该课程的学生的情况，这会破坏数据的完整性和一致性。因此，使用外键约束可以帮助我们避免这些问题。

### MySQL的关键字in和exist

在MySQL中，`IN` 和 `EXISTS` 都是用来处理子查询的关键词，但它们在功能、性能和使用场景上有各自的特点和区别。

> IN关键字

`IN` 用于检查左边的表达式是否存在于右边的列表或子查询的结果集中。如果存在，则`IN` 返回`TRUE`，否则返回`FALSE`。

语法结构：

```sql
SELECT column_name(s)
FROM table_name
WHERE column_name IN (value1, value2, ...);
```

或

```sql
SELECT column_name(s)
FROM table_name
WHERE column_name IN (SELECT column_name FROM another_table WHERE condition);
```

例子：

```sql
SELECT * FROM Customers
WHERE Country IN ('Germany', 'France');
```

> EXISTS关键字

`EXISTS` 用于判断子查询是否至少能返回一行数据。它不关心子查询返回什么数据，只关心是否有结果。如果子查询有结果，则`EXISTS` 返回`TRUE`，否则返回`FALSE`。

语法结构：

```sql
SELECT column_name(s)
FROM table_name
WHERE EXISTS (SELECT column_name FROM another_table WHERE condition);
```

例子：

```sql
SELECT * FROM Customers
WHERE EXISTS (SELECT 1 FROM Orders WHERE Orders.CustomerID = Customers.CustomerID);
```

区别与选择：

- **性能差异**：在很多情况下，`EXISTS` 的性能优于 `IN`，特别是当子查询的表很大时。这是因为`EXISTS` 一旦找到匹配项就会立即停止查询，而`IN`可能会扫描整个子查询结果集。
- **使用场景**：如果子查询结果集较小且不频繁变动，`IN` 可能更直观易懂。而当子查询涉及外部查询的每一行判断，并且子查询的效率较高时，`EXISTS` 更为合适。
- **NULL值处理**：`IN` 能够正确处理子查询中包含NULL值的情况，而`EXISTS` 不受子查询结果中NULL值的影响，因为它关注的是行的存在性，而不是具体值。

### mysql中的一些基本函数，你知道哪些？

> 一、字符串函数

**CONCAT(str1, str2, ...)**：连接多个字符串，返回一个合并后的字符串。

```sql
SELECT CONCAT('Hello', ' ', 'World') AS Greeting;
```

**LENGTH(str)**：返回字符串的长度（字符数）。

```sql
SELECT LENGTH('Hello') AS StringLength;
```

**SUBSTRING(str, pos, len)**：从指定位置开始，截取指定长度的子字符串。

```sql
SELECT SUBSTRING('Hello World', 1, 5) AS SubStr;
```

**REPLACE(str, from_str, to_str)**：将字符串中的某部分替换为另一个字符串。

```sql
SELECT REPLACE('Hello World', 'World', 'MySQL') AS ReplacedStr;
```

> 二、数值函数

**ABS(num)**：返回数字的绝对值。

```sql
SELECT ABS(-10) AS AbsoluteValue;
```

**POWER(num, exponent)**：返回指定数字的指定幂次方。

```sql
SELECT POWER(2, 3) AS PowerValue;
```

> 三、日期和时间函数

**NOW()**：返回当前日期和时间。

```sql
SELECT NOW() AS CurrentDateTime;
```

**CURDATE()**：返回当前日期。

```sql
SELECT CURDATE() AS CurrentDate;
```

> 四、聚合函数

**COUNT(column)**：计算指定列中的非NULL值的个数。

```sql
SELECT COUNT(*) AS RowCount FROM my_table;
```

**SUM(column)**：计算指定列的总和。

```sql
SELECT SUM(price) AS TotalPrice FROM orders;
```

**AVG(column)**：计算指定列的平均值。

```sql
SELECT AVG(price) AS AveragePrice FROM orders;
```

**MAX(column)**：返回指定列的最大值。

```sql
SELECT MAX(price) AS MaxPrice FROM orders;
```

**MIN(column)**：返回指定列的最小值。

```sql
SELECT MIN(price) AS MinPrice FROM orders;
```

### **SQL查询语句的执行顺序是怎么样的？**

![image-20240820114027032](https://cdn.xiaolincoding.com//picgo/image-20240820114027032.png)

所有的查询语句都是从FROM开始执行，在执行过程中，每个步骤都会生成一个虚拟表，这个虚拟表将作为下一个执行步骤的输入，最后一个步骤产生的虚拟表即为输出结果。

```text
(9) SELECT 
(10) DISTINCT <column>,
(6) AGG_FUNC <column> or <expression>, ...
(1) FROM <left_table> 
    (3) <join_type>JOIN<right_table>
    (2) ON<join_condition>
(4) WHERE <where_condition>
(5) GROUP BY <group_by_list>
(7) WITH {CUBE|ROLLUP}
(8) HAVING <having_condtion>
(11) ORDER BY <order_by_list>
(12) LIMIT <limit_number>;
```

