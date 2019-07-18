# Python3.SQLAlchemy1.3提高<pandas.DataFrame>的SQLSever插入速度

&emsp;　`<pandas.DataFrame>`的`to_sql()`方法导入数据库很方便，但是比较慢（参考另一篇[<u>具体插入方法</u>](../Python3.连接（微软的）数据库_赠MATLAB版/Python3.连接（微软的）数据库_赠MATLAB版.md)）。在最近一次大量数据导入SQLSever时让我尤其不耐烦，尝试过给`to_sql()`方法传入`method="multi"​`不知道为啥竟然报错了。

&emsp;　百度`pandas to_sql 加速`的结果只有一篇利用`sqlalchemy`的`copy_from`方法加速PostgreSQL数据插入的教程，但是对于SQLSever并不适用。

&emsp;　但是终于还是被我找到了！

![SQLAlchemy1.3改进](https://img-blog.csdnimg.cn/20190422093814547.png#pic_center)

&emsp;　是的，只要在用`sqlalchemy`的`create_engine`函数时传入`fast_executemany=True`参数就可以大大提高SQLSever插入速度，原来要花5分钟现在只要5秒钟，爽歪歪。

&emsp;　但是要注意的是当设置`fast_executemany=True`时，同样不要指定`to_sql()`的`method`。根据官方文档，默认的`method`是一行一行地插入，而`method="multi"​`是一条SQL语句插入所有数据， 而结果和想象中有些不一样。所以优化方案交给底层，咱不要瞎操心了吧。

---

&emsp;　这个新参数实际上并不是`create_engine`自己的，而是将传给底层`pyodbc`库的，所以直接搜还真很难搜到，我是在stackoverflow上一个类似问题底下最后一个新回答发现的，感动得我立即注册账号给他点赞（遗憾的是新人好像点不了赞）。

![](../weixin_qr.png)