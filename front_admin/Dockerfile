# 使用 Node.js 20 作为基础镜像，这是截至2024年初较新的LTS版本
# Alpine 版本更轻量，适合容器化部署
FROM node:20-alpine as base

# 设置工作目录为 /app
# 后续的命令都将在这个目录下执行
WORKDIR /app

# 依赖安装阶段
# 创建一个名为 deps 的新构建阶段，基于之前的 base 阶段
FROM base as deps
# 复制 package.json 和 yarn.lock 文件到工作目录
# 这两个文件用于确定项目依赖
COPY package.json yarn.lock ./
# 使用 yarn 安装依赖
# --frozen-lockfile 确保安装的依赖版本与 yarn.lock 文件中指定的完全一致
RUN yarn install --frozen-lockfile

# 应用构建阶段
# 创建一个名为 builder 的新构建阶段，同样基于 base 阶段
FROM base as builder
# 从 deps 阶段复制已安装的 node_modules 到当前阶段
COPY --from=deps /app/node_modules ./node_modules
# 复制所有源代码文件到工作目录
COPY . .
# 执行构建命令，生成生产环境可用的文件
RUN yarn build

# 生产环境运行阶段
# 创建最终的生产环境镜像，仍基于 base 阶段
FROM base as runner
# 设置 NODE_ENV 环境变量为 production
# 这通常会启用一些优化，并禁用开发模式特性
ENV NODE_ENV=production

# 从 builder 阶段复制构建产物（.output 目录）到当前阶段
COPY --from=builder /app/.output /app/.output
# 复制 Nuxt 配置文件，这可能在运行时需要
COPY --from=builder /app/nuxt.config.ts ./nuxt.config.ts

# 声明容器将监听的端口
# 注意：这只是一个元数据，实际上并不会导致端口被打开
EXPOSE 3000

# 定义容器启动时执行的命令
# 使用 node 运行 Nuxt 3 生成的服务器文件
CMD ["node", ".output/server/index.mjs"]

# 开发环境阶段
FROM base as dev
ENV NODE_ENV=development

# 复制 package.json 和 yarn.lock
COPY package.json yarn.lock ./

# 安装所有依赖，包括 devDependencies
RUN yarn install

# 复制源代码
COPY . .

# 暴露开发服务器端口
EXPOSE 3000

# 声明卷挂载点
VOLUME ["/app/src"]

# 开发模式启动命令
CMD ["yarn", "dev"]
