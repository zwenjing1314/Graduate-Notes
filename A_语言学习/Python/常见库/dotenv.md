# 介绍

**dotenv** 是一个用于管理环境变量的 Python 库，通常用于加载和处理敏感信息的配置，例如数据库连接信息、API 密钥和密码等。通过使用 dotenv，可以将这些配置存储在 *.env* 文件中，并在代码中动态加载，避免将敏感信息直接写入代码，提高安全性和灵活性。

# 安装

```bash
pip install python-dotenv
```

# 使用示例

在项目根目录下创建一个 *.env* 文件，存储配置变量，例如：

```
DB_HOST=localhost
DB_PORT=5432
DB_USERNAME=admin
DB_PASSWORD=secret
```

在 Python 代码中加载这些变量：

```py
from dotenv import load_dotenv
import os
# 加载 .env 文件中的配置变量
load_dotenv()
# 获取环境变量
db_host = os.getenv("DB_HOST")
db_port = os.getenv("DB_PORT")
db_username = os.getenv("DB_USERNAME")
db_password = os.getenv("DB_PASSWORD")
# 示例：连接到数据库
print(f"Connecting to database at {db_host}:{db_port} with user {db_username}")
```

优势

1. **安全性**：敏感信息存储在 *.env* 文件中，避免直接暴露在代码中。
2. **灵活性**：支持在不同环境（开发、测试、生产）中使用不同的配置，只需修改 *.env* 文件。
3. **易维护**：集中管理配置变量，减少代码修改的风险。

通过 *dotenv*，开发者可以更高效地管理配置变量，同时保护敏感信息，适用于各种需要环境变量的场景。