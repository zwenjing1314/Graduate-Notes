# 介绍

**glob模块**是Python标准库中一个重要的模块，主要用来**查找符合特定规则的目录和文件**，并将搜索的到的**结果返回到一个列表中**

# 常见API

## glob()

glob 模块用于使用通配符搜索匹配的文件路径。
基本语法：

```py
import glob

# 常见通配符：
# *  匹配任意数量的字符（不包括路径分隔符）
# ?  匹配单个字符
# ** 递归匹配所有子目录
# [] 匹配字符范围，如 [0-9] 或 [abc]
```

使用示例

```py
# 读取 data/names 目录下所有的 .txt 文件
files = findFiles('data/names/*.txt')
print(files)
# 输出：
# [
#   'data/names/Arabic.txt',
#   'data/names/Chinese.txt',
#   'data/names/English.txt',
#   'data/names/French.txt',
#   ...
# ]

# 然后可以遍历这些文件读取内容
for filename in files:
    lines = readLines(filename)
    print(f"{filename}: {len(lines)} 行")
```

完整示例

```
import glob

# 查找所有名字文件
all_files = glob.glob('data/names/*.txt')

# 统计每个国家的名字数量
for filepath in sorted(all_files):
    # 从路径提取国家名
    country = os.path.splitext(os.path.basename(filepath))[0]
    
    # 读取文件内容
    with open(filepath, 'r', encoding='utf-8') as f:
        names = [line.strip() for line in f.readlines()]
    
    print(f"{country}: {len(names)} 个名字")

# 输出示例：
# Arabic: 500 个名字
# Chinese: 500 个名字
# English: 500 个名字
# French: 500 个名字
# ...
```

