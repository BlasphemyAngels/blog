---
title: pyinotify模块--python监视文件系统的利器
date: 2017-06-02 20:44:03
categories:
  - python
tags:
  - python
  - pyinotify
---


## 正文

&ensp;&ensp;&ensp;pyinotify模块用来对Linux中的文件系统进行监视，能够实时捕捉到文件系统的变化---增删改查等。pyinotify不仅能对文件监视还能对目录监视，而且当监视对象处于可移动介质时，那么umount此介质上的文件系统后，被监视目标对用的watch对象将被自动删除，并且会产生一个umount事件。

<!--more-->

### 查看内核是否支持inotify机制
   `grep INOTIFY_USER /boot/config-$(uname -r)`
   输出：`CONFIG_INOTIFY_USER=y` 表示支持inotify机制

### <span id=#jumpwatch>Inotify事件</span>

* IN_ACCESS，即文件被访问
* IN_MODIFY，文件被 write
* IN_ATTRIB，文件属性被修改，如 chmod、chown、touch 等
* IN_CLOSE_WRITE，可写文件被 close
* IN_CLOSE_NOWRITE，不可写文件被 close
* IN_OPEN，文件被 open
* IN_MOVED_FROM，文件被移走,如 mv
* IN_MOVED_TO，文件被移来，如 mv、cp
* IN_CREATE，创建新文件
* IN_DELETE，文件被删除，如 rm
* IN_DELETE_SELF，自删除，即一个可执行文件在执行时删除自己
* IN_MOVE_SELF，自移动，即一个可执行文件在执行时移动自己
* IN_UNMOUNT，宿主文件系统被 umount
* IN_CLOSE，文件被关闭，等同于(IN_CLOSE_WRITE | IN_CLOSE_NOWRITE)
* IN_MOVE，文件被移动，等同于(IN_MOVED_FROM | IN_MOVED_TO)

### API

[API](http://seb.dbzteam.org/pyinotify/)

### WatchManager类

&ensp;&ensp;&ensp;pyinotify中使用watch对象进行对文件系统的监视，watch对象中有对监视对象的操作。那么管理这些watch对象的对象就是WatchManager对象。它封装了对 watch的管理操作，我们只需要添加包含特定操作的watch对象到WatchManager即可，剩余的对watch的操作WatchManager会自行完成。

常用方法：
* add_watch(self, path, mask, proc_fun=None, rec=False, auto_add=False, do_glob=False, quiet=True, exclude_filter=None)
  * path 要监视的文件系统的路径
  * mask 要监听的文件系统的[事件](#jumpwatch)
  * rec 如果为True, 则递归的将此watch添加到文件系统下的每个子目录上，默认为 False。
  * auto_add 如果为True，则在监视的文件系统中创建新文件或目录时，会自动将watch 添加到新文件或目录上。
  * return: dict of {str: int} 返回值是一个字典，字典的key代表路径，value代表路径对应的文件系统的watch对象的wd值（可以理解为用来标识watch的值）。

代码：
```python
wm = WatchManager()
wm.add_watch(path, mask, auto_add=True, rec=True)
```

### ProcessEvent类

&ensp;&ensp;&ensp;ProcessEvent对象用来处理文件系统的事件。一般使用时会集成它，然后重写它内部的相应事件方法。

事件方法：

* 方法名用`process_`加上上面提到的事件即可。如`process_IN_CLOSE`、 `process_IN_CREATE`等。

代码：
```python
class EventHandler(ProcessEvent):
    def process_IN_CREATE(self, event):
        print("Create file:%s" % os.path.join(event.path, event.name))

    def process_IN_DELETE(self, event):
        print("Delete file:%s" % os.path.join(event.path, event.name))

    def process_IN_MODIFY(self, event):
        print("Modify file:%s" % os.path.join(event.path, event.name))
```

### Notifier类

&ensp;&ensp;&ensp;Notifier类接受事件并处理它。它会维持一个事件队列，每次从事件队列中取出事件处理，新事件也加入队列。那么事件从哪来？看了上面可以知道，WatchManager用来管理所有的watch及watch观察到的事件。怎么处理事件？由上一节可以知道用户定义的ProcessEvent子类用来处理事件。所以先要为Notifier指定WatchManager和ProcessEvent对象。

`notifier = Notifier(wm, EventHandler())`

接下来就需要处理事件了，这里使用的是同步方式（异步方式可以自行查阅 [API](http://seb.dbzteam.org/pyinotify/))。

这里依赖两个函数：

* process_events(self) 函数从事件队列中取事件，然后交给相应的事件处理对象 (ProcessEvent类或其子类的对象)处理。
* check_events(self, timeout=None) 检查文件系统是否产生新事件, 直到超时timeout毫秒就阻塞它。如果有，返回True。
* read_events(self) 读取事件并为其创建相应数据结构，然后加入事件队列。
* stop(self) 关闭notifier，销毁所有的watch对象，事件等。

```python
    while(True):
        try:
            notifier.process_events()
            if(notifier.check_events()):
                notifier.read_events()
        except keyboardInterrupt:
            notifier.stop()
            break
```

完整代码：
```python

import os
from pyinotify import WatchManager, Notifier, ProcessEvent, IN_CREATE, IN_DELETE, IN_MODIFY

class EventHandler(ProcessEvent):
    def process_IN_CREATE(self, event):
        print("Create file:%s" % os.path.join(event.path, event.name))

    def process_IN_DELETE(self, event):
        print("Delete file:%s" % os.path.join(event.path, event.name))

    def process_IN_MODIFY(self, event):
        print("Modify file:%s" % os.path.join(event.path, event.name))

def FSMonitor(path="."):
    wm = WatchManager()
    mask = IN_DELETE | IN_CREATE | IN_MODIFY
    notifier = Notifier(wm, EventHandler())
    wm.add_watch(path, mask, auto_add=True, rec=True)
    print("now starting monitor %s" % path)
    while(True):
        try:
            notifier.process_events()
            if(notifier.check_events()):
                notifier.read_events()
        except keyboardInterrupt:
            notifier.stop()
            break

if __name__ == "__main__":
    FSMonitor("/home/hadis/Documents/doc/doc/pro/python")
    ```

实验结果：

在目录下创建一个test文件夹后：

![实验结果
](https://github.com/BlasphemyAngels/MarkDownPhotos/blob/master/pyin.png?raw=true)
&ensp;&ensp;&ensp;好了,pyinotify的简单应用就介绍到这里了，它的更深入的应用就靠各位根据[API](http://seb.dbzteam.org/pyinotify/)慢慢探索了。

## 参考链接

* [API](http://seb.dbzteam.org/pyinotify/)
* [python pyinotify模块详解](http://www.cnblogs.com/darkpig/p/5925507.html)
* [Linux文件实时同步--inotify + rsync + pyinotify](http://weipengfei.blog.51cto.com/1511707/1195019)

转载请注明:[Artemis的博客]([https://BlasphemyAngels.github.io)--> [点此看原文
