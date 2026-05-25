# 介绍

Rich 是一个 Python 库，用于在终端中输出富文本和精美格式。它可以让命令行界面的输出更加美观、易读和专业。

主要功能
🎨 彩色文本：支持多种颜色和样式
📊 表格：漂亮的表格输出
📝 Markdown：渲染 Markdown 格式
📋 代码高亮：语法高亮显示
📈 进度条：美观的进度指示
🌳 树形结构：展示层级关系

# 常见类使用

## Console

Console 是 Rich 的核心类，负责管理终端输出。

传统 print

```py
print("准备数据集...")
print("错误：文件未找到")
print("成功！处理了 100 个文档")
"""
准备数据集...
错误：文件未找到
成功！处理了 100 个文档
"""
```

使用 Rich Console

```py
from rich.console import Console

console = Console()

console.print("准备数据集...")
console.print("[red]错误：文件未找到[/red]")
console.print("[green]成功！处理了 100 个文档[/green]")
console.print("[bold yellow]警告：数据可能不完整[/bold yellow]")
"""
准备数据集...
错误：文件未找到          ← 红色
成功！处理了 100 个文档   ← 绿色
警告：数据可能不完整      ← 黄色加粗
"""
```

### console 的主要功能

1. 彩色文本

```py
console.print("[red]红色文字[/red]")
console.print("[green]绿色文字[/green]")
console.print("[blue]蓝色文字[/blue]")
console.print("[bold]加粗文字[/bold]")
console.print("[italic]斜体文字[/italic]")
console.print("[underline]下划线文字[/underline]")

# 组合样式
console.print("[bold red]红色加粗[/bold red]")
console.print("[italic green]绿色斜体[/italic green]")
```

2. 表情符号

```py
console.print(":rocket: 启动成功！")
console.print(":warning: 警告信息")
console.print(":white_check_mark: 完成")
console.print(":x: 失败")
"""
🚀 启动成功！
⚠️ 警告信息
✅ 完成
❌ 失败
"""
```

3. 日志级别样式

```py
console.log("普通日志")
console.print("[dim]调试信息[/dim]")
console.print("[yellow]警告信息[/yellow]")
console.print("[red]错误信息[/red]")
```

4. 状态更新

```py
from rich.live import Live
from rich.spinner import Spinner

with Live(Spinner("dots", text="处理中..."), refresh_per_second=10):
    # 执行耗时操作
    time.sleep(3)
# ⠋ 处理中...
```

## table

Table 类用于创建格式化的表格，让数据展示更清晰。

实际应用

```py
@app.command()
def prepare(dataset: str = "demo", limit_docs: int | None = None) -> None:
    """Prepare raw data into standard parquet tables."""
    result = prepare_dataset(dataset, limit_docs=limit_docs)
    
    # 创建表格
    table = Table(title="Prepare Result")  # 设置标题
    table.add_column("Field")              # 添加列
    table.add_column("Value")
    
    # 添加行
    table.add_row("dataset", result.dataset)
    table.add_row("processed_dir", str(result.processed_dir))
    table.add_row("documents", str(result.documents))
    table.add_row("pages", str(result.pages))
    table.add_row("nodes", str(result.nodes))
    table.add_row("queries", str(result.queries))
    table.add_row("message", result.message)
    
    # 输出表格
    console.print(table)
    
"""
┏━━━━━━━━━━━━━━━━┳━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃                   Prepare Result                      ┃
┡━━━━━━━━━━━━━━━━╇━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┩
│ Field          │ Value                              │
├────────────────┼────────────────────────────────────┤
│ dataset        │ demo                               │
│ processed_dir  │ /path/to/data/processed/demo       │
│ documents      │ 2                                  │
│ pages          │ 3                                  │
│ nodes          │ 4                                  │
│ queries        │ 3                                  │
│ message        │ Demo dataset prepared.             │
└────────────────┴────────────────────────────────────┘
"""
```

### table 的高级功能

1. 列对齐和样式

```py
table = Table(title="评估指标")
table.add_column("Metric", style="cyan")           # 青色
table.add_column("Value", justify="right", style="magenta")  # 右对齐，紫色

table.add_row("Page Recall@1", "0.8500")
table.add_row("MRR", "0.7234")
table.add_row("nDCG@5", "0.8912")

console.print(table)
"""
┏━━━━━━━━━━━━━━━━┳━━━━━━━━━━┓
┃     评估指标               ┃
┡━━━━━━━━━━━━━━━━╇━━━━━━━━━━┩
│ Metric         │    Value │
├────────────────┼──────────┤
│ Page Recall@1  │   0.8500 │
│ MRR            │   0.7234 │
│ nDCG@5         │   0.8912 │
└────────────────┴──────────┘
"""
```

2. 条件样式

```py
table = Table(title="实验结果对比")
table.add_column("Method")
table.add_column("Score")

methods = [
    ("BM25", 0.65),
    ("Dense", 0.82),
    ("Hybrid", 0.89),
]

for method, score in methods:
    # 根据分数设置颜色
    if score >= 0.8:
        style = "green"
    elif score >= 0.7:
        style = "yellow"
    else:
        style = "red"
    
    table.add_row(method, f"{score:.2f}", style=style)

console.print(table)
```

3. 边框样式

```py
# 不同的边框风格
table = Table(box=box.ROUNDED)    # 圆角
table = Table(box=box.DOUBLE)     # 双线
table = Table(box=box.MINIMAL)    # 最小化
table = Table(box=box.SIMPLE)     # 简单
```

4. 添加标记

```py
table = Table()
table.add_column("Status")
table.add_column("Task")

table.add_row("[green]✓[/green]", "数据准备")
table.add_row("[green]✓[/green]", "索引构建")
table.add_row("[yellow]⟳[/yellow]", "模型训练")
table.add_row("[red]✗[/red]", "结果评估")

console.print(table)
```

