# 介绍

**JSON**（JavaScript Object Notation）是一种轻量级的数据交换格式，它基于文本，易于阅读和编写，同时也易于机器解析和生成。

# 常见API

## dumps()

处理中文编码

```py
import json
# 创建一个包含中文的字典
data = {'name': '张三', 'age': 30, 'city': '北京'}
# 不使用 ensure_ascii 参数，默认转换为ASCII编码
json_str_default = json.dumps(data)
print(json_str_default) # 输出: {"name": "\u5f20\u4e09", "age": 30, "city": "\u5317\u4eac"}
# 使用 ensure_ascii=False 参数，保留中文字符
json_str_chinese = json.dumps(data, ensure_ascii=False)
print(json_str_chinese) # 输出: {"name": "张三", "age": 30, "city": "北京"}
```

默认情况下，*json.dumps* 会将非ASCII字符转换成ASCII编码的转义序列。为了在JSON输出中保留中文字符，可以使用 *ensure_ascii=False* 参数。

```py
json.dumps(manifest, sort_keys=True, ensure_ascii=False, indent=2)
```

ensure_ascii = False, 允许非 ASCII 字符（如中文）直接输出，而不是转义为 \uXXXX

sort_keys=True：按键名排序

indent = 2, 格式化输出，缩进 2 空格，便于人工阅读

## dump(), dumps(), load(), loads()

```py
import json

data = {"name": "Gemini", "age": 1}

# 将数据写入文件
with open("data.json", "w") as f:
    json.dump(data, f)

# 将数据转换为字符串
json_string = json.dumps(data)  # 将Python对象序列化为JSON格式的字符串并返回。
print(json_string)
print(type(json_string))

# 从文件读取数据
with open("data.json", "r") as f:
    data = json.load(f)
print(data)

# 从字符串读取数据
json_string = '{"name": "Gemini", "age": 1}'
data = json.loads(json_string)  # 将JSON格式的字符串反序列化为Python对象。
print(data)
print(type(data))

"""
{"name": "Gemini", "age": 1}
<class 'str'>
{'name': 'Gemini', 'age': 1}
{'name': 'Gemini', 'age': 1}
<class 'dict'>
"""
```

