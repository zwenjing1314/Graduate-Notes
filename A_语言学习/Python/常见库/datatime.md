# 介绍

以不同格式显示日期和时间是程序中最常用到的功能

# 常见API

## now()

### isoformat()

*datetime.isoformat()* 方法可将 **datetime 对象** 转换为符合 **ISO 8601 标准** 的字符串，非常适合在数据库存储、API 传输、日志记录等场景中使用。

```py
from datetime import datetime
# 获取当前时间并转换为 ISO 格式
now = datetime.now()
print(now.isoformat()) # 默认 'T' 分隔符
print(now.isoformat(sep=' ')) # 自定义分隔符为空格
print(now.isoformat(timespec='seconds')) # 精确到秒
```

- **JSON 序列化** 在 API 返回中嵌入时间戳：

```py
import json
from datetime import datetime
data = {"event": "login", "time": datetime.now().isoformat()}
print(json.dumps(data))
```

### strftime()

strftime() 是 String Format Time 的缩写，它的作用是将一个时间对象（如 datetime）按照你指定的格式转换成字符串

```py
datetime.now().strftime("%Y%m%d_%H%M%S")
```

它的目的是生成一个基于当前时间的、唯一且有序的文件名或目录名

 常用占位符对照表

%Y	四位数的年份	2026

%m	月份 (01-12)	05
%d	日期 (01-31)	09
%H	小时 (00-23, 24小时制)	14
%M	分钟 (00-59)	30
%S	秒 (00-59)	45
