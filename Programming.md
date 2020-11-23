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