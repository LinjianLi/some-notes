# Notes about Programming

## C++

### struct 内存对齐

[C/C++内存对齐详解 - 知乎专栏](https://zhuanlan.zhihu.com/p/30007037)

struct 第一个成员 (用 `member[0]` 表示) 的地址偏移量（offset）为 0。

后面任意一个成员 `member[x]` 的 `offset[x]` 计算规则如下

```pseudocode
align = sizeof(member[x])
candidate_set = {k * align | k is non-negative integer, k * align > offset of the last byte of member[x - 1]}
offset[x] = minimum(candidate_set)
```

如果人为设置了一个对齐系数 `pack(n)`，那么上面的规则只要稍微修改一下就好，字节大小小于等于 `n` 的成员规则不变，大于 `n` 的就按 `n` 对齐。

```pseudocode
align = minimum(sizeof(member[x]), n)
candidate_set = ...
offset[x] = ...
```

以下是[维基百科](https://en.wikipedia.org/wiki/Data_structure_alignment)中计算 alignment 和 padding 的算法

```pseudocode
padding = (align - (offset mod align)) mod align
aligned = offset + padding
```

其中，`align` 是需要对齐的大小，`offset` 是假如不对齐的话本该放置的下一个地址，`padding` 是如果需要让下一个地址对齐应该从 `offset` 开始垫上多少个字节，`aligned` 就是对齐后该使用的下一个地址。

举例分析：假设我们需要 4-byte 对齐。假如不对齐的时候下一个该放置的地址是 `0x59d`，即十进制的 `1437`。1437 不是 4 的整数倍，但 1437 + 3 = 1440 是 4 的整数倍。所以如果从 `1437` 处垫上 3 个字节，即把 `[1437, 1438, 1439]` 给占用，那么下一个可用地址就是 `1440`，即 `0x5a0`。

那么 "3" 是如何算出来的？对于原本的 `offset`，先计算它比正常的 `align` 对齐的上一个地址超出多少，公式应该是 `offset mod align`，对应到前面的例子可得 `1437 mod 4 = 1`。所以需要垫上的字节数 `padding` 就是 `align` 值减去前面超出的数值，这样通过 `offset + padding` 算出的新地址就是 `align` 的整数倍了。因此就有了前面公式中的 `align - (offset mod align)`。对应前面例子可得 `padding = 4 - 1 = 3`。到这里似乎就可以结束了。但是有一个特殊情况没有考虑到，假如 `offset` 本来就是对齐的，那么 `offset mod align = 0`，然后 `padding = align - 0 = align`，这样算出来意思是还要在垫上 `align` 个字节，而实际上是不需要的，因为已经对齐了。所以这种情况实际上 `padding = 0`。因此如果补上 `padding = padding mod align`，就可以应对这种情况，当原始 `padding` 算出来小于 `align` 的时候，说明确实没对齐，`padding mod align` 的结果仍是 `padding`，就正常垫上 `padding`；当原始 `padding` 算出来等于 `align` 的时候，说明已经对齐，`padding mod align` 的结果等于 `align mod align = 0`，所以实际 `padding = 0`。

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
import os
import time
import logging

# The letter "T" is a delimiter suggested in ISO-8601.
log_dir = os.path.join(os.getcwd(), "log")
os.makedirs(log_dir, exist_ok=True)
logging.basicConfig(
    handlers=[
        logging.FileHandler(
            filename="./log/log-{}.log".format(
                time.strftime("%Y-%m-%dT%H%M%S", time.gmtime())
            ), 
            encoding='utf-8'
        )
    ],
    format="%(asctime)s - %(levelname)s - %(name)s - %(message)s",
    datefmt="%Y-%m-%dT%H:%M:%S",
    level=logging.INFO
)

logger = logging.getLogger(__name__)
```

or make a custom logger

```python
import logging

logger = logging.getLogger("any_name_you_like")
logger.setLevel(logging.DEBUG)  # Or "logging.WARNING", or any level you like.
formatter = logging.Formatter("%(asctime)s - %(levelname)s - %(name)s - %(message)s",
                                datefmt="%Y-%m-%d %H:%M:%S")
file_handler = logging.FileHandler("any_file_you_like")
file_handler.setFormatter(formatter)
logger.addHandler(file_handler)
# You can even output the log to the console screen.
stream_handler = logging.StreamHandler()
stream_handler.setFormatter(formatter)
logger.addHandler(stream_handler)
```

### 多进程、多线程

因为 GIL 的关系，多线程其实是**伪**多线程，CPU 即使有多核也不能同时执行多个线程。多进程才能真正让 CPU 同时进行。

Python 的多线程不能够真的并行，所以在 CPU 密集型的程序中就没什么实际作用。
但是它在 I/O 密集型的程序中就能发挥作用。
当一个线程要长时间等待 I/O 数据传输是，如果只有一个线程就会一直闲置着等待数据传输完毕，如果有多个线程就可以切换到别的线程以充分利用 CPU 资源。

多进程可用原生 `Multiprocessing` 库里的 `Pool` 类及其 `apply_async(...)` 函数。

#### 常用例子

我最常用的情况就是对大量实验数据进行预处理。假如所有数据存在一个 `txt` 文件里，处理后的数据要存到另一个 `txt` 文件里。可以先把数据读入一个 `list` 中，再把数据分组，然后每组数据用一个进程处理，然后合并数据，最后写出数据到目标文件。

```Python
import multiprocessing

# Define the function to process the data.
def data_processing(input_lines: list) -> list:
    result_lines = []
    # Do something to process the input_lines.
    # Store the result into the result_lines.
    return result_lines

cpu_count = multiprocessing.cpu_count()
print("CPU count: {}".format(cpu_count))
with open("input.txt", mode="r") as f_in:
    lines = f_in.readlines()

# Compute the group size for each process.
group_size = len(lines) // cpu_count + 1
grouped_lines = [lines[i*group_size:(i+1)*group_size] for i in range(cpu_count)]

# Process the data use multiprocessing.
results = []
with multiprocessing.Pool(cpu_count) as p:
    for group in grouped_lines:
        result = p.apply_async(data_processing, [group])
        results.append(result)
    results_get = [result.get() for result in results]

# Assemble all the results.
all_results = []
for result in results_get:
    all_results.extend(result)
all_results = [s + "\n" for s in all_results]
with open("output.multiprocess.txt", mode="w") as f_out:
    f_out.writelines(all_results)
print("Finish.")
```

#### 在 Windows 系统上使用 Multiprocessing

之前的笔记里我写了一句

> Windows 系统上基于 `Multiprocessing` 库的程序基本都没能正常运行，不知道原因，同样的代码在 Linux 系统可以正常工作。

后来查到解决办法，就是要加上 `if __name__ == "__main__":` 语句，然后把处理逻辑部分放入这个 `if` 块里面。把处理逻辑放封装在一个 `main()` 函数里，然后再把 `main()` 函数的调用放到 `if __name__ == "__main__":` 块内。例子如下

```Python
import multiprocessing

# Define the function to process the data.
def data_processing(input_lines: list) -> list:
    result_lines = []
    # Do something to process the input_lines.
    # Store the result into the result_lines.
    return result_lines

def main():
    cpu_count = multiprocessing.cpu_count()
    with multiprocessing.Pool(cpu_count) as p:
        # Use `p.apply_async()` and the `data_processing()` function to process the data.
        pass
    print("Finish.")

if __name__ == "__main__":
    main()
```

有兴趣深入了解的话，可以看一下我找到的解决办法对应的网页。

- [stackoverflow 页面 1](https://stackoverflow.com/questions/20222534/python-multiprocessing-on-windows-if-name-main)
- [stackoverflow 页面 2](https://stackoverflow.com/questions/38236211/why-multiprocessing-process-behave-differently-on-windows-and-linux-for-global-o)

在此我也简单摘录一些内容：

> On Linux (and other Unix-like OSs), Python's multiprocessing module using fork() to create new child processes that efficiently inherit a copy of the parent process's memory state. That means the interpreter doesn't need to pickle the objects that are being passed as the Process's args since the child process will already have them available in their normal form.
>
> Windows doesn't have a fork() system call however, so the multiprocessing module needs to do a bit more work to make the child-spawning process work. The fork()-based implementation came first, and the non-forking Windows implementation came later.
>
> ......
>
> Since Windows has no fork, the multiprocessing module starts a new Python process and imports the calling module. If Process() gets called upon import, then this sets off an infinite succession of new processes (or until your machine runs out of resources).
>
> ......

(此外要注意的是，如果还是要把处理后的数据写入一个 `txt` 文件中，在 Windows 系统上最后输出的文件大小可能和在 Linux 系统上输出的不一样（前者比后者大一点），然后用 `diff` 命令去比较得到的两个输出文件也存在大量不同点，但是肉眼看不出来。遇到这种情况，大概率是因为在 Windows 系统输出的换行符是 `CR LF` 即 `\r\n`，而 Linux 系统输出的换行符是 `LF` 即 `\n`，所以前者的每一行都多一个 `\r` 字符。)

##### 在 Windows 系统上的 Jupyter Notebook 环境里使用 Multiprocessing

想在 Windows 系统上的 Jupyter Notebook 环境里使用 Multiprocessing，如果像前面把实际处理数据的函数定义在同一个代码文件里面的话，运行之后会发现函数永远不结束，系统监控也会发现 CPU 根本没有负载，意味着其实没有正常运行。看一下启动 Jupyter Notebook 时的 shell 的输出能看到 `AttributeError: Can't get attribute 'data_processing' on <module '__main__' (built-in)>`。

解决办法：必须把实际处理数据的函数定义在一个单独的文件里，然后在 Jupyter Notebook 里面 `import` 对应的函数，然后再运行。详细内容参考[此 stackoverflow 页面](https://stackoverflow.com/questions/47313732/jupyter-notebook-never-finishes-processing-using-multiprocessing-python-3)。

在此简单摘录一些内容：

> It seems that the problem in Jupyter notebook as in different ide is the design feature. Therefore, we have to write the function (prime_factor) into a different file and import the module.
>
> ......
>
> It works using Pool but doesn't work using Process. What could be the reason?
>
> ......
>
> Mayby it is obviously, but for the next readers: If pool initializing function like prime_factor() in the question calls another functions they also must be putted in the same package together with prime_factor()
>
> ......

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
