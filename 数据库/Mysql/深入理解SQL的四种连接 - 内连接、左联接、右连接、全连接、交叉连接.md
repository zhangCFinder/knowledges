
**SQL连接可以分为内连接、外连接、交叉连接。**


**数据库数据：**
![c7c66c385fb344d12b6e0e9fa7e71272](深入理解SQL的四种连接 - 内连接、左联接、右连接、全连接、交叉连接.resources/AFE71129-DF37-4CF0-B5AC-D7118F51746C.png)


## 1.内连接

内连接查询操作列出与连接条件匹配的数据行，它使用比较运算符比较被连接列的列值。
### 隐式的内连接
没有INNER JOIN，形成的中间表为两个表的**笛卡尔积**。
```sql

select * from book as a,stu as b where a.sutid = b.stuid

select * from book as a inner join stu as b on a.sutid = b.stuid
```

内连接可以使用上面两种方式，其中第二种方式的inner可以省略。
![c519673b64ab86f137a263b3477567cd](深入理解SQL的四种连接 - 内连接、左联接、右连接、全连接、交叉连接.resources/7A585241-EB48-4CBF-8A04-1743D84950A3.png)

其连接结果如上图，是按照a.stuid = b.stuid进行连接。

### 显式的内连接
一般称为内连接，有INNER JOIN，形成的中间表为两个表经过ON条件过滤后的**笛卡尔积**。

```sql
SELECT O.ID,O.ORDER_NUMBER,C.ID,C.NAMEFROM CUSTOMERS C INNER JOIN ORDERS O ON C.ID=O.CUSTOMER_ID;
```


## 2.外连接

### 2.1 左联接

以左表为基准，将a.stuid = b.stuid的数据进行连接，然后将左表没有的对应项显示，右表的列为NULL

```sql

select * from book as a left join stu as b on a.sutid = b.stuid
```
![10c23da11e99408d7aa5f674a981db00](深入理解SQL的四种连接 - 内连接、左联接、右连接、全连接、交叉连接.resources/22B2F416-2D04-4919-BEB2-6DF6E53B4346.png)

### 2.2.右连接
以右表为基准，将a.stuid = b.stuid的数据进行连接，然以将右表没有的对应项显示，左表的列为NULL
```sql
select * from book as a right join stu as b on a.sutid = b.stuid
```
![15143817a33e9b022a4575d95ac5a3ce](深入理解SQL的四种连接 - 内连接、左联接、右连接、全连接、交叉连接.resources/1708621C-AAFD-4E77-B1FF-C8ABFB85C110.png)

### 2.3.全连接
完整外部联接返回左表和右表中的所有行。当某行在另一个表中没有匹配行时，则另一个表的选择列表列包含空值。如果表之间有匹配行，则整个结果集行包含基表的数据值。
```sql
select * from book as a full outer join stu as b on a.sutid = b.stuid
```
![439b527eea253b3eb82976a4593529c9](深入理解SQL的四种连接 - 内连接、左联接、右连接、全连接、交叉连接.resources/D14D4C45-F137-433B-BD95-FDB45498E6B2.png)

## 3.交叉连接

交叉连接：交叉联接返回左表中的所有行，左表中的每一行与右表中的所有行组合。

有两种，显式的和隐式的，不带ON子句，返回的是两表的乘积，也叫笛卡尔积。。

### 隐式的交叉连接,没有CROSS JOIN
```sql
SELECT O.ID, O.ORDER_NUMBER, C.ID, C.NAMEFROM ORDERS O , CUSTOMERS CWHERE O.ID=1;
```
### 显式的交叉连接，使用CROSS JOIN
```sql
select * from book as a cross join stu as b order by a.id
```
![37fa0ef1f2f6778f3e217322f1bf09ff](深入理解SQL的四种连接 - 内连接、左联接、右连接、全连接、交叉连接.resources/5F3BC0E7-0FFD-426D-8AAD-49A5037A6192.png)

