# Python3.用json作为配置文件

最近有些自动化脚本要反复使用，但是每次使用都有一些参数要改：路径啊、时间间隔等等。每次打开源文件改这些参数时免不了看到大片的代码，令人头秃，而且将参数和程序写死在一起也不是好习惯。

最终我决定用json文件作为配置文件来保存这些可能要人工修改的参数，主要技巧在于将字典内容转为变量。

---

首先从我的另一次尝试开始讲：

## py文件作为配置文件

另外建一个`config.py`源文件保存到主程序`main.py`所在路径下，将需要的配置参数用正常赋值语句保存下来像这样：

```python
# config.py
target_dir = r"E:\data"
interval_mins = 5
time_record = "201904011230"
# ......
```

在主程序开头加上

```python
# main.py
from config import *
```

就实现了配置和程序的分离，非常方便。

但是有两个问题：

1. 若是程序要修改配置文件中从某些参数就比较麻烦。比如主程序要在每次启动时读取到配置文件中的`time_record`变量，并在结束时将其保存为新的值以便下次使用。
2. 若是将`main.py`打包成`.exe`可执行文件，`config.py`也会被一起打包进去，就无法再修改配置了。

关于问题1，可以通过将配置文件当做文本文件通过`"r+"`方式读取，修改最后一行`time_record`后的文本值，再全部写入进配置文件。这样算一下其实配置文件会被读取两次（一次被`import`，一次被当做文本），假如有多个配置变量要修改、在一次执行中要多次修改，那代码就比较难写了，而且执行效率也不高。

接下来就是我最终的解决方案：

## json文件作为配置文件

和上文相同的例子，`config.py`改写成`config.json`是这样：
```json
{
    "target_dir" = "E:/data",
    "interval_mins" = 5,
    "time_record" = "201904011230"
}
```
在`main.py`中增加两个函数分别用于读取和更新配置：
```python
# main.py
def read_config():
    """"读取配置"""
    with open("config.json") as json_file:
        config = json.load(json_file)
    return config


def update_config(config):
    """"更新配置"""
    with open("config.json", 'w') as json_file:
        json.dump(config, json_file, indent=4)
    return None
```
当通过`config = read_config()`获得的配置`config`是一个字典，不能直接使用如`target_dir`等键值当做变量使用，可以间接用如`config["target_dir"]`来当变量，但并不方便。通常做法是每个变量执行一次类似`target_dir = config["target_dir"]`的操作，如果配置变量较多就比较累了。

那么，重点来了：
```python
globals().update(config)  # 知识点啊
```
`globals()`获得（模块级）全局变量所组成的字典，修改该字典等同修改全局变量，所以通过`.update(config)`可以将`config`字典内容转为变量。举个例子：
```python
# example
a_dict = {"key": "value"}
globals().update(a_dict)
print(key)
```
虽然没有显式定义变量`key`，但依然可以正确输出:`value`

重点结束。

接着说更新配置文件的事：

修改字典`config`后，只要`update_config(config)`，就实现了配置文件`config.json`的更新，可以说是非常简单了。

---

都用过好久Python了，学的时候也见过`globals()`，但查到它的`.update()`方法将字典内容转为变量这一用法时还是觉得：哇！还能这么搞，有点儿意思！

![](https://img-blog.csdnimg.cn/20190314220833190.png#pic_center)
