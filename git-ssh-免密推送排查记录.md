# Git Push 每次都要输入用户名和密钥的排查与解决记录

## 一、问题现象

在仓库 `/home/wenjing/文档/WorkStation/Graduate-Notes` 中执行 `git push` 时，每次都会要求输入 GitHub 用户名和密钥。

用户的原始诉求有两个：

1. 先帮忙拉取远程仓库。
2. 再把当前仓库改成 SSH 免密推送。

---

## 二、问题根因

排查后确认，导致每次 `git push` 都要输入用户名和密钥的直接原因是：

1. 当前仓库的远程地址使用的是 `HTTPS`：

   ```bash
   https://github.com/zwenjing1314/Graduate-Notes.git
   ```

2. Git 本地配置中没有看到 `credential.helper`，说明 Git 没有把认证信息缓存下来。
3. GitHub 的 HTTPS 推送现在通常不是输入账号密码，而是输入 `Personal Access Token (PAT)`。
4. 因为仓库使用的是 HTTPS 且没有启用凭据持久化，所以每次推送都会重复要求认证。

---

## 三、解决思路

这次处理的整体思路是：

1. 先确认当前仓库远程地址、分支状态和本地工作区状态，避免直接拉取导致覆盖或冲突。
2. 先执行一次保守的拉取：`git pull --ff-only`，只允许快进更新，避免自动合并。
3. 检查 Git 的认证相关配置，确认问题确实出在 HTTPS 认证方式上。
4. 检查本机是否已经存在可用于 GitHub 的 SSH key。
5. 如果没有 SSH key，则生成一把新的 `ed25519` key。
6. 将仓库远程地址从 HTTPS 改为 SSH。
7. 由于当前网络环境下 `github.com:22` 连接超时，因此改为走 GitHub 提供的 SSH over HTTPS 通道，也就是 `ssh.github.com:443`。
8. 再次验证 SSH 认证是否成功。
9. 验证 `git push` 是否已经不再要求输入用户名和 token。

---

## 四、最终结果

本次已经完成的事项：

1. 仓库远程地址已从 HTTPS 改成 SSH：

   ```bash
   git@github.com:zwenjing1314/Graduate-Notes.git
   ```

2. 已生成 SSH key：

   - 私钥：`~/.ssh/id_ed25519`
   - 公钥：`~/.ssh/id_ed25519.pub`

3. 已把 GitHub 的 SSH 连接改为走 `443` 端口。
4. 已成功通过下面的认证测试：

   ```bash
   ssh -T git@github.com
   ```

   返回结果表明：

   ```text
   Hi zwenjing1314! You've successfully authenticated, but GitHub does not provide shell access.
   ```

这说明 SSH 认证已经正常工作，后续不会再因为 HTTPS 认证而要求输入 GitHub 用户名和 token。

---

## 五、为什么后来 `git push` 仍然失败

在 SSH 已经配置成功后，`git push origin main` 的报错变成了：

```text
! [rejected] main -> main (fetch first)
error: failed to push some refs to 'github.com:zwenjing1314/Graduate-Notes.git'
```

这已经不是认证问题，而是分支同步问题，含义是：

1. 远程 `main` 分支上已经有你本地没有的新提交。
2. Git 拒绝非快进推送，防止把远程历史覆盖掉。
3. 同时本地工作区里还有未提交修改，所以在拉取远程新内容前，需要先处理本地改动。

也就是说：

- “反复输入用户名和密钥”的问题已经解决。
- 当前剩下的是“本地与远程分支历史不同步”的问题。

---

## 六、我实际使用过的排查和解决命令

下面按实际用途整理本次使用过的命令，并说明每条命令的作用和结果。

### 1. 查看远程仓库地址

```bash
git remote -v
```

作用：
查看当前仓库的 fetch/push 远程地址。

结论：
确认当前仓库一开始使用的是 HTTPS：

```bash
origin  https://github.com/zwenjing1314/Graduate-Notes.git (fetch)
origin  https://github.com/zwenjing1314/Graduate-Notes.git (push)
```

---

### 2. 查看当前分支和工作区状态

```bash
git status --short --branch
```

作用：
查看当前所在分支、本地与远程分支的关系，以及是否存在未提交改动。

结论：
发现当前位于 `main` 分支，并且工作区里已经有多处已修改和未跟踪文件。这意味着后续拉取必须保守处理，避免把本地改动弄乱。

---

### 3. 尝试查看当前分支名

```bash
git branch --show-current
```

作用：
确认当前所在分支。

说明：
这一步在当前助手运行环境里第一次执行时受到沙箱限制，没有返回有效结果，但不影响后续判断，因为从 `git status --short --branch` 里已经能确认当前分支是 `main`。

---

### 4. 确认当前目录是否是 Git 仓库

```bash
git rev-parse --is-inside-work-tree
```

作用：
检查当前路径是否处于 Git 工作树内。

说明：
这一步第一次执行时同样受到沙箱限制，没有作为后续判断依据。

---

### 5. 保守拉取远程主分支

```bash
git pull --ff-only origin main
```

作用：
只允许“快进”更新，不允许自动合并。

为什么这样做：
因为本地已经有未提交改动，直接普通 `git pull` 可能引入自动 merge，不够稳妥。

第一次执行结果：

```text
Already up to date.
```

含义：
当时本地与远程 `main` 没有新的差异需要拉取。

---

### 6. 查看 Git 的认证相关配置

```bash
git config --show-origin --get-regexp '^(credential|user\.name|user\.email|remote\.origin\.url)'
```

作用：
查看用户名、邮箱、远程地址和凭据相关配置，并标明这些配置来自哪个文件。

结论：

```text
file:/home/wenjing/.gitconfig user.email 2497672467@qq.com
file:/home/wenjing/.gitconfig user.name zwenjing
file:.git/config remote.origin.url https://github.com/zwenjing1314/Graduate-Notes.git
```

分析：

1. `user.name` 和 `user.email` 已经配置好了。
2. 远程地址仍然是 HTTPS。
3. 没看到 `credential.helper`。
4. 所以可以判断，反复输入认证信息主要是因为 HTTPS 推送加上未启用凭据持久化。

---

### 7. 检查本机是否已有 SSH key

```bash
ls -la /home/wenjing/.ssh
```

作用：
查看 `~/.ssh` 目录里是否已经存在可用私钥。

结论：
目录里只有 `authorized_keys` 和空的 `config`，没有现成可用于 GitHub 的私钥，因此需要新建 SSH key。

---

### 8. 检查 GitHub CLI 是否已安装

```bash
gh --version
```

作用：
确认本机是否安装了 GitHub CLI。

结果：

```text
/bin/bash: line 1: gh: command not found
```

含义：
系统里没有 `gh`，所以不能通过命令行自动把公钥上传到 GitHub。

---

### 9. 检查 GitHub CLI 登录状态

```bash
gh auth status
```

作用：
如果已经安装并登录 `gh`，理论上可以更自动化地绑定 SSH key。

结果：

```text
/bin/bash: line 1: gh: command not found
```

含义：
由于 `gh` 未安装，这条路径无法使用。

---

### 10. 生成新的 SSH key

```bash
ssh-keygen -t ed25519 -C "2497672467@qq.com" -f /home/wenjing/.ssh/id_ed25519 -N ""
```

作用：
生成一把新的 `ed25519` SSH 密钥对，作为 GitHub 的身份认证凭据。

参数解释：

- `-t ed25519`：指定使用 `ed25519` 算法。
- `-C`：添加注释，通常写邮箱。
- `-f`：指定输出文件路径。
- `-N ""`：不设置 passphrase，方便本机免密使用。

结果：
成功生成：

- `/home/wenjing/.ssh/id_ed25519`
- `/home/wenjing/.ssh/id_ed25519.pub`

---

### 11. 将仓库远程地址切换为 SSH

```bash
git remote set-url origin git@github.com:zwenjing1314/Graduate-Notes.git
```

作用：
把仓库远程地址从 HTTPS 改成 SSH。

结果：
执行成功。

---

### 12. 读取公钥内容，准备添加到 GitHub

```bash
cat /home/wenjing/.ssh/id_ed25519.pub
```

作用：
读取刚生成的公钥，用于复制到 GitHub 的 `Settings -> SSH and GPG keys` 页面。

结果：
成功读取公钥内容，并由用户完成了 GitHub 侧添加。

---

### 13. 再次确认远程地址是否已切换成功

```bash
git remote -v
```

作用：
验证 fetch 和 push 地址是否都已经变成 SSH。

结果：

```bash
origin  git@github.com:zwenjing1314/Graduate-Notes.git (fetch)
origin  git@github.com:zwenjing1314/Graduate-Notes.git (push)
```

---

### 14. 尝试获取 GitHub 的 SSH 主机指纹

```bash
ssh-keyscan github.com
```

作用：
提前获取 GitHub 主机指纹，避免第一次 SSH 连接时出现交互确认。

结果：
这一步没有拿到可用结果，后续改为在实际 SSH 连接时使用自动接受新主机指纹的方式处理。

---

### 15. 第一次测试 GitHub SSH 认证

```bash
ssh -o StrictHostKeyChecking=accept-new -T git@github.com
```

作用：
测试 SSH 认证是否成功，并自动接受首次出现的主机指纹。

第一次现象：
命令长时间没有返回。

分析：
不像是 key 配置错误，更像是网络连接阶段被卡住。

---

### 16. 用更严格的方式重试 SSH 认证

```bash
ssh -o BatchMode=yes -o ConnectTimeout=10 -o StrictHostKeyChecking=accept-new -T git@github.com
```

作用：

1. `-o BatchMode=yes`：禁止交互式输入，快速暴露真实错误。
2. `-o ConnectTimeout=10`：10 秒内连不上就退出，避免长时间卡住。
3. `-o StrictHostKeyChecking=accept-new`：自动接受首次主机指纹。

结果：

```text
ssh: connect to host github.com port 22: Connection timed out
```

结论：
不是 SSH key 错误，而是当前网络到 `github.com:22` 的连接超时。

---

### 17. 修改 SSH 配置，让 GitHub 走 443 端口

这一步不是通过 shell 命令完成的，而是直接编辑了文件：

文件：

```bash
~/.ssh/config
```

写入内容：

```sshconfig
Host github.com
  HostName ssh.github.com
  Port 443
  User git
  IdentityFile /home/wenjing/.ssh/id_ed25519
  IdentitiesOnly yes
```

作用：

1. 把原本连 `github.com:22` 改成连 `ssh.github.com:443`。
2. 显式指定使用刚生成的私钥。
3. 避免 SSH 尝试其他无关身份。

为什么有效：
很多网络环境会拦截或限制 `22` 端口，但 `443` 端口通常可通。GitHub 官方支持这种 SSH over HTTPS 的方式。

---

### 18. 再次验证 SSH 认证

```bash
ssh -o BatchMode=yes -o ConnectTimeout=10 -o StrictHostKeyChecking=accept-new -T git@github.com
```

结果：

```text
Warning: Permanently added '[ssh.github.com]:443' (ED25519) to the list of known hosts.
Hi zwenjing1314! You've successfully authenticated, but GitHub does not provide shell access.
```

结论：
SSH 认证已经成功，说明免密推送所需的认证链路已经打通。

---

### 19. 实际验证推送是否还会要求用户名和 token

```bash
git push origin main
```

作用：
直接验证当前仓库是否已经能通过 SSH 正常推送。

结果：

```text
To github.com:zwenjing1314/Graduate-Notes.git
 ! [rejected]        main -> main (fetch first)
error: failed to push some refs to 'github.com:zwenjing1314/Graduate-Notes.git'
```

结论：
推送失败的原因已经不是认证，而是远程分支包含本地没有的新提交。

这说明：

- SSH 免密认证已经生效。
- 问题从“认证失败”切换成了“分支历史需要同步”。

---

### 20. 再次尝试保守拉取远程

```bash
git pull --ff-only origin main
```

作用：
在 SSH 已打通后，再尝试把远程的新提交快进拉下来。

说明：
这次命令执行时出现等待时间过长，后续没有继续拿它作为最终结论，而是回到本地状态确认当前问题。

---

### 21. 再次查看本地状态

```bash
git status --short --branch
```

作用：
重新确认当前分支和本地未提交文件情况。

结论：
本地依然存在多处已修改和未跟踪文件，这也是后续直接拉取和变基前必须谨慎处理的原因。

---

### 22. 检查是否有残留的拉取进程

```bash
pgrep -af "git pull --ff-only origin main"
```

作用：
确认之前等待中的 `git pull` 进程是否还在后台运行。

结果：
发现有残留进程。

---

### 23. 结束残留的拉取进程

```bash
kill 28826
```

作用：
清理卡住的后台 `git pull` 进程，避免继续占用会话。

说明：
这个 `28826` 是当时实际查到的进程号，换一台机器或换一次执行后，这个数字不会固定。

---

## 七、当前仓库状态总结

当前状态可以总结为两句话：

1. SSH 免密推送已经配置成功，GitHub 认证没有问题。
2. 当前不能直接 `git push` 的原因，是远程 `main` 上有新的提交，而本地又有未提交改动。

---

## 八、后续推荐操作

如果想继续把当前仓库同步并推送，可以按下面两种方式选择其一。

### 方案 A：先提交本地改动，再拉取并推送

```bash
git add .
git commit -m "save local changes"
git pull --rebase origin main
git push origin main
```

适用场景：
你已经确认本地这些改动需要保留下来，并准备形成一次正式提交。

---

### 方案 B：先临时保存本地改动，再同步远程

```bash
git stash -u
git pull --rebase origin main
git stash pop
git push origin main
```

适用场景：
你暂时还不想提交本地改动，但又希望先把远程更新拉下来。

---

## 九、这次问题的核心结论

这次问题本质上分成两个层次：

1. 第一层是认证方式问题。
   仓库原来走 HTTPS，所以每次 `git push` 都会要求输入 GitHub 用户名和 token。

2. 第二层是分支同步问题。
   在改成 SSH 并验证成功后，`git push` 被拒绝的原因已经变成远程分支领先于本地，而不是认证失败。

所以最终结论是：

- SSH 免密推送已经配置成功。
- 之后如果还推不上去，优先考虑 `pull/rebase` 和本地未提交改动，而不是再怀疑 SSH 认证。
