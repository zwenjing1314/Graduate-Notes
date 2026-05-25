# 介绍

‌**Typer 是一个基于 Python 类型提示（Type Hints）构建命令行界面（CLI）应用程序的现代 Python 库**‌，旨在简化 CLI 工具的开发过程，提升代码可读性与开发效率。

# 第一次试用

1. ## 应用实例创建

```py
app = typer.Typer(help="Multi-modal document evidence retrieval experiments.")
```

首先创建一个 Typer 应用实例，这是所有命令的容器。

2. ## 命令注册过程

当 Python 解释器遇到：

```py
@app.command()
def prepare(dataset: str = "demo", limit_docs: int | None = None) -> None:
    """Prepare raw data into standard parquet tables."""
    # 函数体...
```

Typer 会执行以下操作：

a) 函数签名分析

- 解析函数名 prepare → 成为子命令名 mdr prepare
- 分析参数类型注解和默认值
- 提取 docstring 作为命令帮助文本

b) 参数映射

```py
dataset: Annotated[str, typer.Option(help="Dataset name...")] = "demo"
```

- typer.Option() 表示这是一个可选参数（--dataset）
- Annotated 结合类型提示和元数据
- 自动生成对应的命令行参数：--dataset 和 --limit-docs

c) 命令注册

将函数注册到 app 的命令列表中，建立映射关系：

```py
"prepare" → prepare函数
"retrieve" → retrieve函数
"evaluate" → evaluate函数
"export-demo" → export_demo函数
```

3. ## 运行时执行流程

当用户执行 uv run mdr prepare --dataset demo 时：

```py
1. uv run → 激活虚拟环境并运行 Python
2. mdr → 调用 pyproject.toml 中定义的入口点: "mmdocrag.cli:app"
3. Typer 解析命令行参数:
   - 识别子命令: "prepare"
   - 解析选项: --dataset demo
4. 调用对应的 prepare 函数，传入解析后的参数
5. 执行函数逻辑
```

4. ## 自动生成的功能

Typer 基于类型注解自动生成：

帮助信息

```py
$ uv run mdr --help
Multi-modal document evidence retrieval experiments.

Usage: mdr [OPTIONS] COMMAND [ARGS]...

Commands:
  prepare      Prepare raw data into standard parquet tables.
  retrieve     Run retrieval for an experiment config.
  evaluate     Evaluate a retrieval run.
  export-demo  Export an opening-defense-ready markdown result table.
```

```py
$ uv run mdr prepare --help
Usage: mdr prepare [OPTIONS]

  Prepare raw data into standard parquet tables.

Options:
  --dataset TEXT        Dataset name: demo, mmdocir, cn_annual_reports.
                        [default: demo]
  --limit-docs INTEGER  Optional document limit for quick experiments.
  --help                Show this message and exit.
```

参数验证

- 类型检查：如果 --limit-docs 传入非整数，自动报错
- 必填项检查：标记为 ... 的参数必须提供
- 默认值处理：未提供时使用默认值

错误处理

```py
raise typer.Exit(1) from exc  # 优雅地退出并返回错误码
```

