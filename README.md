# PowerCMS 12 社区版

**一套平台，两种交付：HTML 页面渲染 + Headless REST / GraphQL API。**

[中文](#中文) | [English](#english)

PowerCMS 12 社区版是基于 **Java 21 + Spring Boot 3.5 + React 18** 的开源双模内容管理系统（Hybrid CMS）：
同一套站点、内容模型、权限、版本与发布数据，既可渲染 HTML 页面（常规 CMS 模式），
也可通过 REST / GraphQL 输出 JSON（Headless 模式）。

本仓库是社区版的**聚合工程**，以 Git 子模块组织三个组件：

```text
powercms/
├── README.md          # 本文件
├── .gitmodules        # 子模块声明
├── cms-common/        # 子模块 · 共享内核
├── cms-core/          # 子模块 · 后端核心
└── admin-ui/          # 子模块 · 管理端
```

---

## 中文

### 子模块

| 子模块 | 仓库 | 说明 |
|---|---|---|
| `cms-common` | [knoxlab/cms-common](https://github.com/knoxlab/cms-common) | 共享内核：统一错误模型（`ApiError` / `ErrorCodes`）、领域事件、调用上下文；不依赖 Spring Web |
| `cms-core` | [knoxlab/cms-core](https://github.com/knoxlab/cms-core) | 后端核心：Spring Boot 3.5 / Spring Modulith 模块化单体，覆盖身份、站点、内容模型、内容与版本、媒体、工作流、构建发布、检索、站点包、审计与交付 |
| `admin-ui` | [knoxlab/admin-ui](https://github.com/knoxlab/admin-ui) | 管理端：React 18 + TypeScript + Vite + Ant Design |

三者均为独立仓库，各自带完整的构建配置、测试与文档（子模块目录内各有 `README.md`）。

### 快速开始

克隆时**必须带上 `--recurse-submodules`**，否则子模块目录为空：

```bash
git clone --recurse-submodules https://github.com/knoxlab/powercms.git
cd powercms
```

已有克隆可补拉子模块：

```bash
git submodule update --init --recursive
```

#### 1. 启动后端（`cms-common` → `cms-core`）

`cms-core` 依赖同级的 `cms-common`，**必须先安装**：

```bash
export JAVA_HOME=/usr/lib/jvm/jdk-21     # 强制 JDK 21；PowerShell: $env:JAVA_HOME='D:\GraalVM\jdk-21.0.7'
cd cms-common && mvn -DskipTests install
cd ../cms-core && mvn spring-boot:run     # http://localhost:8080
```

默认使用 **H2 文件库**（`./data/powercms`，PostgreSQL 兼容模式），**零外部依赖**，首次启动由 Flyway 自动建表。
首次启动自动创建系统管理员：**`admin` / `Admin@123456`**。

> 生产环境务必修改密码，并设置 `powercms.identity.reject-factory-password=true` 拒绝出厂密码启动。

#### 2. 启动管理端（新终端）

```bash
cd admin-ui
npm ci
npm run dev        # http://localhost:5173，/api 与媒体路径代理到 8080
```

生产部署时，若构建 `cms-core` 的同级存在 `admin-ui/dist`，会被整体内嵌进 jar，由后端直接提供管理端静态资源：

```bash
cd admin-ui && npm run build
cd ../cms-core && mvn -DskipTests package
java -jar target/cms-core-12.0.2.jar
```

### 环境要求

| 组件 | 版本 | 说明 |
|---|---|---|
| JDK | **21+（强制）** | `maven-enforcer-plugin` 在 `validate` 阶段强制校验，低版本直接失败并提示 |
| Maven | 3.6.3+ | 需先安装 `cms-common` |
| Node.js | 20+ / npm 10+ | 仅构建 `admin-ui` 时需要 |

> **必须让 `JAVA_HOME` 指向 JDK 21**。若 PATH 上默认 `java` 是 8 / 17，直接运行 `mvn` 会因找不到引导类而失败。
> Windows（PowerShell）：`$env:JAVA_HOME='D:\GraalVM\jdk-21.0.7'`；Linux / macOS：`export JAVA_HOME=/usr/lib/jvm/jdk-21`。

### 数据库

同一套代码与迁移脚本支持 **H2（默认）/ PostgreSQL 14+**，按 profile 自动切换数据源、方言与迁移占位符取值，业务代码零改动：

```bash
POWERCMS_DB_PROFILE=db-postgres mvn spring-boot:run    # PostgreSQL
```

连接参数可用环境变量覆盖（默认值见 `cms-core/src/main/resources/application.yml`）：

```bash
export POWERCMS_DB_HOST=localhost
export POWERCMS_DB_PORT=5432
export POWERCMS_DB_NAME=powercms12
export POWERCMS_DB_USER=postgres
export POWERCMS_DB_PASSWORD=postgres
```

### 验证入口

| 入口 | 地址 |
|---|---|
| 应用元信息 | `GET http://localhost:8080/api/v1/meta` |
| 健康检查 | `GET http://localhost:8080/actuator/health` |
| OpenAPI 文档 | `http://localhost:8080/swagger-ui.html` |
| 交付 REST | `GET http://localhost:8080/api/v1/delivery/...` |
| 交付 GraphQL | `POST http://localhost:8080/graphql/sites/{siteId}` |
| 交付 MVC 渲染 | `GET http://localhost:8080/mvc/sites/{siteKey}/**` |

### 构建与测试

```bash
cd cms-common && mvn install                  # 含测试
cd ../cms-core && mvn clean test              # 后端测试（含 Spring Modulith 架构边界测试）
cd ../admin-ui && npm run build && npm test   # 前端 tsc 校验 + Vite 构建 + Vitest
```

### 核心能力

- **双模一体** —— MVC 页面渲染（含渲染缓存）+ REST / 按站点动态 Schema 的 GraphQL，同源数据两种交付
- **内容治理** —— 不可变版本快照、字段级差异与恢复、编辑锁、定时发布；可版本化工作流（会签、定时流转、站内通知 + 邮件 + Webhook）
- **发布工程** —— Release → Artifact → Deployment 三级不可变发布模型、全量 / 增量 SSG、Nginx 与 Git 发布目标、原子切换与回滚
- **多站点与站点包** —— 多站点多环境、站点蓝图、站点包 / 主题包导入导出与签名校验门禁
- **媒体库** —— 本地文件系统存储、秒传去重、版本管理；视频 / 大文件按 Range 直出（社区版无转码）
- **检索** —— 管理索引与访客索引隔离、候选索引原子切换；默认内存实现，可切换 OpenSearch
- **身份与权限** —— 组织 / 用户 / 角色、站点级授权、OAuth 与 API Token、失败登录锁定
- **审计与运维** —— 追加写审计、系统日志与留存策略、异步任务重试、健康检查

### 社区版与企业版

本仓库**仅包含社区版开源能力**。以下为企业版 / 商业模块，**不在本仓库内**（由独立的专有模块提供）：

- Knox-Media 集成、转码、HLS 播放与 DRM
- 互动运营（评论、问卷、考试、聊天室、广告、留言板）与自定义表单
- AI / RAG 知识库、混合检索、知识图谱、智能体与 MCP 工具
- 外部数据源接入、对象级 ACL 高级特性、离线授权
- 企业集群、SSO 单点登录与高风险操作双人复核

### 开发规约

- 时间统一 UTC 存储、按站点 / 用户时区展示；数据与日志一律 UTF-8。
- 数据库 schema 只通过 Flyway 迁移脚本演进，禁用 Hibernate 自动建表（`ddl-auto: validate`）。
- API 错误统一返回 `code / message / traceId / details` 结构，异常经统一处理器转换。
- 模块间只通过公开服务接口与领域事件通信；新增模块 = 新增子包 + `package-info.java` 声明模块边界，`ModularityTests` 会阻断违规依赖。
- 测试默认走内存库，不得改回文件库，避免污染开发数据。

### 许可证

本项目以 [Apache License 2.0](LICENSE) 开源发布，各子模块内亦附有相同的 `LICENSE` 文件。

---

## English

### Overview

PowerCMS 12 Community Edition is an open-source **dual-mode CMS** built on
**Java 21 + Spring Boot 3.5 + React 18**. The same sites, content models, permissions, versions and
publishing data can either render HTML pages (classic CMS mode) or expose JSON via REST / GraphQL
(Headless mode).

This repository is the **aggregator** for the Community Edition and organizes three components as
Git submodules:

```text
powercms/
├── README.md          # this file
├── .gitmodules        # submodule declarations
├── cms-common/        # submodule · shared kernel
├── cms-core/          # submodule · backend core
└── admin-ui/          # submodule · admin console
```

### Submodules

| Submodule | Repository | Description |
|---|---|---|
| `cms-common` | [knoxlab/cms-common](https://github.com/knoxlab/cms-common) | Shared kernel: unified error model (`ApiError` / `ErrorCodes`), domain events, invocation context; no Spring Web dependency |
| `cms-core` | [knoxlab/cms-core](https://github.com/knoxlab/cms-core) | Backend core: Spring Boot 3.5 / Spring Modulith modular monolith covering identity, sites, content models, content & versions, media, workflow, publishing, search, site packages, audit and delivery |
| `admin-ui` | [knoxlab/admin-ui](https://github.com/knoxlab/admin-ui) | Admin console: React 18 + TypeScript + Vite + Ant Design |

Each is a standalone repository with its own build configuration, tests and documentation.

### Quick start

You **must** clone with `--recurse-submodules`, otherwise the submodule directories will be empty:

```bash
git clone --recurse-submodules https://github.com/knoxlab/powercms.git
cd powercms
```

For an existing clone:

```bash
git submodule update --init --recursive
```

#### 1. Start the backend (`cms-common` → `cms-core`)

`cms-core` depends on the sibling `cms-common`, so it must be installed first:

```bash
export JAVA_HOME=/usr/lib/jvm/jdk-21        # JDK 21 required
cd cms-common && mvn -DskipTests install
cd ../cms-core && mvn spring-boot:run        # http://localhost:8080
```

The default profile uses an **H2 file database** (`./data/powercms`, PostgreSQL compatibility mode) —
no external dependencies; Flyway creates the schema on first boot. A bootstrap admin is created:
**`admin` / `Admin@123456`**. In production change the password and set
`powercms.identity.reject-factory-password=true`.

#### 2. Start the admin console (new terminal)

```bash
cd admin-ui
npm ci
npm run dev        # http://localhost:5173, proxies /api and media paths to 8080
```

For production, if `admin-ui/dist` exists next to `cms-core` at build time it is embedded into the
jar so the app serves the admin UI itself:

```bash
cd admin-ui && npm run build
cd ../cms-core && mvn -DskipTests package
java -jar target/cms-core-12.0.2.jar
```

### Requirements

| Component | Version | Notes |
|---|---|---|
| JDK | **21+ (enforced)** | `maven-enforcer-plugin` fails fast in the `validate` phase on older JDKs |
| Maven | 3.6.3+ | `cms-common` must be installed first |
| Node.js | 20+ / npm 10+ | Only needed to build `admin-ui` |

> Make sure `JAVA_HOME` points to JDK 21. PowerShell: `$env:JAVA_HOME='D:\GraalVM\jdk-21.0.7'`;
> Linux/macOS: `export JAVA_HOME=/usr/lib/jvm/jdk-21`.

### Database

The same code and migrations support **H2 (default) and PostgreSQL 14+**; the active profile
(`POWERCMS_DB_PROFILE`) switches the datasource, Hibernate dialect and migration placeholder values
with zero code changes:

```bash
POWERCMS_DB_PROFILE=db-postgres mvn spring-boot:run
```

Connection parameters can be overridden via `POWERCMS_DB_HOST` / `POWERCMS_DB_PORT` /
`POWERCMS_DB_NAME` / `POWERCMS_DB_USER` / `POWERCMS_DB_PASSWORD`.

### Verify the install

| Endpoint | URL |
|---|---|
| App metadata | `GET http://localhost:8080/api/v1/meta` |
| Health check | `GET http://localhost:8080/actuator/health` |
| OpenAPI docs | `http://localhost:8080/swagger-ui.html` |
| Delivery REST | `GET http://localhost:8080/api/v1/delivery/...` |
| Delivery GraphQL | `POST http://localhost:8080/graphql/sites/{siteId}` |
| Delivery MVC rendering | `GET http://localhost:8080/mvc/sites/{siteKey}/**` |

### Build and test

```bash
cd cms-common && mvn install                  # runs tests
cd ../cms-core && mvn clean test              # backend tests incl. Modulith boundary tests
cd ../admin-ui && npm run build && npm test   # tsc check + Vite build + Vitest
```

### Key features

- **Dual-mode delivery** — MVC page rendering (with render cache) + REST / per-site dynamic-schema GraphQL from one source of truth
- **Content governance** — immutable version snapshots, field-level diffs and restore, edit locks, scheduled publishing; versionable workflows (co-sign, scheduled transitions, in-app + email + Webhook notifications)
- **Publishing pipeline** — Release → Artifact → Deployment immutable model, full/incremental SSG, Nginx and Git publish targets, atomic switch and rollback
- **Multi-site & site packages** — multiple sites and environments, site blueprints, signed site/theme package import/export
- **Media library** — local filesystem storage, instant-upload deduplication, versioning, byte-range streaming (no transcoding in the Community Edition)
- **Search** — isolated admin and public indexes with atomic candidate swap; in-memory backend by default, OpenSearch optional
- **Identity & access** — organizations / users / roles, per-site authorization, OAuth and API tokens, failed-login lockout
- **Audit & operations** — append-only audit trail, system logging with retention policies, async task retry, health checks

### Community vs Enterprise

This repository contains **Community Edition open-source capabilities only**. Enterprise / commercial
modules are provided separately and are intentionally not part of this repository:

- Knox-Media integration, transcoding, HLS playback and DRM
- Interaction features (comments, surveys, exams, chat rooms, ads, message boards) and custom forms
- AI / RAG knowledge bases, hybrid retrieval, knowledge graph, agents and MCP tooling
- External data-source connectors, advanced object-level ACL, offline licensing
- Enterprise clustering, SSO and dual approval for high-risk operations

### License

Licensed under [Apache License 2.0](LICENSE); an identical `LICENSE` file ships inside each submodule.
