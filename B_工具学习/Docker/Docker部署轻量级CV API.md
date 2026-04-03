为了保持“轻量”，我们先不引入庞大的深度学习框架，而是用纯 Python 模拟一个接收图像并返回目标检测结果的 API。

## 准备项目文件

在你的电脑上新建一个文件夹，比如叫 `cv-docker-api`，然后在里面创建以下三个文件：

**文件 1：`requirements.txt`** (定义 Python 依赖)

```
fastapi==0.104.1
uvicorn==0.24.0
python-multipart==0.0.6
```

**文件 2：`app.py`** (核心业务逻辑) 这里我们用 FastAPI 写一个极简的 Web 服务，它接收一张上传的图片，并模拟返回一些检测信息。

```py
from fastapi import FastAPI, File, UploadFile
import uvicorn

app = FastAPI(title="轻量级图像检测 API")

@app.post("/predict")
async def predict_image(file: UploadFile = File(...)):
    # 读取图像内容（这里只是为了演示，实际应用中你会在这里加入 YOLO 或 CLIP 的推理代码）
    content = await file.read()
    
    # 模拟返回检测结果，比如在遥感图像中检测到了特定目标
    return {
        "filename": file.filename,
        "content_type": file.content_type,
        "file_size_bytes": len(content),
        "simulated_detections": [
            {"class": "building", "confidence": 0.92, "bbox": [10, 20, 50, 60]},
            {"class": "vehicle", "confidence": 0.85, "bbox": [100, 150, 120, 180]}
        ],
        "message": "图像接收成功！如果这是真实的容器，这里将返回模型的推理结果。"
    }

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

**文件 3：`Dockerfile`** (Docker 的构建图纸) 这是学习的重点。它告诉 Docker 如何从零开始组装你的运行环境。

```
# 1. 指定基础镜像：使用官方的 Python 轻量级镜像
FROM python:3.9-slim

# 2. 设置容器内的工作目录
WORKDIR /app

# 3. 将本地的 requirements.txt 复制到容器中
COPY requirements.txt .

# 4. 在容器内安装依赖（使用清华源加速下载）
RUN pip install --no-cache-dir -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

# 5. 将本地的代码文件复制到容器中
COPY app.py .

# 6. 声明容器运行时监听的端口
EXPOSE 8000

# 7. 容器启动时执行的命令
CMD ["python", "app.py"]
```

## 构建与运行 Docker 镜像

打开终端（命令行），进入你刚才创建的 `cv-docker-api` 目录，执行以下两步：

**第一步：构建镜像 (Build)**

```bash
docker build -t my-cv-api:v1 .
```

*这行命令的意思是：根据当前目录（`.`）下的 Dockerfile，构建一个名为 `my-cv-api`，版本标签为 `v1` 的镜像。你会看到 Docker 在一步步下载 Python 环境并安装依赖。*

**第二步：运行容器 (Run)**

```bash
docker run -d -p 8000:8000 --name my_api_container my-cv-api:v1
```

## 测试你的服务

服务启动后，打开浏览器，访问这个地址： 👉 **http://localhost:8000/docs**

这是 FastAPI 自动生成的交互式文档。你可以直接在网页上点击 `POST /predict` -> `Try it out`，然后上传一张随便什么格式的本地图片，点击 `Execute`。

你就能在网页下方看到服务器返回的 JSON 结果了！在这个过程中，你的电脑并没有直接运行 Python 脚本，一切都在那个隔离的 Docker 容器中完美运行。

这个轻量级的流程跑通后，未来你想部署更复杂的视觉模型，只需要把 `Dockerfile` 里的基础镜像换成带 PyTorch 的版本，并在 `app.py` 里加载你的权重文件就可以了。

## 目录挂载

## Gemini said

目录挂载（Bind Mounts）是 Docker 开发中最核心的技巧之一。

在正常的流程中，如果你修改了 Python 代码，你必须重新执行 `docker build` 重新打包镜像，这在调试代码时非常低效。

通过**目录挂载**，你可以将电脑上的某个文件夹“直接映射”到容器内部。这样，你在电脑上用 VS Code 修改代码，容器里的程序会立刻看到变化，无需重启或重新构建。

### 1. 修改运行命令

我们不需要修改之前的 `Dockerfile`，只需要在执行 `docker run` 时增加一个 `-v` 参数。

假设你还在 `cv-docker-api` 文件夹下，执行以下命令：

```bash
# 如果之前的容器还在运行，先停止并删除它
docker stop my_api_container
docker rm my_api_container

# 使用挂载方式重新启动
docker run -d \
  -p 8000:8000 \
  --name my_api_container \
  -v $(pwd):/app \
  my-cv-api:v1
```

**参数详解：**

- **`-v $(pwd):/app`**: 这是关键。
  - `$(pwd)`：代表“当前路径”（你的电脑文件夹）。
  - `:/app`：代表容器内部的路径（我们在 Dockerfile 里定义的 `WORKDIR`）。
  - 这句话的意思是：**把当前目录的内容，实时同步到容器的 /app 目录。**

### 2. 配合 FastAPI 的热重载 (Hot Reload)

为了让代码修改后服务自动重启，我们需要在代码里开启 `reload` 模式。

请修改你本地的 **`app.py`** 最后两行：

```py
if __name__ == "__main__":
    # 增加 reload=True，这样检测到文件变化时 Uvicorn 会自动重启
    uvicorn.run("app:app", host="0.0.0.0", port=8000, reload=True)
```

### 3. 体验效果

1. 运行上面的 `docker run -v ...` 命令。
2. 在你的电脑上修改 `app.py` 中的返回文字（比如把 `"message": "图像接收成功！"` 改成 `"message": "代码已实时更新！"`）。
3. **保存文件。**
4. 刷新浏览器 `http://localhost:8000/docs` 并重新测试 API。你会发现结果立刻变了，而你并没有重新构建镜像！

------

为了让你更直观地理解这种“跨时空”的同步机制，你可以通过下面的交互式演示查看主机与容器之间的数据流动：