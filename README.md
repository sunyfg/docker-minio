# MinIO 本地部署

基于 [AIStor](https://min.io/aistor) 镜像的 MinIO 单节点本地部署，通过 `docker-compose.yml` 编排，数据持久化到宿主机。

## 目录结构

```
.
├── docker-compose.yml   # 容器编排定义
├── .env.example          # 环境变量模板
├── .env                  # 实际环境变量（不入库，需手动创建）
├── .gitignore
├── minio.license         # AIStor 商业许可证（机密，不入库）
├── data/                 # 对象存储持久化数据（不入库）
└── README.md
```

## 端口映射

| 宿主机端口 | 容器端口 | 用途                        |
| ---------- | -------- | --------------------------- |
| 9000       | 9000     | MinIO S3 API                |
| 9101       | 9101     | MinIO 管理控制台（Console） |

两个端口均绑定到 `0.0.0.0`，可局域网通过本机 IP 访问。

## 快速开始

### 1. 准备许可证

将有效的 AIStor 许可证文件放置到项目根目录：

```
minio.license
```

该文件在容器内以只读方式挂载到 `/minio.license`。无此文件 AIStor 商业特性无法启用。

### 2. 创建环境变量文件

```bash
cp .env.example .env
```

按需修改 `.env` 中的凭据，建议用强密码：

```bash
openssl rand -hex 24   # 生成随机密码
```

| 变量                  | 说明                                |
| --------------------- | ----------------------------------- |
| `MINIO_ROOT_USER`     | MinIO 根用户（管理员账号），≥3 字符 |
| `MINIO_ROOT_PASSWORD` | 根用户密码，生产环境建议强密码      |

### 3. 启动服务

```bash
docker compose up -d
```

### 4. 访问

- **管理控制台**：<http://localhost:9101>（或 `http://<本机IP>:9101`）
- **S3 API**：<http://localhost:9100>（SDK / `mc` / `aws s3` 配置 endpoint）

使用 `.env` 中的 `MINIO_ROOT_USER` / `MINIO_ROOT_PASSWORD` 登录。

### 5. 验证（可选）

通过官方客户端 `mc` 快速验证：

```bash
mc alias set local http://localhost:9100 <MINIO_ROOT_USER> <MINIO_ROOT_PASSWORD>
mc ls local
```

## 常用命令

```bash
# 启动
docker compose up -d

# 查看日志
docker compose logs -f minio

# 停止
docker compose stop

# 停止并删除容器（保留数据）
docker compose down

# 停止并删除容器与数据卷（data/ 目录需手动清理）
docker compose down -v
```

## 配置说明

`docker-compose.yml` 关键项：

- **镜像**：`quay.io/minio/aistor/minio:latest`（AIStor 版本，包含商业增强功能）
- **重启策略**：`unless-stopped`
- **数据卷**：`./data` → `/mnt/data`（对象存储持久化）
- **许可证**：`./minio.license` → `/minio.license`（只读挂载）
- **启动参数**：`minio server /mnt/data --console-address ":9101" --license /minio.license`

## 安全提示

- `.env` 与 `minio.license` 已通过 `.gitignore` 忽略，切勿入库。
- 生产环境应使用强随机密码，并考虑在前端部署反代 / TLS 终结（本配置仅用于本地）。
- 端口绑定 `0.0.0.0`，如仅本机访问应改为 `127.0.0.1:9100:9000`。

## 文件入库

已入库：`docker-compose.yml`、`.env.example`、`.gitignore`、`README.md`
不入库：`.env`、`minio.license`、`data/`、`.DS_Store`、`.omo/`
