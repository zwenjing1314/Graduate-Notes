# 介绍

主要用于进行哈希（hash）操作。

# 常用API

## sha512

**SHA512** 是一种安全的哈希算法，可将任意长度的数据映射为 **64 字节（512 位）** 的固定长度摘要，常用于密码存储、数据完整性验证等。 它由 *hashlib* 模块提供，支持直接计算或分步更新数据。

**示例：计算字符串的 SHA512 哈希值**

```py
import hashlib
data = "Hello, World!"
sha512_hash = hashlib.sha512(data.encode()).hexdigest()
print("SHA512 哈希值：", sha512_hash)
```

输出为 128 位十六进制字符串，且不同输入几乎不可能产生相同结果。

**基本用法** 使用 *hashlib.sha512()* 创建哈希对象，然后调用 *.update()* 添加数据，最后用 *.hexdigest()* 获取十六进制摘要：

```py
import hashlib
m = hashlib.sha512()
m.update(b"Part1")
m.update(b"Part2")
print(m.hexdigest())
```

这种方式适合处理大文件或分块数据。

**重要特性：增量哈希**

```py
import hashlib

# 方法 A：一次性哈希
data = b"Hello" + b" " + b"World"
hash_a = hashlib.sha256(data).hexdigest()

# 方法 B：分块哈希（结果完全相同！）
digest = hashlib.sha256()
digest.update(b"Hello")
digest.update(b" ")
digest.update(b"World")
hash_b = digest.hexdigest()

print(hash_a == hash_b)  # True ← 两种方法结果一致
```

