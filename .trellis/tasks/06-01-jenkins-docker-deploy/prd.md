# 配置 Jenkins Docker 部署文件

## Goal

为当前 new-api 项目补齐 Jenkins + Docker 的部署配置，让 Jenkins 可以构建项目镜像、推送到公司镜像仓库，并通过 SSH 到部署服务器执行 Docker Compose 更新。

## What I Already Know

* 项目根目录已有 `Dockerfile`，会构建两个前端目录并编译 Go 后端。
* 项目根目录已有 `docker-compose.yml`，默认运行 `new-api + redis + postgres`。
* 当前部署目标是 Jenkins 平台通过 Docker 部署。
* Git 远端已配置为推送到 GitHub fork 和公司 GitLab：`https://gitlab.gigimed.cn/gigi/new-api-gigi`。

## Assumptions

* 真实镜像仓库地址、部署服务器地址、Jenkins 凭据 ID 会由部署人员按公司环境替换。
* 不把生产密码、数据库密码、Redis 密码、SSH 私钥等敏感信息提交进仓库。
* 默认采用 `latest` + Jenkins `BUILD_NUMBER` 双标签策略。

## Requirements

* 新增 Jenkins Pipeline 文件，用于 checkout、构建 Docker 镜像、登录镜像仓库、推送镜像、SSH 远程部署、健康检查。
* 新增生产 Docker Compose 模板，用于服务器端拉取 Jenkins 推送的镜像并启动 `new-api + redis + postgres`。
* 新增生产环境变量示例文件，指导部署人员创建真实 `.env`。
* 新增 Jenkins 部署说明，说明 Jenkins 凭据、变量替换、首次部署、后续发布、回滚方式。

## Acceptance Criteria

* [ ] 仓库根目录存在可用于 Jenkins Pipeline 的 `Jenkinsfile`。
* [ ] 仓库存在生产 Compose 模板，避免使用默认 `123456` 生产密码。
* [ ] 仓库存在环境变量示例文件，不包含真实密钥。
* [ ] 文档说明如何替换公司环境参数并执行部署。

## Definition of Done

* 配置文件语法清晰、可维护。
* 不引入业务代码变更。
* 不提交真实敏感配置。
* 本地完成基础静态检查。

## Out of Scope

* 不直接连接公司 Jenkins 创建 Job。
* 不创建真实 GitLab/Harbor/Docker Registry 凭据。
* 不在部署服务器上执行真实部署。
* 不改造应用代码和 Dockerfile 构建逻辑。

## Technical Notes

* `Dockerfile` 暴露端口 `3000`，运行入口为 `/new-api`。
* Compose 健康检查可继续使用 `/api/status`。
* 生产 `.env` 已被 `.gitignore` 忽略，示例文件应命名为 `.env.production.example`。
