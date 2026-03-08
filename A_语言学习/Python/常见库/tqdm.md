# 介绍

tqdm是一个快速、可扩展的Python进度条库，用于在Python长循环中添加一个进度提示信息，用户只需封装任何迭代器 `tqdm(iterator)`。

# 常用的API

## tqdm()

传入可迭代对象

```py
# tqdm.tqdm实现进度条
import time
from tqdm import *

for i in tqdm(range(1000)):
	time.sleep(0.1)
```

## trange()

作为tqdm(range())的简洁替代，如下例

```py
from tqdm import trange
import time

for i in trange(10):
	timte.sleep(0.5)
```



# 踩过的坑+解决方法