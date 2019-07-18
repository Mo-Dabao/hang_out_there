# Python3.读取txt数据

```python
import numpy
import pyodbc

# 设置并进入目标目录
target_path = r'E:\硕士资料\智慧气象\原始数据（下载格式）\中国地面气候资料日值数据集(V3.0)'
os.chdir(target_path)
filelist = [x for x in os.listdir() if x.startswith('SURF_CLI_CHN_MUL_DAY-')]
factordict = {'本站气压':['PRS',13],
              '气温':['TEM',13],
              '相对湿度':['RHU',11],
              '降水':['PRE',13],
              '蒸发':['EVP',11],
              '风向风速':['WIN',17],
              '日照时数':['SSD',9],
              '0cm地温':['GST',13]}
for x in factordict:
    templist = [y for y in filelist if y.startswith('SURF_CLI_CHN_MUL_DAY-' + factordict[x][0])]
    temptable = numpy.array([list(range(factordict[x][1]))], dtype=numpy.int32)
    for y in templist[:2]:
        temptable = numpy.concatenate((temptable, numpy.loadtxt(y, dtype=numpy.int32)), axis=0)
    print(x)
    temptable = temptable.tolist()
    temptable.pop(0)
    factordict[x] = temptable
```
