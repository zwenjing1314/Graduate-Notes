# 介绍

Polars 是一个用 Rust 编写的高性能 Python 数据分析库，专为快速、低内存占用的结构化数据处理设计，常被视为 Pandas 的现代化替代品。

# 常见API

## read_parquet() 

- 作用：它是 Polars 库提供的专门用于读取 Apache Parquet 格式文件的函数。
- 为什么用它？：Parquet 是一种“列式存储”格式，非常适合数据分析。Polars 读取它的速度比传统的 Pandas 快得多，而且内存占用更低。
- 返回类型：它返回一个 pl.DataFrame（Polars 的数据框对象）。你可以把它想象成一个功能超级强大的 Excel 表格或者数据库表，里面包含了所有的行和列。

示例

```py
import polars as pl

def read_records(path: Path, model: type[T]) -> list[T]:
    if not path.exists():
        raise FileNotFoundError(f"Missing required table: {path}")
    rows = pl.read_parquet(path).to_dicts()
    cleaned = [{key: _unjsonify(key, value) for key, value in row.items()} for row in rows]
    return [model(**row) for row in cleaned]
```

.to_dicts() 的作用

在 pl.read_parquet(path).to_dicts() 中：

- 作用：将刚才读进来的 DataFrame 转换成 Python 原生的字典列表（list[dict]）。
- 转换示例： 假设 Parquet 文件里有两行数据：

```py
    # 转换前 (DataFrame):
    | doc_id | title       |
    |--------|-------------|
    | doc_1  | 万科年报    |
    | doc_2  | 比亚迪年报  |

    # 转换后 (rows):
    [
        {"doc_id": "doc_1", "title": "万科年报"},
        {"doc_id": "doc_2", "title": "比亚迪年报"}
    ]
```

## write_parquet()

作用：将内存中的 DataFrame 以 Apache Parquet 格式保存到指定的路径。

- 为什么要用 Parquet？
  - 体积小：它会自动压缩数据，比 CSV 小得多。
  - 速度快：读取速度极快，尤其是当你只需要读取其中几列时（列式存储的优势）。
  - 保留类型：它能记住哪些列是数字、哪些是日期，下次读出来不用重新转换。
- 参数 path： 就是你想要保存文件的完整路径，例如 data/processed/demo/documents.parquet
