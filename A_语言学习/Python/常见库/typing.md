# 介绍

Python的typing库提供了类型提示的类和函数，用于支持静态类型检查并增强代码可读性。

# 常见API

## Any 

表示“什么类型都行”（但会让类型检查“放弃严格”）

### 1) `Any` 的含义

`Any` 是 typing 里最“宽松”的类型，意思是：

- 这个位置可以是任意类型
- 静态类型检查器通常会对 `Any` 放宽检查（很多潜在错误不会被报出来）

所以 `Any` 常用于：

- JSON / YAML / 不同数据源的“杂项元数据”
- 你还没设计好结构、或结构确实不可控的扩展字段

### 2) Any 代码用法

```py
    # Keep anything dataset-specific here for forward compatibility
    meta: dict[str, Any] = field(default_factory=dict)
```

### 3） Any  的常见坑与替代方案

```py
# Keep anything dataset-specific here for forward compatibility
meta: dict[str, Any] = field(default_factory=dict)
```

## Literal

把取值范围限制为“指定的几个常量”

### 1) `Literal` 的含义

`Literal[...]` 用于声明：一个变量/字段的值必须是列出的某些常量之一（通常是字符串、数字、布尔值等）。

比如：

- `Literal["train", "val", "test"]` 表示只能取这三个字符串之一
- 好处是：拼写错误（如 `"trian"`）在静态检查时就能被发现；IDE 自动补全也更强

### 2) Literal 代码用法

```py
DatasetName = Literal["docvqa", "chartqa", "unknown"]
SplitName = Literal["train", "val", "test", "unknown"]
```

会带来两个直接收益：

- 约束语义：dataset/split 不再是随便一个 `str`，而是“项目定义的受控集合”
- 更好的开发体验：IDE 会提示可选值，减少魔法字符串错误

### 3) `Literal` 的边界：它主要是“静态约束”，不是运行时校验

`Literal` 自身不会在运行时自动 `raise`。也就是说你仍然可以运行下面这种代码（除非你加了额外校验/类型检查步骤）：

sample.dataset_name = "docvqqa"  *# 拼写错了：类型检查器会报，但运行时不一定报*

如果你希望运行时也强制校验，常见做法是：

- 在 `__post_init__` 里手动检查（你这个文件已经有一些字段清洗/校验逻辑）
- 或改用 `Enum` 并在赋值/解析阶段验证