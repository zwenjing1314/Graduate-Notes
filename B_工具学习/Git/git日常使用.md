## 一些小命令

```
git status --short && git branch --show-current && git remote -v && tail -n 100 docs/run_project/02_git_use.md
```

**命令串联符 `&&`**
前一条命令执行成功（退出码为0）后，才执行下一条；若中途报错则停止，适合做状态检查链。

**1. `git status --short`**
**简洁模式查看文件状态**（输出两列状态码 + 文件名）

- **第一列**：暂存区（Stage）状态（`M`修改，`A`新增，`D`删除）
- **第二列**：工作区（Worktree）状态（`M`修改，`D`删除，`??`未跟踪）
- *示例*：`MM file` = 暂存区和工作区都修改了；`?? new.txt` = 全新未跟踪

**2. `git branch --show-current`**
**仅打印当前所在分支名**（不带列表，无 `*` 号，纯文本输出，方便脚本调用）

**3. `git remote -v`**
**查看远程仓库别名及地址**（`-v` = verbose 详细模式）

- 显示 `fetch`（拉取）和 `push`（推送）对应的 URL
- 通常默认远程别名是 `origin`

**4. `tail -n 100 docs/run_project/02_git_use.md`**
**查看该 Markdown 文档的末尾 100 行**

- 适用于快速查阅文档底部的更新日志、注意事项或附录指令，不用翻页打开整个文件

## 分支处理命令

**创建分支并切换过去**

```shell
git switch -c paper/evidence-set-retrieval
```

**查看本地所有分支**

```shell
git branch
```

**查看远程所有分支**

```shell
git branch -r
```

**查看本地+远程所有分支**

```shell
git branch -a
```

**删除本地分支**

安全删除

```shell
git branch -d <分支名>
```

强制删除

```shell
git branch -D <分支名>
```

**切换分支**

```shell
git switch paper/evidence-set-retrieval
```

