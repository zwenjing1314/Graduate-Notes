## 代码片段

```py
def make_parser() -> argparse.ArgumentParser:
    p = argparse.ArgumentParser("mmclip_day1")
    sub = p.add_subparsers(dest="cmd", required=True)

    p.add_argument("--log-level", default="INFO")
    p.add_argument("--seed", type=int, default=42)

    # build
    b = sub.add_parser("build", help="Build image index")
    b.add_argument("--image-dir", required=True, help="Folder containing images")
    b.add_argument("--out-dir", default="artifacts", help="Output folder")
    b.add_argument("--model", default="openai/clip-vit-base-patch32")
    b.add_argument("--device", default="cuda")
    b.add_argument("--batch-size", type=int, default=32)
    b.add_argument("--no-amp", action="store_true", help="Disable AMP autocast")
    b.set_defaults(func=build_cmd)

    # search
    s = sub.add_parser("search", help="Search images by text query")
    s.add_argument("--index-dir", default="artifacts", help="Index folder (out-dir)")
    s.add_argument("--query", required=True, help="Text query")
    s.add_argument("--topk", type=int, default=5)
    s.add_argument("--model", default="openai/clip-vit-base-patch32")
    s.add_argument("--device", default="cuda")
    s.add_argument("--no-amp", action="store_true")
    s.set_defaults(func=search_cmd)

    return p
```

### `p.add_subparsers()` 的用法

**作用**：在主解析器（Main Parser）下创建一个“子命令分发器”。

**代码中的体现**：`sub = p.add_subparsers(dest="cmd", required=True)`

**解释**：它告诉程序，“我的脚本不仅有主参数，还要支持多个不同的操作指令”。`dest="cmd"` 表示用户输入的子命令名称（如 `build` 或 `search`）会被解析并存储在 `args.cmd` 变量中。`required=True` 强制要求用户运行时必须输入一个子命令，否则报错。

### `sub.add_parser()` 的用法

**作用**：在刚才创建的“分发器”中，添加一个具体的子命令。

**代码中的体现**：`b = sub.add_parser("build", help="Build image index")`

**解释**：这相当于创建了一个全新的、独立的 ArgumentParser（即变量 `b`），专门用来处理 `build` 这个词后面的参数。在这个子解析器 `b` 上添加的参数（如 `--image-dir`），只有在用户输入 `build` 子命令时才有效。

### `set_defaults(func=...)` 的用法

**作用**：为特定的解析器绑定一个默认的属性（通常是绑定一个执行函数）。

**代码中的体现**：`b.set_defaults(func=build_cmd)`

**解释**：这是 `argparse` 处理子命令的**最佳实践**。如果不这么写，你解析完参数后，需要写冗长的 `if-else` 来判断执行什么：

```py
# 如果不使用 set_defaults 的笨办法：
if args.cmd == "build1":
    build_cmd(args)
elif args.cmd == "search":
    search_cmd(args)
```

使用了 `set_defaults` 后，无论用户调用哪个子命令，都会将对应的处理函数悄悄存入 `args.func` 中。后续只需要极其优雅的一行代码即可执行：

```py
# 使用 set_defaults 后的聪明办法：
args.func(args)
```

