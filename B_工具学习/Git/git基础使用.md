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

### 先拉取最新代码再上传

```bash
# 第一步：拉取远程最新代码并 rebase（你之前的第一条命令）
git pull --rebase origin main

# 第二步：把你的修改推送到 GitHub
git push origin main
```

### **合并本地和远程历史**

```bash
git pull origin main --no-rebase --allow-unrelated-histories
```

- --no-rebase：告诉 Git 用 merge 方式拉取
- --allow-unrelated-histories：允许合并“两套完全不相关的历史”

### **用本地直接覆盖远程**

如果确定远程仓库里的内容都不要了，只保留本地现在这份代码，可以直接强推：

```bash
git push -u origin main --force
```

更安全一点可以用：

```bash
git push -u origin main --force-with-lease
```

这会覆盖远程 main，所以只有在确认远程内容不需要时才这么做。

## 拉取项目

**养成习惯**：在每次开始写新代码前，先执行 `git pull --rebase origin main`，确保基础代码是最新的，能极大减少后续冲突。

**善用 Stash**：如果你写到一半想拉取代码，但又不想现在就 commit（因为功能还没写完），`git stash` 是你最好的朋友。

### 基础拉取命令

```
# 标准拉取（Merge 模式）
git pull origin main

# 变基拉取（Rebase 模式）—— 推荐
git pull --rebase origin main
```

- 地的修改合并在一起。之后你可以继续工作或推送。
- **产生冲突**：如果双方修改了同一个文件的同一部分，Git 无法自动合并，会提示冲突，需要你手动解决。

### 当“本地”与“远程”都有修改时

这种情况非常普遍，处理方式取决于你的**本地修改是否已经执行过 `git commit`**。

#### 场景 A：本地修改已 commit（产生冲突）

当你运行 `git pull --rebase` 时，如果远程和本地修改了**同一文件的同一行**，Git 会暂停并报错，提示“Conflict”。

**处理步骤：**

1. **找到冲突文件**：终端会显示具体文件名。
2. **手动解决**：打开文件，你会看到 `<<<<<<<`, `=======`, `>>>>>>>` 这种标记。删除标记并保留你想要的代码。
3. **标记解决**：

```bash
git add <冲突的文件名>
```

4. **继续流程**：

```bash
git rebase --continue
```

注意：此时不需要再执行 commit，Git 会自动帮你完成。

#### 场景 B：本地修改还没 commit（拉取失败）

如果你本地有还没保存的改动（Modified），Git 通常会拒绝 pull，提示“Your local changes... would be overwritten”。

**处理方式（Stash 暂存法）：**

1. **把改动藏起来**：

```bash
git stash
```

2. **正常拉取**：

```bash
git pull --rebase origin main
```

3. **把改动拿回来**：

```bash
git stash pop
```

## 克隆仓库

**一般正确方式**
如果这个项目本来就是你的远程仓库，最推荐：

```
git clone 仓库地址
```

比如：

```
git clone https://github.com/zwenjing1314/DocU_Demo.git
```



## 分支管理

1. **创建新分支**

```bash
git switch -c ocr-pipeline-enhancements
```

2. **更换分支**

```bash
# 检查分支状态
git branch

git switch main
```

3. **上传新分支**

```bash
  git add .
  git add ocr_inspector/README.md ocr_inspector/app.py ocr_inspector/ocr_engine.py ocr_inspector/web/index.html
  
  
  git commit -m "feat(ocr_inspector): add OCR enhancement pipeline and tuning options"
  git push -u origin ocr-pipeline-enhancements
```

-u是设置upstream

`-u` 的出现并不是因为分支不同，而是**因为“首次推送”与“已存在追踪关系”的区别**

`-u` 的出现仅表示“**我是第一次推送这个分支，顺便帮我记住上游，下次少打几个字**”

git switch -c 是现在更推荐的新命令。

git checkout -b，是因为它是 Git 里长期存在的经典写法，很多教程和团队还在用。但如果我们按更现代、语义更清晰的方式来写，应该优先用：

```bash
git switch -c feature/retrieve-webpage-content

# 创建并切换到一个新分支
```

而老写法是：

```bash
git checkout -b feature/retrieve-webpage-content

# 两者效果基本一样，都是：
# 从当前分支创建 feature/retrieve-webpage-content，并立刻切过去
```

区别在于：

- git checkout 是老命令，功能很多，既能切分支，也能恢复文件，语义比较混杂。
- git switch 是较新的命令，专门用于切换分支，读起来更清楚。
- git restore 则用于恢复文件，和 switch 一起替代了 checkout 的一部分职责。



• 可以。Git 里的“分支”可以理解成：从当前代码状态拉出来的一条独立开发线。

  你现在在 main 分支，通常 main 表示项目的主线代码，比如稳定版本、最终版本。新建
  分支后，你可以在新分支上修改代码、做实验、写新功能，而不会直接影响 main。

  查看当前分支：

  git branch

  带 * 的就是你当前所在分支，例如：

  * main

  从当前 main 新建一个分支并切换过去：

  git switch -c 新分支名

  例如：

  git switch -c feature/login

  老一点的写法也可以：

  git checkout -b feature/login

  这两个命令的意思都是：基于当前分支创建一个新分支，并切换到新分支上。

  新分支的作用主要有这些：

  1. 隔离修改

  你可以在新分支上随便改代码，不会马上影响 main。

  2. 开发新功能

  例如：

  git switch -c feature/search

  然后专门在这个分支里开发搜索功能。

  3. 修复 bug

  例如：

  git switch -c fix/login-error

  专门修登录错误。

  4. 方便合并

  当新分支上的功能完成后，可以再把它合并回 main：

  git switch main
  git merge feature/login

  5. 方便回退和试验

  如果你在新分支上试错失败，可以直接切回 main，主分支不会被污染。

  一个常见流程是：

  git switch main
  git pull
  git switch -c feature/my-work

  ### 修改代码
  git add .
  git commit -m "完成某个功能"
  git switch main
  git merge feature/my-work

  如果你只是想现在新建一个分支，最常用命令就是：

  git switch -c my-new-branch

  分支名可以自己取，比如 test、dev、feature-note、fix-bug 等。



• feature/your-branch-name 里的 feature/ 不是必须的。
  它只是团队里常见的一种命名习惯，用来把分支按类型分类，比如：

  - feature/... 或 feat/...：新功能
  - fix/...：修复
  - chore/...：杂项
  - docs/...：文档

  Git 和 GitHub 都不要求你一定加这个前缀。你一个人用仓库，直接写成 ocr-pipeline-enhancements 也完全可以。

  结合你这次改动内容，我更建议：

  - 分支名：ocr-pipeline-enhancements
  - commit message：feat(ocr_inspector): add OCR enhancement pipeline and tuning options

  对应命令直接用这套：

  git switch -c feature/your-branch-name
  git add ocr_inspector/README.md ocr_inspector/app.py ocr_inspector/ocr_engine.py ocr_inspector/web/index.html
  git commit -m "Describe this change"
  git push -u origin feature/your-branch-name

  git switch -c ocr-pipeline-enhancements
  git add ocr_inspector/README.md ocr_inspector/app.py ocr_inspector/ocr_engine.py ocr_inspector/web/index.html
  git commit -m "feat(ocr_inspector): add OCR enhancement pipeline and tuning options"
  git push -u origin ocr-pipeline-enhancements

  如果你想要不用英文前缀、更直白一点，也可以这样：

  - 分支名：improve-ocr-inspector
  - commit message：improve OCR accuracy with preprocessing, fallback scan, and artifact filtering

  如果你愿意，我还可以继续帮你把这次改动整理成更像正式 PR 标题和说明的一版。





• 以后切换分支，最常用就是这两个命令：

  git switch main
  git switch ocr-pipeline-enhancements

  如果想先看看有哪些本地分支：

  git branch

  如果想看远程分支也一起看：

  git branch -a

  切换完分支后，工作区里的代码通常会直接变成那个分支对应的版本，不需要你手动再“换一套代码”。
  也就是说：

  - git switch main：代码会切回 main 的状态
  - git switch ocr-pipeline-enhancements：代码会变成你这个新分支的状态

  但有两个前提你要知道：

  1. 这个分支已经在你本地存在
     你自己刚创建过的分支，之后直接 git switch 分支名 就行。
  2. 你当前没有冲突的未提交改动
     如果你改了文件但还没提交，Git 可能不让你切换，或者把这些改动一起带过去。

  如果某个分支只是 GitHub 上有、本地还没有，那就要先拿下来：

  git fetch origin
  git switch --track origin/ocr-pipeline-enhancements

  以后这个分支建好本地后，再切换就只需要：

  git switch ocr-pipeline-enhancements

  如果你切到某个分支后，还想同步 GitHub 上这个分支的最新内容，再执行：

  git pull

  你可以把它理解成：

  - git switch：切换到哪个分支
  - git fetch：查看并下载远程最新分支信息
  - git pull：把远程这个分支的最新提交拉到本地

  按你现在的情况，最常用的就是：

  git switch main
  git switch ocr-pipeline-enhancements

  如果你愿意，我可以接着给你画一个“本地分支 / GitHub 分支 / switch / pull”之间的最小理解图。

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

>>>>>>> 26317f8
