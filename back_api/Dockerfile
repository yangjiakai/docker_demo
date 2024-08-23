# 使用官方 Python 3.9 镜像作为基础镜像
FROM python:3.9 as base

# 设置工作目录为 /app （最佳实践）
WORKDIR /app

# 将 requirements.txt 文件从宿主机复制到容器的当前工作目录（/app）
COPY requirements.txt .

# 使用 pip 安装 requirements.txt 中列出的所有依赖
# --no-cache-dir 选项防止 pip 缓存下载的包，有助于减小镜像大小
RUN pip install --no-cache-dir -r requirements.txt

# 生产环境阶段
FROM base as production

# 将当前目录（包含 Dockerfile 的目录）中的所有文件和文件夹复制到容器的工作目录
COPY . .

# 设置容器启动时执行的命令
# 使用 uvicorn 运行 FastAPI 应用
# main:app 指定 uvicorn 从 main.py 文件中导入名为 app 的对象
# --host "0.0.0.0" 允许从容器外部访问应用
# --port "8000" 指定应用监听的端口
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]

# 开发环境阶段
FROM base as development

# 安装额外的开发依赖
RUN pip install --no-cache-dir uvicorn[standard] watchfiles

# 将当前目录（包含 Dockerfile 的目录）中的所有文件和文件夹复制到容器的工作目录
COPY . .

# 设置环境变量
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# 暴露端口
EXPOSE 8000

# 声明卷挂载点，用于热重载
VOLUME ["/app"]

# 设置开发模式的启动命令
# 使用 uvicorn 运行 FastAPI 应用，启用热重载
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000", "--reload"]
