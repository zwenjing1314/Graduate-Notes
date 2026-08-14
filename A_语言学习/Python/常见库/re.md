# 介绍

re 是 Python 内置的标准库，用于处理正则表达式（Regular Expression），实现字符串的匹配、搜索、替换和分割等操作

# 常见API

## sub()

函数签名

```py
re.sub(pattern, repl, string)
```

- pattern: 正则表达式模式（用来匹配你想找的内容）。
- repl: 替换后的内容（如果为空字符串 ""，就相当于删除匹配到的内容）。
- string: 原始字符串。

示例

```py
def _compact_numeric_text(text: str) -> str:
    return re.sub(r"[^0-9.\-()%]", "", text or "")
```

从一段文本中剔除所有“非数字”相关的字符，只保留数字、小数点、负号、括号和百分号