## 配置.gitignore文件

```
# 第一步：忽略data目录下的所有内容
data/*

# 第二步：但不要忽略.gitkeep文件
!data/.gitkeep
# 数据目录 - 保留空目录结构但忽略内容
data/*
!data/.gitkeep
!data/README.md

# 模型权重文件 - 保留目录但忽略.pth文件
weights/models/
weights/models/*
!weights/.gitkeep
!weights/README.md

# 输出文件
outputs/

# Python缓存
__pycache__/
*.py[cod]
*$py.class
*.so
.Python

# 包和构建文件
build/
develop-eggs/
dist/
downloads/
eggs/
.eggs/
lib/
lib64/
parts/
sdist/
var/
wheels/
*.egg-info/
.installed.cfg
*.egg

# IDE文件
.vscode/
.idea/
*.swp
*.swo
*~

# 系统文件
.DS_Store
Thumbs.db

# 环境文件
.env
.venv
env/
venv/
ENV/
env.bak/
venv.bak/

# 其他大文件
*.pth
*.pt
*.bin
*.h5
*.hdf5
*.pkl
*.pickle
```

## 上传项目到GitHub

**打开终端/命令提示符**，进入项目目录：

```bash
cd C:\Users\WenJin\Documents\GitHub\Rust-Project
```

**初始化Git仓库**：

```bash
git init
```

**检查要上传的文件**：

```bash
git status
```

**添加文件到暂存区**：

```bash
git add .
# 或者逐个添加
git add README.md requirements.txt .gitignore experiments/ tools/ 
# 注意：根据.gitignore，data/和weights/等不会被添加
```

**配置邮箱用户**

```bash
git config --global user.email "你的邮箱"
git config --global user.name "你的名字"

# 仅当前仓库配置（不加 --global）：
git config user.email "你的邮箱"
git config user.name "你的名字"仅当前仓库配置（不加 --global）：
```

**提交更改**：

```bash
git commit -m "Initial commit: RS-Prompt-WarmStart project"
```

**连接到远程仓库**：

```bash
# 首先获取GitHub仓库的URL
# 在GitHub仓库页面点击"Code"按钮，复制HTTPS URL
git remote add origin https://github.com/你的用户名/RS-Prompt-WarmStart.git
```

**推送代码**：

```bash
git branch -M main
# 你可以把 origin 想象成一个“快捷方式”的名字，它代表了你那长长的 GitHub 链接。
git push -u origin main
```

## 将修改后的项目上传到github

### 检查当前状态

```bash
# 查看哪些文件被修改/新增
git status
```

### .gitignore误忽略图片

如果.gitignore中有规则忽略了.png文件或.assets文件夹：

```bash
git add .\.assets\image-20260117110956250.png

# 临时覆盖忽略规则
git add -f .assets/image-20260117110956250.png

# 或者修改.gitignore，然后重新添加
git add .
```

### 确认.gitignore是否生效

```bash
# 1. 检查.gitignore规则是否正确
cat .gitignore

# 2. 如果应该忽略的文件已被跟踪，需要从Git中移除（但不删除本地文件）
git rm -r --cached data/
git rm -r --cached weights/models/
git rm -r --cached outputs/
```

### 添加修改的文件

```bash
# 添加所有修改和新增的文件（会跳过.gitignore中指定的文件）
git add .
```

或者更安全的方式（逐个检查）：

```bash
# 先查看会被添加的文件
git add -n .
# 如果没有问题，再实际添加
git add .
```

### 提交更改

```bash
# 提交修改，并添加描述信息
git commit -m "更新项目：添加了新功能X，修复了bug Y，新增了文件夹Z"
```

建议使用具体的提交信息：

```bash
git commit -m "feat: 新增实验配置模块
- 添加exp04_cross_dataset模块
- 优化数据加载逻辑
- 修复内存泄漏问题"
```

### 推送到GitHub

```bash
# 推送到远程仓库的main分支
git push origin main
```

如果遇到错误（可能是因为其他人也修改了代码）：

```bash
# 先拉取远程最新代码
git pull origin main

# 如果有冲突，解决冲突后再次提交
git add .
git commit -m "解决合并冲突"
git push origin main
```

## 更新项目

### 简单流程

```bash
# 1. 开始工作前，先拉取最新代码
git pull origin main

# 2. 进行你的修改...
#    （编辑文件、新增功能等）

# 3. 提交修改
git add .
git commit -m "你的提交信息"

# 4. 再次拉取（防止别人在此期间推送了代码）
git pull origin main

# 5. 推送
git push origin main
```

## 拉取项目

### 本地有**未提交**的修改

如果工作区（working directory）或暂存区（staging area）还有未提交的改动，直接执行 `git pull` 通常会失败，并出现类似以下的错误提示：

text

```
error: Your local changes to the following files would be overwritten by merge:
    <文件名>
Please commit your changes or stash them before you merge.
Aborting
```



**原因**：Git 为了保护你的本地修改不被远程提交覆盖，默认拒绝在“脏”工作区下进行合并。

**解决方法**：

- **提交修改**：如果这些修改是完整的，可以先 `git add` 和 `git commit` 提交到本地仓库，然后再执行 `git pull`。
- **暂存修改**：如果暂时不想提交，可以用 `git stash` 将当前修改暂时保存到一个栈中，使工作区恢复干净。然后执行 `git pull` 拉取远程更新，最后用 `git stash pop` 将暂存的修改重新应用到工作区。如果有冲突，则需要解决冲突。

### 本地有**已提交**的修改，但未推送到远程

这种情况下，你的本地分支和远程分支已经“分叉”（diverged）：你有一些本地提交，远程也有新的提交。此时执行 `git pull` 会执行两个步骤：

1. `git fetch`：下载远程的最新提交到本地（但不合并）。
2. `git merge`：将远程分支合并到当前本地分支（默认是 merge 操作，会产生一个新的合并提交）。

**可能的结果**：

- **自动合并成功**：如果修改的文件不冲突，Git 会自动创建一个合并提交（merge commit），将远程和本地的修改合并在一起。之后你可以继续工作或推送。
- **产生冲突**：如果双方修改了同一个文件的同一部分，Git 无法自动合并，会提示冲突，需要你手动解决。

**示例流程**：

```bash
# 假设你在 main 分支上有本地提交 A，远程有新的提交 B
git pull
# 如果自动合并成功，会生成合并提交 M
# 如果有冲突，Git 会提示，需要手动解决冲突后提交
```

### 处理合并冲突

如果 `git pull` 导致冲突，Git 会标记冲突文件，你需要：

- 打开冲突文件，查找 `<<<<<<<`、`=======`、`>>>>>>>` 标记，手动编辑文件保留正确内容。
- 编辑完成后，用 `git add` 标记为已解决。
- 最后执行 `git commit` 完成合并（如果是 merge 操作，Git 会自动生成合并提交信息）。
- 若想放弃合并，可以用 `git merge --abort` 回到合并前的状态。

### 使用 `git pull --rebase` 避免合并提交

如果你希望保持提交历史的线性，可以用变基（rebase）代替合并：

```bash
git pull --rebase
```

这会将你的本地提交“放在”远程提交之后，相当于先应用远程提交，再逐个应用你的本地提交。如果遇到冲突，解决后执行 `git rebase --continue`，最后你的历史就是一条直线。但需注意：**不要对已推送到公共仓库的提交进行变基**，否则会给他人的工作带来麻烦。

------

## 撤销暂存文件

撤销所有已暂存的文件

```bash
git reset
```

撤销特定的文件

```bash
git reset <文件路径>
```



## 中文显示问题

```bash
git config --global core.quotepath false
```



## 修改代理

打开[Git CMD](https://zhida.zhihu.com/search?content_id=736588962&content_type=Answer&match_order=1&q=Git+CMD&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3Njg4Nzk1NjAsInEiOiJHaXQgQ01EIiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6NzM2NTg4OTYyLCJjb250ZW50X3R5cGUiOiJBbnN3ZXIiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.87WigpNgB9PCyTpoT5PkUUGIouQgRs67Pqtp8h-QXBw&zhida_source=entity)，添加全局代理

这里面用的是clash verge代理

```bash
git config --global http.proxy http://127.0.0.1:7897
git config --global https.proxy http://127.0.0.1:7897
```

验证代理设置:

```bash
git config --global --get http.proxy
git config --global --get https.proxy
```

取消代理:

```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```



## 从暂存区移除 .idea文件夹

```bash
git rm -r --cached .idea
```

这里的 `-r` 表示递归（recursive），用于处理整个文件夹；`--cached` 表示只从 Git 的追踪缓存中移除，保留本地磁盘上的实体文件：

## 远程仓库链接出现问题

### 确认当前的 `origin` 地址是否正确

虽然 `origin` 已经存在，但为了保险起见，我们需要确认它指向的是不是你想要的那个 GitHub 仓库。在终端输入：

```bash
git remote -v
```

### 修改远程仓库地址（仅在地址错误时执行）

```bash
git remote set-url origin https://github.com/zhouwengjing/Code-repository.git
```





## PAT(Personal Access Token)

GitHub提供的一种用于替代密码的身份验证凭证

设置入口

右上角头像 → Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token。

 GitHub 目前支持 Fine-grained tokens 和 Tokens (classic) 两种，官方更推荐前者。 你按这个填就行： Token name：随便起名，比如 macbook-git-push Expiration：选 30 天或 90 天都可以 Resource owner：选你的账号，通常就是 zhouwenjing Repository access：选 Only select repositories 只勾选你的仓库 CLIP_Stu Repository permissions 里给 Contents 至少开到 Read and write 这是为了让 token 只访问你需要的仓库，并具备 push 所需的写权限；GitHub 也建议尽量只给最小必要权限。

按这个顺序点： 

在当前页面这样选 Resource owner 选 zhouwengjing

 Repository access 选 Only select repositories 然后选 zhouwengjing/CLIP_Stu 

Permissions 这里： 保持在 Repositories 这个标签上 不要切到 Account 点右边 Add permissions 在弹出的搜索框里输入：Contents 找到 Contents 把它设成 Read and write 

Account 保持 0，什么都不要加



linux
<<<<<<< HEAD

=======
>>>>>>> 26317f8 (3.16)
