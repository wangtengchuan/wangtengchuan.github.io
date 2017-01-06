---
layout: post
title: "Process and Threads"
description: ""
category: 
tags: []
---
{% include JB/setup %}
#进程与线程
本文介绍进程与线程中的各种基本原理与用法，文中但凡是涉及具体实现或者代码示例的均以Python为例。
##进程间通讯
那么多个进程之间怎么通讯呢？我们都知道进程通讯有很多种方式#TODO
###信号(signal)
[信号](http://blog.csdn.net/jhonguy/article/details/7716257)

###join与Deamon
Python多线程编程中，经常会用到join与setDeamon方法。那么它们使用场景是什么呢？有什么作用呢？

####join

假设我们在主线程A中创建了子线程B，其实两个线程的执行先后顺序是没法保证的。但是如果我们想保证它们按照先后顺序执行怎么办？这时候只需要B.join([timeout])就可以了。其中timeout参数时可选的，代表线程运行的最大时间，即如果超过这个时间，不管这个此线程有没有执行完毕都会被回收，然后主线程或函数都会接着执行的。

```
import threading
import time
class MyThread(threading.Thread):
        def __init__(self,id):
                threading.Thread.__init__(self)
                self.id = id
        def run(self):
                x = 0
                time.sleep(10)
                print self.id

if __name__ == "__main__":
        t1=MyThread(999)
        t1.start()
        for i in range(5):
                print i
                
```

运行结果：

```
0
1
2
3
4
999
```
子线程t1->start之后，主线程并没有等它执行完毕。如果我们把join()加入进去，其他不变。

```
import threading
import time
class MyThread(threading.Thread):
        def __init__(self,id):
                threading.Thread.__init__(self)
                self.id = id
        def run(self):
                x = 0
                time.sleep(10)
                print self.id

if __name__ == "__main__":
        t1=MyThread(999)
        t1.start()
        t1.join()
        for i in range(5):
                print i
                
```

我们再来看结果：

```
999
0
1
2
3
4
```
线程t1->start后，主线程停在了join()方法处，等sleep（10）后，线程t1操作结束被join，接着，主线程继续循环打印。

####setDaemon
程序运行中，执行一个主线程，如果主线程创建了子线程，它们就兵分两路分别运行。如果主线程完成想退出，就会检查子线程是否完成。如果子线程未完成，主线程就会等待子线程完成后再退出。但是有时候我们仅仅只想，如果主线程完成了，不管子线程是否完成，都要和主线程一起退出。例如这些子线程只是做一些监控之类的辅助功能，主线程结束的时候子线程也没必要运行了。

可以通过setDaemon()将子线程设置为守护进程来达到这样的目的。

在Python程序中，当没有**活着的非守护线程**的时候，整个程序退出。
注意：必须在start() 方法调用之前设置，如果不设置为守护线程，程序会被无限挂起。

```
import threading
import time
class MyThread(threading.Thread):
        def __init__(self,id):
                threading.Thread.__init__(self)
        def run(self):
                time.sleep(5)
                print "This is " + self.getName()
 
if __name__ == "__main__":
        t1=MyThread(999)
        t1.setDaemon(True)#也可以t1.daemon = True
        t1.start()
        print "I am the father thread."
```
运行结果：

```
I am the father thread.
```

