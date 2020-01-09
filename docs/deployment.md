# Saiblo

## 介绍

Saiblo 网站是一个通用 AI 游戏平台，支持用户编写 AI 参加比赛或是进行人机对局，支持游戏开发者上传按照开发规范编写的游戏，支持比赛举办者举办关于某个游戏的比赛。

## 部署方式


### Docker (不推荐)

使用 Dockerfile 自动化部署

```bash
docker build -t saiblo .
docker run -d -p 127.0.0.1:3000:80 --name saiblo saiblo
docker start saiblo
```

网站将会在 `127.0.0.1:3000` 上启动，你可以修改为 `0.0.0.0:80` 监听来自所有地址的链接。这种方法不推荐

### 本地部署

需要提前安装好 `Node.js, python3, redis`，其中 `redis` 需要运行在 `6379` 端口（可以在 `backend/app/settings.py` 的 `CACHES`  处进行配置）。

#### 安装依赖

```bash
cd frontend
npm install
cd ../backend
pip install -r requirements.txt
```

#### 初始化数据库

```bash
cd backend
python manage.py migrate
python manage.py loadtestdata # 如果有需要，可以载入这套测试数据，包含两个游戏以及若干小组
```

#### 启动前端

```bash
cd frontend
npm run build # 编译前端
export NUXT_HOST=0.0.0.0 # 指定监听地址
export NUXT_PORT=80 # 指定端口
npm run start # 启动前端
```

#### 启动后端

```bash
cd backend
uvicorn app.asgi:application --workers 1 --port 8000 --host 127.0.0.1 # 以 1 个 worker 启动后端，如有需要可以增加 worker 数量，但需要换用 Mysql 等并发友好数据库，否则会在访问过快时死锁
```

#### 可持久化运行

推荐使用 `supervisor` 配置服务，也可使用 `tmux` 来可持久化运行前后端

#### 测评端连接

测评段需要连接到后端监听的地址，如 `127.0.0.1:8000`，由于 `127.0.0.1:8000/judger/` 下的 API 不需要鉴权，所以不要将后端地址暴露出来，前端会有 