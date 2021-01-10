---
title: Python多进程库遇到信号signal死锁问题
categories: python
toc: true
---
最近写了一个多进程调用另一个python函数的脚本，
由于这个python函数我希望设置Timeout，使用了timeout_decorator库，
该库使用signal实现。
而python的multiprocessing库在调用包含signal的进程时会出现死锁问题。
我的代码见[gist](https://gist.github.com/chansonyhu/3fb61840d1100222473506de90aba9ce)。

在网上遇到类似的问题：
- [https://stackoverflow.com/questions/39230902/multiprocessing-join-deadlock](https://stackoverflow.com/questions/39230902/multiprocessing-join-deadlock)
- [https://bugs.python.org/issue29759](https://bugs.python.org/issue29759)

两处修改：
- [Release lock to prevent deadlocks on terminate](https://github.com/michael-a-cliqz/cpython/commit/1536c8c8cfc5a87ad4ab84d1248cb50fefe166ae)
- [More fixes preventing deadlock](https://github.com/michael-a-cliqz/cpython/commit/3a767ee7b33a194c193e39e0f614796130568630)

修改pool.py仍未解决问题，最后用os.system把cfa_diff当做系统脚本调用，使用bash的timeout来实现超时控制。注意超时返回值为整数**31744**，函数原来返回值**10**在bash中为**2560**。

Typo:
- Python的with语句默认加锁，在打开文件时建议使用保证进程安全
  ```python
  with open(file, 'w') as writer:
    # some code
  ```