# Jenkins Docker 部署说明

本文档说明如何通过 Jenkins 构建 new-api Docker 镜像，并在 Jenkins 所在机器上通过 Docker Compose 部署。

## 1. 部署架构

```text
GitLab/GitHub -> Jenkins -> Docker Registry -> Jenkins 本机 Docker Compose
```

Jenkins 负责：

- 拉取代码
- 执行 `docker build`
- 推送 `new-api-gigi-${BUILD_NUMBER}` 和 `latest` 两个镜像标签
- 在 Jenkins 本机执行 `docker compose pull` 和 `docker compose up -d`

Jenkins 机器负责：

- 保存生产 `.env`
- 保存 `/data/new-api-gigi/docker-compose.yml`
- 运行 `new-api`、`redis`、`postgres`

## 2. Jenkins 凭据

在 Jenkins 中添加以下凭据：

| Credentials ID | 类型 | 用途 |
| --- | --- | --- |
| `docker-registry-creds` | Username with password | 登录镜像仓库 |
| `git-creds` | Username/password 或 SSH key | 拉取私有代码仓库 |

如果你要使用不同的 Credentials ID，请同步修改仓库根目录的 `Jenkinsfile`。

## 3. Jenkinsfile 参数

当前按公司 Nexus 配置：

```groovy
REGISTRY = '192.168.16.102:8082'
IMAGE_REPO = 'gigi-docker/new-api-gigi'
IMAGE_TAG = "new-api-gigi-${BUILD_NUMBER}"
DEPLOY_DIR = '/data/new-api-gigi'
COMPOSE_SOURCE = 'deploy/docker-compose.prod.yml'
COMPOSE_FILE = 'docker-compose.yml'
```

构建并推送的镜像：

```text
192.168.16.102:8082/gigi-docker/new-api-gigi:new-api-gigi-${BUILD_NUMBER}
192.168.16.102:8082/gigi-docker/new-api-gigi:latest
```

`deploy/docker-compose.prod.yml` 默认部署：

```yaml
image: 192.168.16.102:8082/gigi-docker/new-api-gigi:latest
```

## 4. 初始化 Jenkins 本机部署配置

当前 `Jenkinsfile` 会把仓库里的 `deploy/docker-compose.prod.yml` 覆盖拷贝到 `/data/new-api-gigi/docker-compose.yml`，然后在 `/data/new-api-gigi` 目录执行 Docker Compose。

首次部署前，在 Jenkins 机器上准备固定部署目录和真实环境变量文件：

```bash
mkdir -p /data/new-api-gigi
cp deploy/.env.production.example /data/new-api-gigi/.env
```

编辑 `/data/new-api-gigi/.env`，至少替换：

```text
POSTGRES_PASSWORD
REDIS_PASSWORD
SQL_DSN
REDIS_CONN_STRING
SESSION_SECRET
```

注意：

- `POSTGRES_PASSWORD` 必须和 `SQL_DSN` 中的 PostgreSQL 密码一致。
- `REDIS_PASSWORD` 必须和 `REDIS_CONN_STRING` 中的 Redis 密码一致。
- `.env` 包含生产密钥，不要提交到 Git。
- Jenkins 每次构建会覆盖 `/data/new-api-gigi/docker-compose.yml`，但不会覆盖 `/data/new-api-gigi/.env`。

## 5. 本地校验 Compose 模板

在仓库本地校验 Compose 模板时，可以使用示例环境变量文件：

```bash
set -a
. deploy/.env.production.example
set +a
NEW_API_ENV_FILE=.env.production.example docker compose -f deploy/docker-compose.prod.yml config
```

## 6. Jenkins 创建 Pipeline

Jenkins 页面创建任务：

```text
New Item -> Pipeline
```

推荐配置：

```text
Definition: Pipeline script from SCM
SCM: Git
Repository URL: https://gitlab.gigimed.cn/gigi/new-api-gigi
Credentials: git-creds
Branch: main-gigi
Script Path: Jenkinsfile
```

保存后点击 `Build Now`。

## 7. 部署验证

Jenkins 部署完成后，在 Jenkins 机器检查：

```bash
cd /data/new-api-gigi
docker compose -f docker-compose.yml ps
docker compose -f docker-compose.yml logs -f new-api
```

健康检查：

```bash
curl -fsS http://127.0.0.1:3000/api/status
```

浏览器访问：

```text
http://Jenkins机器IP:3000
```

## 8. 回滚

如果需要回滚到某次 Jenkins 构建镜像，将 `docker-compose.prod.yml` 中的镜像标签从 `latest` 改为指定构建号：

```yaml
image: 192.168.16.102:8082/gigi-docker/new-api-gigi:new-api-gigi-123
```

然后执行：

```bash
cd /data/new-api-gigi
docker compose -f docker-compose.yml up -d
```
