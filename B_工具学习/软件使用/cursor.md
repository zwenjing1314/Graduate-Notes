## 配置conda环境

### 1) 安装/启用 Python 扩展

- 打开 Extensions（扩展）
- 搜索并安装 Python（Microsoft 出品，通常叫 `Python`）

（一般还会自动提示装 `Pylance`，装上更好。）

### 2) 选择 Conda 环境解释器（最关键）

- 打开命令面板：`Ctrl+Shift+P`
- 搜索：`Python: Select Interpreter`
- 在列表里选你的 Conda 环境，例如：
  - `base (conda)`
  - 或 `your_env_name (conda)`
  - 路径一般类似 `.../anaconda3/envs/xxx/bin/python`

选完后，Cursor 右下角通常会显示当前 Python 解释器。

## 设置UI大小

- 打开设置（`Ctrl+,`）
- 搜索 `zoom`
- 把 Window: Zoom Level 调大（比如从 `0` 改到 `2`）

## 终端复制文本快捷键修改

写入的文件是：`~/.config/Cursor/User/keybindings.json`，内容是给终端加了一个条件快捷键（仅在 `terminalTextSelected` 时触发复制）。

```json
[
  {
    "key": "ctrl+c",
    "command": "workbench.action.terminal.copySelection",
    "when": "terminalFocus && terminalTextSelected"
  }
]
```

