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
