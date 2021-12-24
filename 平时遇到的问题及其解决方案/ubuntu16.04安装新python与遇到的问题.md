[toc]

# 背景描述

今天在做南京大学的[计算机系统基础实验](https://nju-projectn.github.io/ics-pa-gitbook/ics2020/) ，在make fceux-am的时候出现了python语法错误的提示，首先想到应该是python3的版本出问题了遂找方法替换16.04里预装的3.5

# 安装新版本python3并替换为默认选项

## 使用deadsnakes PPA源

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt install python3.8
```

## 配置默认python3

```bash
$ which python3.8
/usr/bin/python3.8

$ sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.8 1

$ which python3.5
/usr/bin/python3.5

$ sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.5 2
```

通过以上命令，把python的各个版本添加到update-alternatives

然后配置python3指向python3.8

```bash
$ sudo update-alternatives --config python3


There are 2 choices for the alternative python3 (providing /usr/bin/python3).

  Selection    Path                Priority   Status
------------------------------------------------------------
* 0            /usr/bin/python3.5   2         auto mode
  1            /usr/bin/python3.5   2         manual mode
  2            /usr/bin/python3.8   1         manual mode

Press <enter> to keep the current choice[*], or type selection number: 2
```

# 问题与修复

用了一段时间很顺利，然后发现terminal打不开了，但是xTerm依旧可以使用。

## 定位问题

```bash
$ gnome-terminal

Traceback (most recent call last):
  File "/usr/bin/gnome-terminal", line 9, in <module>
    from gi.repository import GLib, Gio
  File "/usr/lib/python3/dist-packages/gi/__init__.py", line 42, in <module>
    from . import _gi
ImportError: cannot import name '_gi'
```

所以问题出在`_gi`上

进入目录`/usr/lib/python3/dist-packages/gi/`

```bash
$ ls | grep _gi

_gi_cairo.cpython-35m-x86_64-linux-gnu.so  
_gi.cpython-35m-x86_64-linux-gnu.so
```

解决方法是复制一份38版本的出来

```bash
sudo cp _gi.cpython-35m-x86_64-linux-gnu.so _gi.cpython-38m-x86_64-linux-gnu.so
$ sudo cp _gi_cairo.cpython-35m-x86_64-linux-gnu.so _gi_cairo.cpython-38m-x86_64-linux-gnu.so
```

还可以见：[Cannot import name '_gi'](https://stackoverflow.com/questions/59389831/cannot-import-name-gi)



最后还是选择开一个20.04的虚拟机去做lab了，就怕修了这个坑再出新的坑。