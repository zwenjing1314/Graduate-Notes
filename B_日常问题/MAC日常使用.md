## Homebrew下载"假锁"

手动关闭了tesseract的下载,现在重新下载报了如下错误该怎么解决?

```
(base) zhouwenjing@zhouwenjings-MacBook-Pro ~ % brew
  install tesseract-lang
  ==> Fetching downloads for: tesseract-lang
  ✔︎ Bottle Manifest tesseract-lang (4.1.0)                                                     Downloaded    6.7KB/
  6.7KB
  ✘ Bottle tesseract-lang (4.1.0)
  Error: A `brew install tesseract-lang` process has already locked /Users/zhouwenjing/Library/Caches/Homebrew/
  downloads/c9ce904ddb95bef037f58a8739c202fa6b3b698d7b5eda9c2e69dd46d4a2e9be--tesseract-
  lang--4.1.0.all.bottle.2.tar.gz.incomplete.
  Please wait for it to finish or terminate it to continue.

```

这是 Homebrew 的下载锁没释放，不一定是还在下载，很多时候是你上次中断后留下了一个“假锁”。

 按这个顺序处理：

1. 先确认没有残留的 brew 进程

```bash
 ps aux | grep brew
```

2. 如果看到还在跑的 brew install tesseract-lang 或相关进程，先结束它

```bash
 kill <PID>
```

3. 如果确认没有 Homebrew 进程了，删掉那个未完成下载文件

```bash
 rm /Users/zhouwenjing/Library/Caches/Homebrew/downloads/

 c9ce904ddb95bef037f58a8739c202fa6b3b698d7b5eda9c2e69dd46d4a2e9be--tesseract-lang--4.1.0.all.bottle.2.tar.gz.incomplete
```

4. 然后重新安装

```bash
 brew install tesseract-lang
```

 如果还报缓存相关问题，再执行一次清理：

```bash
 brew cleanup
 brew install tesseract-lang
```

 如果你想更稳一点，也可以直接把这个包的缓存删掉后重装：

```bash
 rm -f ~/Library/Caches/Homebrew/downloads/*tesseract-lang*
 brew install tesseract-lang
```

 关键点是：

 \-  有 brew 进程时，不要删锁文件

 \-  没有 brew 进程时，删 .incomplete 文件即可

