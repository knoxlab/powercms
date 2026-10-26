PowerCMS 12 社区版首个正式发布：基于 **Java 21 + Spring Boot 3.5 + React 18** 的开源双模（Hybrid）CMS——同一套站点、内容模型、权限、版本与发布数据，既可渲染 HTML 页面（常规 CMS），也可通过 REST / GraphQL 输出 JSON（Headless）。

First public release of PowerCMS 12 Community Edition — an open-source hybrid CMS built on **Java 21 + Spring Boot 3.5 + React 18**: one set of sites, content models, permissions, versions and publishing data serving both HTML rendering (classic CMS) and JSON via REST / GraphQL (headless).

[中文](#中文) | [English](#english)

---

## 中文

### 主要能力

- **双模一体** —— MVC 页面渲染（含渲染缓存）+ REST / 按站点动态 Schema 的 GraphQL，同源数据两种交付
- **内容治理** —— 不可变版本快照、字段级差异与恢复、编辑锁、定时发布；可版本化工作流（会签、定时流转、站内通知 + 邮件 + Webhook）
- **发布工程** —— Release → Artifact → Deployment 三级不可变发布模型、全量 / 增量 SSG、Nginx 与 Git 发布目标、原子切换与回滚
- **多站点与站点包** —— 多站点多环境、站点蓝图、站点包 / 主题包导入导出与签名校验门禁
- **媒体库** —— 本地文件系统存储、秒传去重、版本管理；视频 / 大文件按 Range 直出（社区版无转码）
- **检索** —— 管理索引与访客索引隔离、候选索引原子切换；默认内存实现，可切换 OpenSearch
- **身份与权限** —— 组织 / 用户 / 角色、站点级授权、OAuth 与 API Token、失败登录锁定
- **审计与运维** —— 追加写审计、系统日志与留存策略、异步任务重试、健康检查

### 本版本修复

- **建站 500**（`36f0d45`）：修复新建站点时 `entitlementGate` Bean 缺失导致的 SpEL 求值失败，以及站点删除时的自引用外键失败。
- **数字媒体预览 500**（`a2cb027`）：挂载的媒体在预览时带 `Range` 请求（`<video>` 播放必发）返回 500。`/media-preview`、`/media-file`、`/media-thumb` 统一改为返回 `ResponseEntity<Resource>`，分片由 Spring MVC 输出 206；`/media-file` 一并修正；中文文件名按 RFC 6266 / RFC 5987 编码，避免 Tomcat 丢弃整个 `Content-Disposition`。新增 `MediaRangePreviewTest` 回归用例（分片字节与源文件逐字节一致）。

### 交付物

| 文件 | 说明 |
|---|---|
| `cms-core-12.0.2.jar` | Spring Boot 可执行 jar（约 93.5 MB），内嵌管理端静态资源 |
| `cms-core-12.0.2.jar.sha256` | SHA-256 校验文件 |

```text
4071587d5c6e73f81f6b8b7251f234f1e0a35c6160b5945697305a1b83f6f8ef  cms-core-12.0.2.jar
```

### 运行

```bash
java -jar cms-core-12.0.2.jar     # 需 JDK 21+；默认 H2 文件库（./data/powercms），零外部依赖
```

首次启动由 Flyway 自动建表并创建管理员：**`admin` / `Admin@123456`**。生产环境务必修改密码，并设置 `powercms.identity.reject-factory-password=true` 拒绝出厂密码启动。

切换 PostgreSQL 14+：`POWERCMS_DB_PROFILE=db-postgres POWERCMS_DB_HOST=... java -jar cms-core-12.0.2.jar`

从源码构建（必须带子模块）：

```bash
git clone --recurse-submodules https://github.com/knoxlab/powercms.git
cd cms-common && mvn -DskipTests install     # cms-core 依赖同级 cms-common
cd ../cms-core && mvn -DskipTests package
```

> 克隆时若未加 `--recurse-submodules`，可执行 `git submodule update --init --recursive` 补拉。

---

## English

### Key features

- **Dual-mode delivery** — MVC page rendering (with render cache) + REST / per-site dynamic-schema GraphQL from one source of truth
- **Content governance** — immutable version snapshots, field-level diffs and restore, edit locks, scheduled publishing; versionable workflows (co-sign, scheduled transitions, in-app + email + Webhook notifications)
- **Publishing pipeline** — Release → Artifact → Deployment immutable model, full/incremental SSG, Nginx and Git publish targets, atomic switch and rollback
- **Multi-site & site packages** — multiple sites and environments, site blueprints, signed site/theme package import/export
- **Media library** — local filesystem storage, instant-upload deduplication, versioning, byte-range streaming (no transcoding in the Community Edition)
- **Search** — isolated admin and public indexes with atomic candidate swap; in-memory backend by default, OpenSearch optional
- **Identity & access** — organizations / users / roles, per-site authorization, OAuth and API tokens, failed-login lockout
- **Audit & operations** — append-only audit trail, system logging with retention policies, async task retry, health checks

### Fixes in this release

- **Site creation 500** (`36f0d45`): fixed SpEL evaluation failure caused by a missing `entitlementGate` bean when creating a site, and the self-referencing foreign-key failure when deleting one.
- **Digital media preview 500** (`a2cb027`): mounted media preview returned 500 whenever the request carried a `Range` header (which `<video>` playback always sends). `/media-preview`, `/media-file` and `/media-thumb` now uniformly return `ResponseEntity<Resource>` and let Spring MVC emit 206 partial content; `/media-file` was fixed alongside; Chinese filenames are encoded per RFC 6266 / RFC 5987 so Tomcat no longer drops the whole `Content-Disposition`. Added the `MediaRangePreviewTest` regression suite (range bytes verified byte-for-byte against the source file).

### Artifacts

| File | Description |
|---|---|
| `cms-core-12.0.2.jar` | Spring Boot executable jar (~93.5 MB), admin console assets embedded |
| `cms-core-12.0.2.jar.sha256` | SHA-256 checksum file |

```text
4071587d5c6e73f81f6b8b7251f234f1e0a35c6160b5945697305a1b83f6f8ef  cms-core-12.0.2.jar
```

### Getting started

```bash
java -jar cms-core-12.0.2.jar     # requires JDK 21+; H2 file DB (./data/powercms) by default, zero external deps
```

Flyway creates the schema on first boot and creates the bootstrap admin: **`admin` / `Admin@123456`**.
In production change the password and set `powercms.identity.reject-factory-password=true`.

PostgreSQL 14+: `POWERCMS_DB_PROFILE=db-postgres POWERCMS_DB_HOST=... java -jar cms-core-12.0.2.jar`

Build from source (submodules required):

```bash
git clone --recurse-submodules https://github.com/knoxlab/powercms.git
cd cms-common && mvn -DskipTests install     # cms-core depends on sibling cms-common
cd ../cms-core && mvn -DskipTests package
```

> If you cloned without `--recurse-submodules`, run `git submodule update --init --recursive`.

### Verify the install

| Endpoint | URL |
|---|---|
| App metadata | `GET http://localhost:8080/api/v1/meta` |
| Health check | `GET http://localhost:8080/actuator/health` |
| OpenAPI docs | `http://localhost:8080/swagger-ui.html` |

### Submodule revisions

| Submodule | Revision |
|---|---|
| `cms-common` | `d3f01bb` |
| `cms-core` | `a2cb027` |
| `admin-ui` | `0ac6383` |

### License

Licensed under [Apache License 2.0](https://github.com/knoxlab/powercms/blob/main/LICENSE).
