# Python CRUD

> 虽然在我们接触到 ORM 思想以后，就很少再用到 Python 直连 MySQL（但个人开发我还是推荐 [SQLite](https://www.sqlite.org/) 或 [PostgreSQL](https://www.postgresql.org/) 这两种关系型数据库），然后通过写 SQL 语句来操作数据库/表，但毕竟是基础，还是要了解一下的。
>
> 另外 Python 操作 MySQL 数据库的库有很多，推荐 pymysql，这也是 [骆昊](https://github.com/jackfrued) 大佬在其文章中推荐的第三方库。
>
> 首先安装 pymysql 库，使用命令 `conda install pymysql` 或 `pip install pymysql` 都可以。

## Python 直连 MySQL

```python
import pymysql

# 打开数据库连接
db_conn = pymysql.connect(host='localhost', port=3306, user='root', password='root', db='test_db')  # `localhost` 可以使用回环地址 `127.0.0.1` 代替，同时不建议大家在项目中直接使用 `root` 超级管理员账号访问数据库

# 打印数据库连接对象
print(db_conn)  # <pymysql.connections.Connection object at 0x000001BB985C14F0>

# 关闭数据库连接
db_conn.close()

```

### 查看 MySQL 版本号

```python
import pymysql

# 打开数据库连接
db_conn = pymysql.connect(host='localhost', port=3306, user='root', password='root', db='test_db')

# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db_conn.cursor()

# 使用 execute()  方法执行 SQL 查询
cursor.execute("SELECT VERSION()")

# 使用 fetchone() 方法获取单条数据.
data = cursor.fetchone()

print("Database version : %s " % data)  # Database version : 8.0.12

# 关闭游标对象
cursor.close()

# 关闭数据库连接
db_conn.close()

```

## 创建数据表

```python
import pymysql

# 打开数据库连接
db_conn = pymysql.connect(host='localhost', port=3306, user='root', password='root', db='test_db')

# 使用 cursor() 方法创建一个游标对象 cursor
cursor = db_conn.cursor()

# 使用 execute() 方法执行 SQL，如果表存在则删除
cursor.execute('DROP TABLE IF EXISTS EMPLOYEE')

# 使用预处理语句创建表
sql = '''CREATE TABLE EMPLOYEE (
         FIRST_NAME  CHAR(20) NOT NULL,
         LAST_NAME  CHAR(20),
         AGE INT,  
         SEX CHAR(1),
         INCOME FLOAT )'''

cursor.execute(sql)

# 关闭游标对象
cursor.close()

# 关闭数据库连接
db_conn.close()

```

> 说明：在 PyCharm 中书写 SQL 语句的地方，如果提示如下警告 `No data sources are configured to run this SQL and provide advanced code assistance.`，不会影响编码，但如果强迫症如我，可以 `Configure data source`，也就是添加一个数据库连接配置以消除该警告信息。
>
> ![pycharm-warning-1](./pic/pycharm-warning-1.png)
>
> ![pycharm-warning-2](./pic/pycharm-warning-2.png)
>
> ![pycharm-warning-3](./pic/pycharm-warning-3.png)

## CRUD

> 如果执行 `insert`、`delete` 或 `update` 操作，需要根据实际情况提交或回滚事务。因为创建连接时，默认开启了事务环境，在操作完成后，需要使用连接对象的 `commit` 或 `rollback` 方法，实现事务的提交或回滚，`rollback` 方法通常会放在异常捕获代码块 `except` 中。

### 插入数据

#### 插入单条记录

```python
import pymysql

# 打开数据库连接
db_conn = pymysql.connect(host='localhost', port=3306, user='root', password='root', db='test_db')

# SQL 插入语句
sql = """INSERT INTO EMPLOYEE(FIRST_NAME,
         LAST_NAME, AGE, SEX, INCOME)
         VALUES ('Mac', 'Mohan', 20, 'M', 2000)"""
try:
    # 使用 cursor() 方法创建一个游标对象 cursor
    # 通过游标对象向数据库服务器发出 SQL 语句
    with db_conn.cursor() as cursor:
        # 执行 SQL 语句
        cursor.execute(sql)
    # 提交事务到数据库执行
    db_conn.commit()
except pymysql.MySQLError as err:
    # 如果发生错误则回滚事务
    db_conn.rollback()

# 关闭数据库连接
db_conn.close()

```

> 还有一个知识点是插入多条记录，建议使用游标对象的 `executemany` 方法做批处理，后面再加。

### 更新数据

```python
import pymysql

# 打开数据库连接
with pymysql.connect(host='localhost', port=3306, user='root', password='root', db='test_db') as db_conn:
    # SQL 更新语句
    sql = "UPDATE EMPLOYEE SET AGE = AGE + 1 WHERE FIRST_NAME = %s" % 'Mac'
    try:
        # 使用 cursor() 方法创建一个游标对象 cursor
        # 通过游标对象向数据库服务器发出 SQL 语句
        with db_conn.cursor() as cursor:
            # 执行 SQL 语句
            cursor.execute(sql)
        # 提交事务到数据库执行
        db_conn.commit()
    except pymysql.MySQLError as err:
        # 如果发生错误则回滚事务
        db_conn.rollback()

```

### 删除数据

```py
import pymysql

# 打开数据库连接
with pymysql.connect(host='localhost', port=3306, user='root', password='root', db='test_db') as db_conn:
    try:
        # 使用 cursor() 方法创建一个游标对象 cursor
        # 通过游标对象向数据库服务器发出 SQL 语句
        with db_conn.cursor() as cursor:
            # 执行 SQL 语句
            affected_row = cursor.execute(
                "DELETE FROM EMPLOYEE WHERE FIRST_NAME = %s",
                'Mac'
            )
            if affected_row == 1:
                print('删除成功')
        # 提交事务到数据库执行
        db_conn.commit()
    except pymysql.MySQLError as err:
        # 如果发生错误则回滚事务
        db_conn.rollback()

```

### 查询数据

#### 查询单条记录

```python
import pymysql

# 打开数据库连接
with pymysql.connect(host='localhost', port=3306, user='root', password='root', db='test_db') as db_conn:
    try:
        # 使用 cursor() 方法创建一个游标对象 cursor
        # 通过游标对象向数据库服务器发出 SQL 语句
        with db_conn.cursor() as cursor:
            # 执行 SQL 语句
            cursor.execute("SELECT * FROM EMPLOYEE")
            row = cursor.fetchone()
            # print(row)  # 单次输出单条
            while row:  # 循环输出多条
                print(row)
                row = cursor.fetchone()
    except pymysql.MySQLError as err:
        # print(type(err), err)
        print("Error: unable to fetch data")

```

#### 查询多条记录

```py
import pymysql

# 打开数据库连接
with pymysql.connect(host='localhost', port=3306, user='root', password='root', db='test_db') as db_conn:
    try:
        # 使用 cursor() 方法创建一个游标对象 cursor
        # 通过游标对象向数据库服务器发出 SQL 语句
        with db_conn.cursor() as cursor:
            # 执行 SQL 语句
            cursor.execute("SELECT * FROM EMPLOYEE")
            rows = cursor.fetchall()
            for row in rows:
                print(row)
    except pymysql.MySQLError as err:
        # print(type(err), err)
        print("Error: unable to fetch data")

```

