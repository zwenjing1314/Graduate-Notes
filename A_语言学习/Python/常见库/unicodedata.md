# 介绍

unicodedata 是 Python 的标准库，用于处理 Unicode 字符的各种属性。它提供了访问 Unicode 字符数据库的功能，包括：获取字符的名称、类别、数值等属性，字符规范化（normalization），双向属性信息等 

# 常用API

## normalize() 

这个函数用于将 Unicode 字符串转换为标准形式。Unicode 允许用不同的码点序列表示同一个字符，normalize 就是为了解决这个问题。
四种标准化形式：

```py
import unicodedata

# NFD: 规范分解 (Canonical Decomposition)
# NFC: 规范合成 (Canonical Composition)
# NFKD: 兼容分解 (Compatibility Decomposition)
# NFKC: 兼容合成 (Compatibility Composition)

text = "O'Néàl"
normalized_nfd = unicodedata.normalize('NFD', text)
print(f"原始：{text}")
print(f"NFD: {normalized_nfd}")

# NFD 会将带重音的字符分解为基本字符 + 组合标记
# 'é' 会被分解为 'e' + '́' (组合重音标记)
# 'à' 会被分解为 'a' + '̀' (组合重音标记)
```

## category()