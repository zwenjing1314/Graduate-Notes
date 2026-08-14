# 介绍

是一种以数据为中心、高度可读的数据序列化语言。它旨在替代 XML 和 JSON，成为配置文件和数据交换的首选格式，尤其在云原生生态（如 Kubernetes、Docker Compose）中占据核心地位 。

# 常见函数

## safe_load() 

功能强大但危险。它会执行 YAML 文件中包含的任何 Python 代码。如果 YAML 文件来自不可信的来源（比如用户上传），黑客可以在里面写恶意代码，导致你的电脑被攻击。

safe_load() 会自动把 YAML 的语法映射为 Python 的对象：

key: value	{"key": "value"} (字典)<br>\- item1\<br>- item2	["item1", "item2"] (列表)<br>count: 10	{"count": 10} (整数)<br>enabled: true	{"enabled": True} (布尔值)

示例

```py
def load_yaml(path: Path) -> dict[str, Any]:
    with path.open("r", encoding="utf-8") as f:
        return yaml.safe_load(f) or {}
```



## yaml.safe_load()

安全且推荐。它只支持标准的 YAML 数据类型（如字符串、数字、列表、字典、布尔值等），不会执行任何代码。

