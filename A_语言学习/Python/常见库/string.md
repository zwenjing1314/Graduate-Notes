# 介绍

一个辅助模块，提供常用的字符串常量和工具，其内容本质上还是 `str` 对象。

# 常用API

## ascii_letters()

string.ascii_letters 是 Python string 模块中的一个常量，它包含所有 ASCII 字母字符：

```py
import string

print(string.ascii_letters)   # 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'
print(string.ascii_lowercase) # 'abcdefghijklmnopqrstuvwxyz'
print(string.ascii_uppercase) # 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'
```

```
all_letters = string.ascii_letters + " .,;'-"
# 这行代码创建了一个包含所有英文字母和一些标点符号的字符集，用于处理名字中的字符。
```

