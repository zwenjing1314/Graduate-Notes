# 介绍

以不同格式显示日期和时间是程序中最常用到的功能

# 常见API

## isoformat()

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

