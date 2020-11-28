# Notes about Programming

## C++

### struct 内存对齐

[C/C++内存对齐详解 - 知乎专栏](https://zhuanlan.zhihu.com/p/30007037)

struct 第一个成员 (用 `member[0]` 表示) 的地址偏移量（offset）为 0。

后面任意一个成员 `member[x]` 的 `offset[x]` 计算规则如下

```pseudocode
align = sizeof(member[x])
candidate_set = {k | k * align > offset of the last byte of member[x - 1]}
offset[x] = minimum(candidate_set)
```

如果人为设置了一个对齐系数 `pack(n)`，那么上面的规则只要稍微修改一下就好，字节大小小于等于 `n` 的成员规则不变，大于 `n` 的就按 `n` 对齐。

```pseudocode
align = minimum(sizeof(member[x]), n)
candidate_set = {k | k * align > offset of the last byte of member[x - 1]}
offset[x] = minimum(candidate_set)
```

以下是维基百科中计算 alignment 和 padding 的算法

```pseudocode
padding = (align - (offset mod align)) mod align
aligned = offset + padding
```

除了成员在内部需要对齐，struct 自己的起始地址也要对齐

```pseudocode
align = minimum(
            n,
            maximum(sizeof(member[0]), sizeof(member[1]), sizeof(member[2]), ......)
        )
```

## Python

### Logging

```python
import logging

logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)
file_handler = logging.StreamHandler()
formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(name)s - %(message)s')
file_handler.setFormatter(formatter)
logger.addHandler(file_handler)
```

### 多进程、多线程

因为 GIL 的关系，多线程其实是**伪**多线程，CPU 即使有多核也不能同时执行多个线程。多进程才能真正让 CPU 同时进行。

多进程可用原生 `Multiprocessing` 库里的 `Pool` 类及其 `apply_async(...)` 函数。

Windows 系统上基于 `Multiprocessing` 库的程序基本都没能正常运行，不知道原因，同样的代码在 Linux 系统可以正常工作。

### 按行读文本文件

常用的做法是

```Python
with open(file) as f:
    for line in f:
        do_something(line)
```

最近在做一些统计数据集信息的工作的时候，打算用 `pandas.Dataframe` 来先存放读入的数据。于是打算先读取文件的第一行来分析数据集包含哪些属性，即 `pandas.Dataframe.columns` ，然后再重新读每一行并使用 `pandas.Dataframe.append()` 存下来。于是很自然的就有了以下做法

```Python
with open(file) as f:
    for line in f:
        analysis_columns(line)
        break
    for line in f:
        xxx.append(json.loads(line))
```

结果发现，我的 `Dataframe` 缺少了第一行数据。原因其实是因为我进行第一次 `for line in f` 的操作的时候，让 `f` 的迭代器往前走了一步，于是第二次就直接从第二行开始了。以下两种方法都可以解决这个问题

```Python
with open(file) as f:
    data = f.readlines()
    for line in data:
        analysis_columns(json.loads(line))
        break
    for line in data:
        xxx.append(json.loads(line))
```

或者重新 `open` 一次文件

```Python
with open(file) as f:
    for line in f:
        analysis_columns(line)
        break

with open(file) as f:
    for line in f:
        xxx.append(json.loads(line))
```

### 大量数据存入`pandas.DataFrame`

接上一节。我发现 3mil 量级的数据花了 6 小时只完成了 20%，觉得很奇怪因为之前直接用 Python 改这个数据集的格式（不牵扯到 `pandas.DataFrame` ）的时候也没那么久。于是我直接先把 list of `dict`s 转换成一个完整的 `dict` 然后直接整个转换成 `pandas.DataFrame` ，发现也很快。然后去查 StackOverflow 和 `pandas.DataFrame` 自己的文档，发现是我自己用得太少了，`pandas.DataFrame.append` 效率就是很低，以及 `pandas.DataFrame` 的初始化函数的参数是可以接受 list of `dict`s 的。直接让 `pandas.DataFrame` 帮我处理而不是用 `append` 效率就提高了。

### Print UnicodeEncodeError

[python(三)：Python3—UnicodeEncodeError 'ascii' codec can't encode characters in position 0-1](https://blog.csdn.net/AckClinkz/article/details/78538462)

[Python3.6 sys.stdout.encoding的输出为'ANSI_X3.4-1968'](https://blog.csdn.net/j___t/article/details/97705231)

### Encoding

我在 Ubuntu 18.04 上运行 Python 程序的时候遇到过源文件的编码问题，会出现类似以下的情况

```shell
SyntaxError: Non-ASCII character '\xe6' in file my_python_program.py on line 5, but no encoding declared; see http://python.org/dev/peps/pep-0263/ for details
```

```shell
if not line.startswith('  File "<frozen importlib._bootstrap'))
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe5 in position 11: ordinal not in range(128)
```

这时需要在源代码的开头加上

```shell
# -*- coding: utf-8 -*-
import sys
reload(sys)
sys.setdefaultencoding("utf-8")
```

### 正则表达式

在参加大数据比赛的时候，我负责数据清洗的工作，但我在使用正则表达式去除特殊符号的时候，我发现数字全被去除了，而我本意没有要去除这些数字。原来是我在正则表达式中“不小心”使用了 character range 的表示方法。例子如下

```python
text = '+12-3ab+c_456d_ef'
text = re.sub(r'[+-_]+', '', text)
print(text)
```

我想去除加号、减号、下划线，但是会输出 `abcdef`。因为我写的 `[+-_]` 会被解析成类似 `[0-9]` 这种使用范围表示的字符。如果我把 `[+-_]` 替换成 `[a-_]`，运行就会报错 `bad character range a-_ at position 1`。
