# TransWork 演示环境 Docker 部署说明

## 1. 交付文件

将整个 `deploy` 目录复制到服务器。目录中必须包含：

```text
演示部署/
├── docker-compose.yaml
├── .env
└── 后台网站Docker部署操作手册.md
```

`.env` 已包含演示环境默认配置，必须与 `docker-compose.yaml` 放在同一目录。

## 2. 启动

进入 `演示部署` 目录执行：

```bash
cd 演示部署
docker-compose up -d
```

首次启动会自动拉取三个镜像并启动：

- Web：`3000`
- 文档解析服务：`5002`
- PostgreSQL：仅 Compose 内部访问

Web 容器会只读挂载宿主机的 `/etc/machine-id` 和
`/sys/class/dmi/id/product_uuid`，据此生成管理后台“组织管理 → 授权配置”中的
“系统识别码”。授权码必须使用这串识别码签发；如果识别码为空，请先确认宿主机
是 Linux 且上述两个文件存在，然后重建 Web 容器：

```bash
docker-compose up -d --force-recreate transwork-web
```

可用以下命令确认容器内已读到标识文件：

```bash
docker-compose exec transwork-web sh -c 'test -s /run/path2ai-host-machine-id && echo machine-id-ok'
docker-compose exec transwork-web sh -c 'test -s /run/path2ai-host-product-uuid && echo product-uuid-ok'
```

Web 容器名和 Compose 服务名均为 `transwork-web`。

查看状态：

```bash
docker-compose ps
```

## 3. 访问地址

- 网站首页：`http://localhost:3000/`
- 管理后台：`http://localhost:3000/admin`
- 网站健康检查：`http://localhost:3000/healthz`
- 文档解析服务：`http://localhost:5002`

远程访问时，将 `localhost` 替换为服务器 IP。需要修改端口或公网地址时，编辑 `.env` 后重新执行：

```bash
docker-compose up -d
```

## 4. 默认账号

后台管理员：

```text
账号：admin
密码：TransWork_Admin_2026
```

数据库账号：

```text
数据库：transwork
用户：transwork
密码：TransWork_DB_2026_Demo
```

以上密码仅适用于演示环境。

## 5. pgvector

Web 启动时会自动检查并注册 PostgreSQL `vector` 扩展，成功后才启动 Web 服务。

查看日志：

```bash
docker-compose logs -f transwork-web
docker-compose logs -f postgres
docker-compose logs -f fileext
```

如果出现 `pgvector is not available`，请确认数据库镜像包含 pgvector 扩展。

## 6. 常用命令

```bash
# 停止服务，保留数据
docker-compose stop

# 启动已创建的服务
docker-compose start

# 重启 Web
docker-compose restart transwork-web

# 拉取新镜像并更新
docker-compose pull
docker-compose up -d
```

不要执行以下命令，除非确认要删除数据库数据：

```bash
docker-compose down -v
```

## 7. 数据持久化

- `uploads/`：文档解析上传文件
- `static/`：文档解析静态文件
- Docker 卷 `transwork-data`：Web 运行数据
- Docker 卷 `transwork-postgres-data`：PostgreSQL 数据

备份数据库：

```bash
docker-compose exec -T postgres sh -c 'pg_dump -U "$POSTGRES_USER" "$POSTGRES_DB"' > transwork-backup.sql
```
