# Python3.tkinter实践之文献复制工具ChinglishCopier

> tkinter非常适合用于快速搭建一个简单的GUI程序

tkinter是Python的标准库，只要有Python环境就可以用它搭建简单的桌面程序，并且结合Pyinstaller可以打包成.exe程序运行在没有Python环境的电脑上。

tkinter十分适合在用Python进行科学计算时，定制GUI程序用于人为干预计算、绘图展示等需求。常用库都会提供相应的支持如matplotlib直接支持在tkinter中嵌入式绘图等。

本文将介绍一个我用tkinter实现的文献复制工具ChinglishCopier（[GitHub链接](https://github.com/Mo-Dabao/ChinglishCopier)），并提供用Pyinstaller打包的可执行程序（[百度网盘链接](https://pan.baidu.com/s/1srRgb2zAqavrzp9wnCtjbw)）。

---


## 目的

为了解决从pdf等格式的中文文献中复制粘贴常会出现的一些问题：

- 数字、字母等非中文字符变成全角字符
- 产生多余的空格、换行
- 中、英文标点（全、半角）混乱
- 英文上下文中的标点后缺空格

![直接复制.gif](./直接复制.gif)


## 实现

- 删除多余的空白符（换行、空格、制表等）
- 英文、数字、部分标点符号的全角字符换成半角字符
- 修正误判为中文标点（，；：！？）的英文标点字符
- 英文标点后确保一个空格

![使用工具.gif](./使用工具.gif)


## 程序结构

程序主要结构有两个模块：<br>
├── [organize.py](https://github.com/Mo-Dabao/ChinglishCopier/blob/master/organize.py)<br>
└── [gui_tkinter.py](https://github.com/Mo-Dabao/ChinglishCopier/blob/master/gui_tkinter.py)

`organize.py`负责逻辑操作：遍历复制的字符串按照条件整理，这不是本文的重点。（逻辑仍然可以继续优化，感兴趣的读者可以一起改进）

`gui_tkinter.py`负责GUI的搭建，除去字符串和注释其实大概其实就50行代码左右：

```python
# coding: utf-8
"""
ChinglishCopier's GUI powered by tkinter
未实现：
- 监听剪贴板

@author: Mo Dabao
"""


import tkinter as tk
from organize import organize as organize_core
from ChinglishCopier_pic import weixin_qr


# 帮助文本
HELP = \
"""\n\n\n
清空：
    清空此文本框的内容。\n\n\n
整理：
    若文本框内有内容，则整理文本框内容；
    若文本框内无内容，则整理剪贴板内容。\n\n\n
剪切：
    将文本框内容剪切到剪贴板。

"""

# 关于文本
ABOUT = \
"""\n\n\n
此软件对查重结果概不负责！！！
\n\n\n
所有整理的文献内容归原作者所有
\n\n\n
==================================================\n\n
                                      作者：墨大宝\n
                                微信公众号：碎积云\n
                                         2019-2-22
"""


class MainWindow(tk.Tk):
    """
    主窗口
    """
    def __init__(self):
        super().__init__()
        # 禁止窗口缩放
        self.resizable(width=False, height=False)
        # 主窗口标题
        self.title("查重100%！")
        # 菜单栏
        self.menubar = tk.Menu(self)
        self.menubar.add_command(label="帮助", command=self.help)
        self.menubar.add_command(label="关于", command=self.about)
        self.config(menu=self.menubar)
        # 文本框
        self.text_disp = tk.Text(self, width=50, height=30)
        self.text_disp.insert(tk.INSERT, HELP)
        self.text_disp.grid(row=0, column=0, columnspan=3)
        # “清空”按钮
        self.button_clear = tk.Button(self, text="清空", command=self.clear)
        self.button_clear.grid(row=1, column=0, sticky=tk.W+tk.E)
        # “整理”按钮
        self.button_oganize = tk.Button(self, text="整理", command=self.organize)
        self.button_oganize.grid(row=1, column=1, sticky=tk.W+tk.E)
        # “剪切”按钮
        self.button_cut = tk.Button(self, text="剪切", command=self.cut)
        self.button_cut.grid(row=1, column=2, sticky=tk.W+tk.E)
        # 读取公众号二维码图片的base64编码
        self.weixin_qr = tk.PhotoImage(data=weixin_qr)
        # 事件循环
        self.mainloop()

    def clear(self):
        """
        清空text_disp文本框
        """
        self.text_disp.delete(0.0, tk.END)  # 清空text_disp文本框内容

    def organize(self):
        """
        获得text_disp文本框的内容并整理
        整理完覆盖掉text_disp文本框的内容
        """
        text_old = self.text_disp.get(0.0, tk.END)  # 获取text_disp文本框内容
        text_old = text_old if not text_old.isspace() else self.clipboard_get()
        self.clear()
        text_new = organize_core(text_old)
        self.text_disp.insert(tk.INSERT, text_new)

    def cut(self):
        """
        将文本框内的内容剪切到剪切板
        """
        text = self.text_disp.get(0.0, tk.END)
        self.clear()
        # 更新剪贴板内容为指定字符串
        self.clipboard_clear()  # 清除剪贴板内容
        self.clipboard_append(text)  # 向剪贴板追加内容

    def help(self):
        """
        菜单栏“帮助”
        """
        self.clear()
        self.text_disp.insert(tk.INSERT, HELP)

    def about(self):
        """
        菜单栏“关于”
        """
        self.clear()
        self.text_disp.insert(tk.INSERT, ABOUT)
        # 插入图片
        self.text_disp.image_create(tk.END , image=self.weixin_qr)


if __name__ == "__main__":
    root = MainWindow()

```

详细各种控件的用法以后有机会慢慢写。

---

这个程序从去年开始断断续续当Python练手的项目在做，基本实现了既定目标——但是我已经不需要写课程报告了，只能在摘录文献记笔记的时候用一用了。<font color=red>**千万不要用于论文中，复制一时爽，一直复制一直爽！**</font>

| 时间 | 备注 |
|--|--|
| 2018-10-04 | 创建 |
| 2018-10-26 | 优化空格、换行的删除逻辑；优化代码风格 |
| 2019-02-22 | 重构，优化中英文标点和空格的判断逻辑，<br>界面和逻辑分离；新增菜单栏，去掉广告窗口 |

