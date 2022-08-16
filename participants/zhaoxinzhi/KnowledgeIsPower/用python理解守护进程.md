## 1、写在前面

关于多进程中，守护进程的作用。（用python来体现）

> https://blog.csdn.net/weixin_39876645/article/details/111021511

## 2、多进程

### 1、现象

```python
import sys
import multiprocessing
import os
import time
print_text_pattern = "Output from process {0:s} - pid: {1:d}, ppid: {2:d}"
def child(name):
    while True:
        print(print_text_pattern.format(name, os.getpid(), os.getppid()))
        time.sleep(1)
def main():
    procs = list()
    for x in range(1, 3):
        proc_name = "Child{0:d}".format(x)
        proc = multiprocessing.Process(target=child, args=(proc_name,))
        proc.daemon = x % 2 == 0 # True
        print("Process {0:s} daemon: {1:}".format(proc_name, proc.daemon))
        procs.append(proc)
    for proc in procs:
        proc.start()
        counter = 0
        while counter < 3:
            print(print_text_pattern.format("Main", os.getpid(), os.getppid()))
            time.sleep(1)
            counter += 1
if __name__ == "__main__":
    print("Python {0:s} {1:d}bit on {2:s}\n".format(" ".join(item.strip() for item in sys.version.split("\n")), 64 if sys.maxsize > 0x100000000 else 32, sys.platform))
    main()
    print("\nDone.")
```

输出如下：

```bash
Python 3.7.0 (v3.7.0:1bf9cc5093, Jun 27 2018, 04:59:51) [MSC v.1914 64 bit (AMD64)] 64bit on win32

Process Child1 daemon: False
Process Child2 daemon: True
Output from process Main - pid: 20232, ppid: 7536
Output from process Child1 - pid: 1804, ppid: 20232
Output from process Main - pid: 20232, ppid: 7536
Output from process Child1 - pid: 1804, ppid: 20232
Output from process Main - pid: 20232, ppid: 7536
Output from process Child1 - pid: 1804, ppid: 20232
Output from process Main - pid: 20232, ppid: 7536
Output from process Child2 - pid: 20652, ppid: 20232
Output from process Child1 - pid: 1804, ppid: 20232
Output from process Main - pid: 20232, ppid: 7536
Output from process Child1 - pid: 1804, ppid: 20232
Output from process Child2 - pid: 20652, ppid: 20232
Output from process Main - pid: 20232, ppid: 7536
Output from process Child2 - pid: 20652, ppid: 20232
Output from process Child1 - pid: 1804, ppid: 20232

Done.
Output from process Child1 - pid: 1804, ppid: 20232
Output from process Child1 - pid: 1804, ppid: 20232
Output from process Child1 - pid: 1804, ppid: 20232
Output from process Child1 - pid: 1804, ppid: 20232
Output from process Child1 - pid: 1804, ppid: 20232
Output from process Child1 - pid: 1804, ppid: 20232
Output from process Child1 - pid: 1804, ppid: 20232
Output from process Child1 - pid: 1804, ppid: 20232
Output from process Child1 - pid: 1804, ppid: 20232
Output from process Child1 - pid: 1804, ppid: 20232
Output from process Child1 - pid: 1804, ppid: 20232
```

如果换成`proc.daemon = True`

则输出为：

```bash
Python 3.7.0 (v3.7.0:1bf9cc5093, Jun 27 2018, 04:59:51) [MSC v.1914 64 bit (AMD64)] 64bit on win32

Process Child1 daemon: True
Process Child2 daemon: True
Output from process Main - pid: 17364, ppid: 7536
Output from process Child1 - pid: 19244, ppid: 17364
Output from process Main - pid: 17364, ppid: 7536
Output from process Child1 - pid: 19244, ppid: 17364
Output from process Main - pid: 17364, ppid: 7536
Output from process Child1 - pid: 19244, ppid: 17364
Output from process Main - pid: 17364, ppid: 7536
Output from process Child1 - pid: 19244, ppid: 17364
Output from process Child2 - pid: 22428, ppid: 17364
Output from process Main - pid: 17364, ppid: 7536
Output from process Child1 - pid: 19244, ppid: 17364
Output from process Child2 - pid: 22428, ppid: 17364
Output from process Main - pid: 17364, ppid: 7536
Output from process Child2 - pid: 22428, ppid: 17364
Output from process Child1 - pid: 19244, ppid: 17364

Done.
```

### 2、解释

daemon=True:

python中主进程退出时，守护进程一同退出

daemon=False:

python中主进程退出时，子进程继续运行。

