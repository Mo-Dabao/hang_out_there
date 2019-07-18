# Python3.连接（微软的）数据库_赠MATLAB版

&emsp;　数据库比我想象中繁琐很多，路过的大神请不吝赐教。这篇写Python连接微软的两款数据库（Access和SQL Server）并简单插入数据，用的两种方案为pyodbc执行SQL语句和SQLAlchemy结合`<pandas.DataFrame>`的`to_sql()`方法。附赠MATLAB版简单教程。

&emsp;　本文教程环境都是64位：Anaconda3.6，MATLAB2017a，Access2016，SQL Server2017。


&emsp;　依次介绍如下：


## pyodbc连接本地Access
&emsp;　ODBC是微软推出现在较广接受的数据库接口，可以在设置完数据源后很方便地从程序连接该数据源（后来才知道）。但是手动设置数据源这个动作我并不喜欢，（借口）在电脑上留下的额外的痕迹。所以设置数据源的方法后介绍。

&emsp;　首先需要确定电脑上装了AccessDatabaseEngine_X64，在`控制面板\所有控制面板项\管理工具\ODBC 数据源(64 位)`中查看以获得下面代码中DBDRIVER的字符串。

```python
"""
pyodbc连接本地Access示例
目标：将.txt文件中的数据写入本地test.accdb中的"目标表名"表
描述：pyodbc连接本地Access，结合SQL的插入语句执行插入
"""
import numpy
import pyodbc

# 数据库驱动程序名称
DBDRIVER = "{Microsoft Access Driver (*.mdb, *.accdb)}"
# 数据库路径
DBPATH = r"E:\test.accdb"
# 数据库连接字符串
CONNECTSTR = f"Driver={DBDRIVER};DBQ={DBPATH}"
TABLENAME = "目标表名"
# 连接到（新创建的）数据库
DBCONN = pyodbc.connect(CONNECTSTR)
# 创建数据库游标
DBCURSOR = DBCONN.cursor()
# 要插入的二维数组
DATA = numpy.loadtxt(r"E:\example.txt", dtype=numpy.int32)
insertSQL = f"INSERT INTO {TABLENAME} VALUES ({"?, " * len(DATA[0])}"
# SQL的插入语句
insertSQL = insertSQL[:-2] + ");"
# 转为二维列表
DATA = DATA.tolist()
for row in DATA:
    # 执行SQL语句
    DBCURSOR.execute(insertSQL, *row)
    # 确认对数据库的修改
    DBCURSOR.commit()  
DBCURSOR.close()  # 释放数据库游标
DBCONN.close()  # 释放数据库连接
```

&emsp;　可以看出pyodbc连接数据库后就是通过手动执行SQL语句进行对数据的操作，我这个示例是把要插入的数据一行一行拆开插入，每行`execute`后都`commit`，效率很低，当时对SQL不了解（其实现在也不咋了解），这么做也是没办法，仔细想想SQL本身其实也是有批量插入的语法来着，应该可以一次就构造完一句插入语句执行一次就可以回避这个循环。

&emsp;　记得要`commit`以及最后**释放**数据库游标和数据库连接。

&emsp;　设置数据源的方法则是在“`控制面板\所有控制面板项\管理工具\ODBC 数据源(64 位)`”中添加用户数据源指向你的test.accdb，给它起个合适的`ODBC data source name`即下面的`DataSourceName`然后将上一段代码中的`DBCONN`换成

```python
# DataSourceName为数据源名，user为用户名，password为密码
DBCONN = pyodbc.connect("DSN=DataSourceName; UID=user; PWD=password")
```

就行了，其他部分不变。

## pyodbc连接远程SQL Server

&emsp;　pyodbc连接远程SQL Server可以当作类似于pyodbc连接本地Access，区别也仅在于DBCONN：

```python
# IPaddress为远程SQL Server的IP地址，portnumber为远程SQL Server的端口号
# databasename为远程SQL Server中的目标数据库
DBCONN =pyodbc.connect("DRIVER={SQL Server}; SERVER=IPaddress,portnumber; DATABASE=databasename; UID=user; PWD=password")
```
&emsp;　设置数据源的方法也类似。

## MATLAB连接远程SQL Server

&emsp;　MATLAB连接数据库大多有ODBC和JDBC两种方式，我目前只用过ODBC方式（必须设置数据源），如下：

&emsp;　MATLAB的话连接之前必须先设置好ODBC data source name，需在“`控制面板\所有控制面板项\管理工具\ODBC 数据源(64 位)`”中添加用户数据源，给你的远程SQL Server取个合适的ODBC data source name。然后代码如下

```matlab
% database函数创建了ODBC连接，参数都是字符串，依次是数据源名称、用户名、密码
conn = database("ODBC data source name","username","pwd");
conn.Message  % 测试连接若返回空矩阵则连接正常
tablename = "databasename.dbo.tablename";  % 表名，最好类似绝对路径写全
colnames =  {"年", "月", "日", "吃"};  % 列名
% data可以是numeric matrix、cell array、table、dataset array或structure
data = {2018, 2, 5, "黄焖鸡"; 2018, 2, 6, "麻辣香锅"}
% 若data是cell array，插入顺序将对应colnames
fastinsert(conn, tablename, colnames, data);  % 执行插入动作
close(conn)  % 释放数据连接
```

&emsp;　我是在用过pyodbc连接Access和远程SQL Server之后再尝试MATLAB的fastinsert，感觉这个真的是非常快，尤其是给远程SQL Server插入大量数据时，带宽几乎能占满，简直是把整个大表一口气甩了过去。现在想想可能是用pyodbc时是循环一条一条插数据，而MATLAB的fastinsert是真的整个一起插。

## MATLAB连接本地Access

&emsp;　与MATLAB连接远程SQL Server相似，添加用户数据源时选择Access的驱动就行。与MATLAB连接远程SQL Server区别在于连接本地Access的`username`和`pwd`通常为空

```matlab
conn = database("ODBC data source name","","");
```
（我仿佛记得之前用MATLAB连接本地Access时好像没有用database函数，而是直接通过.accdb文件的路径打开数据库来着，但是那个脚本找不着了，网上也不知道哪儿去了，路过的大神若是知道劳烦赐教）

## SQLAlchemy连接本地SQL Server

&emsp;　在用Python连接Microsoft SQL Server 2017时身份验证方式为“SQL Server 身份验证”，所以先要在SQL Server中事先设置好登录名和密码（注意不是“Windows 身份验证”）

```python
"""
Python3连接数据库示例
目标：将.csv文件中的数据写入本地SQLServer中的databasename库中的tablename表
描述：SQLAlchemy连接本地SQLServer，结合pandas.DataFrame的.to_sql()方法直接插入
"""
import pandas as pd
from sqlalchemy import create_engine

DATAFRAME = pd.read_csv(r"E:\example.csv", sep="\s+", engine="python", encoding="utf-8")
# USERSTR为包含用户名、密码、本地SQL Server名、数据库名的字符串
USERSTR = "username:password@hostname/databasename"
# LINKSTR为create_engine要用到的字符串，除了USERSTR还包括了数据库类型的信息
LINKSTR = f"mssql+pyodbc://{USERSTR}?driver=SQL+Server"
# CON为sqlalchemy.engine.base.Engine对象，该对象创建时貌似还没有真正连接到数据库
CON = create_engine(LINKSTR, encoding="utf-8")
DATAFRAME.to_sql("tablename", CON, if_exists="append", index=False)
# 仅上一行操作：33205*11的数据存入过程花费21.03s
```
&emsp;　要注意DATAFRAME中列名和数据库中tablename的列名对应，顺序可乱但要对应。

&emsp;　`pandas.DataFrame`的`to_sql()`方法中的参数：

- if_exists有三个选项fail（默认）、replace或append，常用append添加，replace会删掉原表重新创建，fail只会在该表不存在时创建
- index有两个选项True（默认）或False，决定是否将DataFrame index当作一列写入
- chunksize接受int型正数表示每次写入的行数，默认为None即所有行一次性写入

&emsp;　SQLAlchemy通过设置数据源的方式连接数据库没试过，如果有的话应该也只是建立连接时传入的字符串不同，有机会再试试。

教训：

&emsp;　“`8月22日;22日;22日;30日`”无法存入`char(20)`，这是我把chunksize设为1时发现的，也算是查bug的一个方法吧。

## SQLAlchemy连接远程SQL Server

&emsp;　与SQLAlchemy连接本地SQL Server相似，区别在于USERSTR中hostname换成远程SQL Server的IP地址和端口号，格式为

```python
# USERSTR为包含用户名、密码、远程SQL Server的IP地址、端口号、数据库名的字符串
USERSTR = "username:password@IPaddress:portnumber/databasename"
```

# 总结

&emsp;　MATLAB连接远程SQL Server那次给我印象最深，看着MATLAB占满带宽“甩”数据不得不承认MATLAB确实很管用。不过仔细想想，我感觉SQLAlchemy结合`<pandas.DataFrame>`的`to_sql()`方法应该和MATLAB的fastinsert的插入速度差不多，只是到目前都再没有大量远程插入的机会试试。

&emsp;　目前让我说连接数据库的话，SQLAlchemy应该是最简单的，它好像封装了很多数据库的操作，虽然我暂时还没接触到。MATLAB其实差不多简单，因为某些原因不推荐。pyodbc复杂一些，但是逻辑清楚，从SQLAlchemy的连接字符串看，估计SQLAlchemy底层也调用了pyodbc，以及中规中矩的“释放连接”的操作莫名觉得很“专业”。

&emsp;　总的来说我觉得就连接数据库而言，难点仅在于那一个承载连接信息的字符串，找对这个字符串，一切都好说。连上之后基本上就靠的是数据库方面的知识了，那才是深坑。