### 1. import 搜索路径

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210902135149.png)

### 路径搜索

- 从上面列出的目录里依次查找要导入的模块文件
- '' 表示当前路径
- 列表中的路径的先后顺序代表了python解释器在搜索模块时的先后顺序

### 程序执行时添加新的模块路径

```python
sys.path.append('/home/itcast/xxx')
sys.path.insert(0, '/home/itcast/xxx')  # 可以确保先搜索这个路径
In [37]: sys.path.insert(0,"/home/python/xxxx")
In [38]: sys.path
Out[38]: 
['/home/python/xxxx',
 '',
 '/usr/bin',
 '/usr/lib/python35.zip',
 '/usr/lib/python3.5',
 '/usr/lib/python3.5/plat-x86_64-linux-gnu',
 '/usr/lib/python3.5/lib-dynload',
 '/usr/local/lib/python3.5/dist-packages',
 '/usr/lib/python3/dist-packages',
 '/usr/lib/python3/dist-packages/IPython/extensions',
 '/home/python/.ipython']
```

### 2. 重新导入模块

模块被导入后，`import module`不能重新导入模块，重新导入需用`reload`

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210902135104.png)

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210902135115.png)

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210902135123.png)

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210902135135.png)

### 3. 多模块开发时的注意点

recv_msg.py模块

```python
from common import RECV_DATA_LIST
# from common import HANDLE_FLAG
import common


def recv_msg():
    """模拟接收到数据，然后添加到common模块中的列表中"""
    print("--->recv_msg")
    for i in range(5):
        RECV_DATA_LIST.append(i)


def test_recv_data():
    """测试接收到的数据"""
    print("--->test_recv_data")
    print(RECV_DATA_LIST)


def recv_msg_next():
    """已经处理完成后，再接收另外的其他数据"""
    print("--->recv_msg_next")
    # if HANDLE_FLAG:
    if common.HANDLE_FLAG:
        print("------发现之前的数据已经处理完成，这里进行接收其他的数据(模拟过程...)----")
    else:
        print("------发现之前的数据未处理完，等待中....------")
       
```

handle_msg.py模块

```python
from common import RECV_DATA_LIST
# from common import HANDLE_FLAG
import common

def handle_data():
    """模拟处理recv_msg模块接收的数据"""
    print("--->handle_data")
    for i in RECV_DATA_LIST:
        print(i)

    # 既然处理完成了，那么将变量HANDLE_FLAG设置为True，意味着处理完成
    # global HANDLE_FLAG
    # HANDLE_FLAG = True
    common.HANDLE_FLAG = True

def test_handle_data():
    """测试处理是否完成，变量是否设置为True"""
    print("--->test_handle_data")
    # if HANDLE_FLAG:
    if common.HANDLE_FLAG:
        print("=====已经处理完成====")
    else:
        print("=====未处理完成====")
```

main 模块

```python
from recv_msg import *
from handle_msg import *


def main():
    # 1. 接收数据
    recv_msg()
    # 2. 测试是否接收完毕
    test_recv_data()
    # 3. 判断如果处理完成，则接收其它数据
    recv_msg_next()
    # 4. 处理数据
    handle_data()
    # 5. 测试是否处理完毕
    test_handle_data()
    # 6. 判断如果处理完成，则接收其它数据
    recv_msg_next()


if __name__ == "__main__":
    main()
```



![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210902135615.png)

![img](https://raw.githubusercontent.com/codecodeabc/Note-len/main/img/20210902135623.png)