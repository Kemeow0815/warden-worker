# Warden: 运行在 Cloudflare Workers 上的 Bitwarden 兼容服务器

[![Powered by Cloudflare](https://img.shields.io/badge/Powered%20by-Cloudflare-F38020?logo=cloudflare&logoColor=white)](https://www.cloudflare.com/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Deploy to Cloudflare Workers](https://img.shields.io/badge/Deploy%20to-Cloudflare%20Workers-orange?logo=cloudflare&logoColor=white)](https://workers.cloudflare.com/)

本项目提供了一个可以部署到 Cloudflare Workers 的自托管 Bitwarden 兼容服务器，完全免费。它的设计理念是"部署后无需维护"，让你无需担心服务器管理或持续的费用支出。

## 为什么要做这个项目？

虽然像 [Vaultwarden](https://github.com/dani-garcia/vaultwarden) 这样的项目提供了优秀的自托管解决方案，但它们仍然需要你管理服务器或 VPS。这可能会很麻烦，而且如果你忘记续费服务器，可能会失去对密码的访问权限。

Warden 通过利用 Cloudflare Workers 生态系统来解决这个问题。通过将 Warden 部署到 Cloudflare Worker 并使用 Cloudflare D1 进行数据存储，你可以拥有一个完全免费、无服务器、低维护的 Bitwarden 服务器。

## 功能特性

* **核心密码库功能：** 创建、读取、更新和删除密码条目（Ciphers）和文件夹。
* **文件附件：** 可选的 Cloudflare KV 或 R2 存储用于附件。
* **Bitwarden Send：** 通过链接分享加密文本或文件。
* **设备管理：** 查看和撤销活跃会话。
* **实时同步与推送通知：** 通过 WebSocket 和移动端推送实现实时密码库更新。
* **TOTP 支持：** 存储和生成基于时间的一次性密码。
* **Bitwarden 兼容：** 可与官方 Bitwarden 客户端配合使用。
* **免费托管：** 运行在 Cloudflare 的免费套餐上。
* **低维护：** 部署一次，无需再管。
* **安全可靠：** 你的加密数据存储在你自己的 Cloudflare D1 数据库中。
* **易于部署：** 使用 Wrangler CLI 几分钟内即可运行。

### 附件支持

Warden 支持使用 **Cloudflare KV** 或 **Cloudflare R2** 作为存储后端：

| 特性 | KV | R2 |
|---------|----|----|  
| 最大文件大小 | **25 MB**（硬性限制） | 100 MB（受 Workers 请求体大小限制） |
| 需要信用卡 | **否** | 是 |
| 流式 I/O | 支持 | 支持 |

**后端选择：** R2 优先 — 如果配置了 R2，将使用 R2。否则使用 KV。

更多设置细节请参阅[部署指南](docs/deployment.md)。R2 可能会产生额外费用；请参阅 [Cloudflare R2 定价](https://developers.cloudflare.com/r2/pricing/)。

### Bitwarden Send

- **文本 Send：** 默认启用，无需额外配置。
- **文件 Send：** 需要存储后端（KV 或 R2），与[附件](#附件支持)配置相同。

> [!NOTE]
> 由于 D1 单行大小限制为 2 MB，文本 Send 的最大大小约为 **1.8 MiB**。此外，`/api/sync` 端点会将当前用户的所有 Send 序列化到响应中。大量的 Send 或非常大的文本 Send 会显著增加 CPU 时间和响应大小。

## 当前状态

**本项目尚未功能完整**，~~也可能永远不会完整~~。目前支持个人密码库的核心功能，包括 TOTP。但是，**不支持**以下功能：

* 密码共享
* 2FA 登录（除 TOTP 外）
* 紧急访问
* 管理员操作
* 组织/团队功能
* 其他 Bitwarden 高级功能

目前没有计划实现这些功能。本项目的主要目标是提供一个简单、免费、低维护的个人密码管理器。

## 兼容性

* **浏览器扩展：** Chrome、Firefox、Safari 等（已在 Chrome 上测试 2026.3.0 版本）
* **Android 应用：** 官方 Bitwarden Android 应用（已测试 2026.4.0 版本）
* **iOS 应用：** 官方 Bitwarden iOS 应用（已测试 2026.4.0 版本）

## 在线演示

演示实例可在 [warden.qqnt.de](https://warden.qqnt.de) 访问。

你可以使用以 `@warden-worker.demo` 结尾的邮箱注册新账户（该邮箱无需验证）。

如果你决定停止使用演示实例，请删除你的账户以便为其他用户腾出空间。

强烈建议部署你自己的实例，因为演示实例可能会触发速率限制或被 Cloudflare 禁用。

## 快速开始

- 选择部署方式：[命令行部署](#命令行部署) 或 [GitHub Actions CI/CD 部署](#cicd-部署使用-github-actions)。
- 按照部署文档设置密钥和可选的附件功能。
- 配置 Bitwarden 客户端指向你的 Worker URL。

## 前端（Web Vault）

前端通过 [Cloudflare Workers Static Assets](https://developers.cloudflare.com/workers/static-assets/) 与 Worker 捆绑部署。GitHub Actions 工作流会下载**固定版本**的 [bw_web_builds](https://github.com/dani-garcia/bw_web_builds)（Vaultwarden 网页版，默认：`v2026.3.1`）并随后端一起部署。你可以通过 GitHub Actions 变量覆盖版本（生产环境使用 `BW_WEB_VERSION`，开发环境使用 `BW_WEB_VERSION_DEV`），或设置为 `latest` 以跟随上游最新版本。

**工作原理：**
- 静态文件（HTML、CSS、JS）由 Cloudflare 的边缘网络直接提供。
- API 请求（`/api/*`、`/identity/*`）路由到 Rust Worker。
- 无需单独的 Pages 部署或域名配置。

**UI 覆盖（可选）：**
- 本项目在 `public/css/` 中提供了一组轻量级的"自托管"UI 调整。
- 在 CI/CD（以及可选的本地环境）中，我们在解压 `bw_web_builds` 后应用它们：
  - `mkdir -p public/web-vault/css/ && cp public/css/vaultwarden.css public/web-vault/css/`

> [!NOTE]
> 从单独的前端部署迁移？如果你之前将前端单独部署到 Cloudflare Pages，可以删除 `warden-frontend` Pages 项目并重新设置 Worker 的路由。前端现在与 Worker 捆绑，不再需要单独部署。

> [!WARNING]
> 网页版前端来自 Vaultwarden，因此暴露了许多高级 UI 功能，但大多数都无法使用。请参阅[当前状态](#当前状态)。

## 配置自定义域名（可选）

默认的 `*.workers.dev` 域名默认被禁用，因为它可能会抛出 1101 错误。你可以通过在 `wrangler.toml` 中设置 `workers_dev = true` 来启用它。

如果你想使用自定义域名而不是默认的 `*.workers.dev` 域名，请按照以下步骤操作：

### 第一步：添加 DNS 记录

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 选择你的域名（例如 `example.com`）
3. 进入 **DNS** → **记录**
4. 点击 **添加记录**：
   - **类型：** `A`（或 `AAAA` 用于 IPv6）
   - **名称：** 你的子域名（例如 `vault` 对应 `vault.example.com`）
   - **IPv4 地址：** `192.0.2.1`（这是占位符，实际路由由 Worker 处理）
   - **代理状态：** **已代理**（橙色云图标 — 这是必需的！）
   - **TTL：** 自动
5. 点击 **保存**

> [!IMPORTANT]
> **代理状态必须是"已代理"**（橙色云）。如果显示"仅 DNS"（灰色云），Worker 路由将无法工作。

### 第二步：添加 Worker 路由

1. 进入 **Workers & Pages** → 选择你的 `warden-worker`
2. 点击 **设置** → **域名和路由**
3. 点击 **添加** → **路由**
4. 配置路由：
   - **路由：** `vault.example.com/*`（替换为你的域名）
   - **区域：** 选择你的域名区域
   - **Worker：** `warden-worker`
5. 点击 **添加路由**

## 内置速率限制

本项目包含由 [Cloudflare 速率限制 API](https://developers.cloudflare.com/workers/runtime-apis/bindings/rate-limit/) 驱动的速率限制。敏感端点受到保护：

| 端点 | 速率限制 | 键类型 | 用途 |
|----------|------------|----------|---------|
| `/identity/connect/token` | 5 请求/分钟 | 邮箱地址 | 防止密码暴力破解 |
| `/api/accounts/register` | 5 请求/分钟 | IP 地址 | 防止批量注册和邮箱枚举 |
| `/api/accounts/prelogin` | 5 请求/分钟 | IP 地址 | 防止邮箱枚举 |

你可以在 `wrangler.toml` 中调整速率限制设置：

```toml
[[ratelimits]]
name = "LOGIN_RATE_LIMITER"
namespace_id = "1001"
# 调整 limit（请求数）和 period（10 或 60 秒）
simple = { limit = 5, period = 60 }
```

> [!NOTE]
> `period` 必须是 `10` 或 `60` 秒。详情请参阅 [Cloudflare 文档](https://developers.cloudflare.com/workers/runtime-apis/bindings/rate-limit/)。

如果绑定缺失，请求将继续而不进行速率限制（优雅降级）。

## 配置

### CPU 卸载（通过 Durable Objects）

Cloudflare Workers 免费套餐的每个请求 CPU 预算非常小。有两种端点特别消耗 CPU：

- 导入端点：大型 JSON 负载（通常 500kB–1MB）+ 解析 + 批量插入。
- 注册、登录和密码验证端点：服务器端 PBKDF2 密码验证。

为了保持主 Worker 快速运行，同时仍然支持这些操作，Warden 可以将**选定的端点卸载到 Durable Objects (DO)**：

- **Heavy DO (`HEAVY_DO`)**：用 Rust 实现为 `HeavyDo`（重用现有的 axum 路由器），因此 CPU 密集型端点可以在更高的 CPU 预算下运行。

**如何启用/禁用**

是否卸载 CPU 密集型端点取决于 `wrangler.toml` 中是否配置了 `HEAVY_DO` Durable Object 绑定。

> [!NOTE]
> Durable Objects 在免费套餐中每个请求有高达 30 秒的 CPU 预算（请参阅 [Cloudflare Durable Objects 限制](https://developers.cloudflare.com/durable-objects/platform/limits/)），因此我们可以用它来处理 CPU 密集型端点。
>
> Durable Objects 可能产生两种计费：计算和存储。本项目不使用存储，免费套餐每天允许 100,000 个请求和 13,000 GB-秒持续时间，这对大多数用户来说应该足够了。详情请参阅 [Cloudflare Durable Objects 定价](https://developers.cloudflare.com/durable-objects/platform/pricing/)。
>
> 如果你选择禁用 Durable Objects，你可能需要订阅付费套餐以避免被 Cloudflare 限制。

### 实时同步和推送通知

Warden 通过两种机制支持密码库数据的实时同步：WebSocket 推送（用于桌面应用和浏览器扩展）和移动端推送通知（用于官方移动应用）。

**WebSocket 推送（桌面和扩展）**

此功能由 Durable Objects 驱动，当 `wrangler.toml` 中配置了 `NOTIFY_DO` Durable Object 绑定时默认启用。移除此绑定（和迁移）将优雅地禁用 WebSocket 通知。

**移动端推送通知**

Warden 通过 Bitwarden 推送中继服务支持向官方 Bitwarden 移动应用发送推送通知。

**设置：**

1. 从 [https://bitwarden.com/host/](https://bitwarden.com/host/) 获取安装 ID 和密钥。
2. 通过 Cloudflare 仪表板或 `wrangler` CLI 将凭证存储为密钥（`PUSH_INSTALLATION_ID` 和 `PUSH_INSTALLATION_KEY`）。
3. 通过在 `wrangler.toml` 的 `[vars]` 中设置 `PUSH_ENABLED` 为 `true` 或在 Cloudflare 仪表板中启用推送。

可选地，你可以通过设置 `PUSH_RELAY_URI` 和 `PUSH_IDENTITY_URI` 覆盖默认的中继端点（默认为 `https://push.bitwarden.com` 和 `https://identity.bitwarden.com`）。

有关详细配置和故障排除，请参阅 [Vaultwarden 关于推送通知的 wiki](https://github.com/dani-garcia/vaultwarden/wiki/Enabling-Mobile-Client-push-notification)。

### 其他环境变量

在 `wrangler.toml` 的 `[vars]` 下配置环境变量，或通过 Cloudflare 仪表板设置：

* **`BASE_URL`**（可选）：
  - 覆盖文件上传/下载 URL 的提取基础 URL。
  - 格式：包含 HTTPS 协议、域名和端口（如果使用非 443 反向代理）。不要包含任何尾部路径。
  - 示例：`https://vault.example.com` 或 `https://vault.example.com:8443`
  - 如果未设置，则从传入请求中提取。
* **`PASSWORD_ITERATIONS`**（可选，默认：`600000`）：
  - 服务器端密码哈希的 PBKDF2 迭代次数。
  - 最小值为 600000。
* **`TRASH_AUTO_DELETE_DAYS`**（可选，默认：`30`）：
  - 软删除项目在清除前保留的天数。
  - 设置为 `0` 或负数以禁用。
* **`IMPORT_BATCH_SIZE`**（可选，默认：`30`）：
  - 导入/删除操作的批处理大小。
  - `0` 禁用批处理。
* **`DISABLE_USER_REGISTRATION`**（可选，默认：`true`）：
  - 控制是否在客户端 UI 中显示注册按钮（服务器行为不变）。
* **`AUTHENTICATOR_DISABLE_TIME_DRIFT`**（可选，默认：`false`）：
  - 设置为 `true` 以禁用 TOTP 验证的 ±1 时间步漂移。
* **`ATTACHMENT_MAX_BYTES`**（可选）：
  - 单个附件文件的最大大小。
  - 示例：`104857600` 表示 100MB。
* **`ATTACHMENT_TOTAL_LIMIT_KB`**（可选）：
  - 每个用户的附件存储总限制（KB）。
  - 示例：`1048576` 表示 1GB。
* **`ATTACHMENT_TTL_SECS`**（可选，默认：`300`，最小：`60`）：
  - 附件上传/下载 URL 的 TTL。
* **`SEND_TEXT_MAX_BYTES`**（可选，默认：`1887436` ≈ 1.8 MiB）：
  - 文本 Send 内容的最大大小。受 D1 的 2 MB 单行限制约束。
* **`SEND_MAX_BYTES`**（可选，默认：`104857600` = 100 MiB）：
  - 文件 Send 的最大文件大小。受与附件相同的 KV/R2 限制约束。
* **`USER_SEND_LIMIT_KB`**（可选）：
  - 每个用户的 Send 文件存储总限制（KB）。
* **`SEND_TTL_SECS`**（可选，默认：`300`）：
  - Send 文件上传/下载 URL 的 TTL。

### 定时任务（Cron）

Worker 运行定时任务来清理软删除的项目。默认情况下，它每天在 UTC 时间 03:00 运行（`wrangler.toml` `[triggers]` cron `"0 3 * * *"`）。根据需要调整；有关 cron 表达式语法，请参阅 [Cloudflare Cron Triggers 文档](https://developers.cloudflare.com/workers/configuration/cron-triggers/)。

## 数据库操作

- **备份和恢复：** 请参阅[数据库备份与恢复](docs/db-backup-recovery.md#github-actions-备份)了解自动备份和手动恢复步骤。
- **时间旅行：** 请参阅 [D1 时间旅行](docs/db-backup-recovery.md#d1-时间旅行-时间点恢复)以恢复到某个时间点。
- **种子全局等效域名（可选）：** 请参阅 [docs/deployment.md](docs/deployment.md) 了解在 CLI 部署和 CI/CD 中的种子设置。
- **使用 D1 进行本地开发：**
  - 快速开始：`wrangler dev --persist`
  - 完整堆栈（带网页版）：按照部署文档下载前端资源，然后 `wrangler dev --persist`
  - 本地导入备份：`wrangler d1 execute vault1 --file=backup.sql`
  - 检查本地数据库：SQLite 文件位于 `.wrangler/state/v3/d1/`

## 本地开发

使用 Wrangler 在本地运行带有 D1 支持的 Worker。

**快速开始（仅 API）：**

```bash
wrangler dev --persist
```

**完整堆栈（带 Web Vault）：**

1. 下载前端资源（请参阅[部署文档](docs/deployment.md#下载前端-web-vault)）。
2. 本地启动：

   ```bash
   wrangler dev --persist
   ```

3. 在 `http://localhost:8787` 访问密码库。

**临时使用生产数据：**

1. 下载并解密备份（请参阅[备份文档](docs/db-backup-recovery.md#恢复数据库到-cloudflare-d1)）。
2. 不带 `--remote` 本地导入：

   ```bash
   wrangler d1 execute vault1 --file=backup.sql
   ```

3. 启动 `wrangler dev --persist` 并将客户端指向 `http://localhost:8787`。

**检查本地 SQLite：**

```bash
ls .wrangler/state/v3/d1/
sqlite3 .wrangler/state/v3/d1/miniflare-D1DatabaseObject/*.sqlite
```

> [!NOTE]
> 本地开发需要 Node.js 和 Wrangler。Worker 通过 [workerd](https://github.com/cloudflare/workerd) 在模拟环境中运行。

## 更新你的 Fork

如果你通过 GitHub Fork 部署，保持更新很简单：

1. **关注新发布** — 在[本仓库](https://github.com/qaz741wsd856/warden-worker)上点击 **Watch** → **Custom** → 勾选 **Releases**。新版本发布时你会收到通知。
2. **同步你的 Fork** — 进入你在 GitHub 上的 Fork，点击 **Sync fork** → **Update branch**。这将把上游的最新更改拉取到你 Fork 的默认分支。
3. **自动部署** — 如果你通过 GitHub Actions 设置了 CI/CD，推送到 main 分支的工作流将自动构建并部署新版本到你的 Cloudflare Worker。无需手动操作。

> [!TIP]
> 建议在上游发布新版本时同步你的 Fork，这样你总是拥有最新的功能和安全修复。

## 贡献

欢迎提交 Issue 和 PR。提交前请运行 `cargo fmt` 和 `cargo clippy --target wasm32-unknown-unknown --no-deps`。

## 许可证

本项目采用 MIT 许可证。详情请参阅 `LICENSE` 文件。

---

# 自部署指南

本文档介绍两种部署方式，选择适合你工作流程和基础设施的方式。

## 目录

- [命令行部署](#命令行部署)
- [CI/CD 部署使用 GitHub Actions](#cicd-部署使用-github-actions)

---

## 命令行部署

### 前置要求

- [Node.js](https://nodejs.org/)  installed
- [Wrangler CLI](https://developers.cloudflare.com/workers/wrangler/install-and-update/) installed
- Cloudflare 账户

### 部署步骤

#### 1. 克隆仓库

```bash
git clone https://github.com/your-username/warden-worker.git
cd warden-worker
```

#### 2. 创建 D1 数据库

```bash
wrangler d1 create warden-db
```

创建成功后，会输出 `database_id`，请记录下来。

#### 3. （可选）启用 R2 存储桶用于附件

Warden 默认使用 KV 进行附件存储。如果你想使用 R2 作为存储后端：

```bash
# 创建生产存储桶
wrangler r2 bucket create warden-attachments
```

然后在 `wrangler.toml` 中取消注释 R2 存储桶配置部分以启用 R2 绑定。

**注意：** 附件是可选的。如果你移除 KV 和 R2 绑定，附件功能将被禁用，但其他所有功能仍将正常工作。

#### 4. 配置数据库 ID

创建 D1 数据库时，Wrangler 会输出 `database_id`。为避免将此密钥提交到仓库，本项目使用环境变量来配置数据库 ID。

你有两个选项：

**选项 1：（推荐）使用 `.env` 文件：**

在项目根目录创建名为 `.env` 的文件，并添加以下行，将占位符替换为你的实际 `database_id`：

```
D1_DATABASE_ID="your-database-id-goes-here"
```

确保将 `.env` 文件添加到你的 `.gitignore` 文件中，以防止它被提交到 git。

**选项 2：在 shell 中设置环境变量：**

你可以在部署前在 shell 中设置环境变量：

```bash
export D1_DATABASE_ID="your-database-id-goes-here"
wrangler deploy
```

#### 5. 下载前端（Web Vault）

```bash
# 默认固定版本（通过导出 BW_WEB_VERSION 覆盖）
BW_WEB_VERSION="${BW_WEB_VERSION:-v2026.3.1}"
if [ "${BW_WEB_VERSION}" = "latest" ]; then
  BW_WEB_VERSION="$(curl -s https://api.github.com/repos/dani-garcia/bw_web_builds/releases/latest | jq -r .tag_name)"
fi

# 下载并解压
wget "https://github.com/dani-garcia/bw_web_builds/releases/download/${BW_WEB_VERSION}/bw_web_${BW_WEB_VERSION}.tar.gz"
tar -xzf "bw_web_${BW_WEB_VERSION}.tar.gz" -C public/
rm "bw_web_${BW_WEB_VERSION}.tar.gz"

# 删除大型 source map 以满足 Cloudflare 静态资源单文件限制
find public/web-vault -type f -name '*.map' -delete
```

**可选：** 应用轻量级 UI 覆盖以生成 `public/web-vault/css/vaultwarden.css`：

```bash
mkdir -p public/web-vault/css/ && cp public/css/vaultwarden.css public/web-vault/css/
```

#### 6. 设置数据库并部署 Worker

```bash
# 仅在首次部署前运行一次
wrangler d1 execute vault1 --file sql/schema.sql --remote
# 用于迁移
wrangler d1 migrations apply vault1 --remote

# （可选）将全局等效域名种子导入 D1
# 默认下载 Vaultwarden 的 global_domains.json
bash scripts/seed-global-domains.sh --db vault1 --remote

wrangler deploy
```

这将部署 Worker 并设置必要的数据库表。

#### 7. 设置环境变量为 Secret

- `ALLOWED_EMAILS` your-email@example.com（支持 glob 模式如 `*@example.com`）
- `JWT_SECRET` 一个长随机字符串
- `JWT_REFRESH_SECRET` 一个长随机字符串

**可选移动端推送中继设置：**
`PUSH_ENABLED=true`、`PUSH_RELAY_URI`、`PUSH_IDENTITY_URI` 作为文本变量；
`PUSH_INSTALLATION_ID`、`PUSH_INSTALLATION_KEY` 作为密钥变量。
详情请参阅[移动端推送通知](#移动端推送通知可选)。

#### 8. 配置你的 Bitwarden 客户端

在你的 Bitwarden 客户端中，进入自托管登录屏幕并输入你部署的 Worker URL。

默认情况下，`*.workers.dev` 域名被禁用，因为它可能会抛出 1101 错误。强烈建议使用自定义域名；详情请参阅[配置自定义域名](#配置自定义域名可选)。

---

## CI/CD 部署使用 GitHub Actions

本项目包含 GitHub Actions 工作流用于自动部署。这是生产环境的推荐方法，因为它确保一致的构建和部署。

### 所需 Secrets

将以下 secrets 添加到你的 GitHub 仓库（`Settings > Secrets and variables > Actions`）：

| Secret | 必需 | 描述 |
|--------|----------|-------------|
| `CLOUDFLARE_API_TOKEN` | 是 | 你的 Cloudflare API 令牌 |
| `CLOUDFLARE_ACCOUNT_ID` | 是 | 你的 Cloudflare 账户 ID |
| `D1_DATABASE_ID` | 是 | 你的生产 D1 数据库 ID |
| `D1_DATABASE_ID_DEV` | 否 | 开发 D1 数据库 ID（仅当你使用 `dev` 分支上的 `Deploy Dev` 工作流时需要） |

#### 如何获取 Cloudflare 账户 ID

1. 登录 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 选择你的账户
3. 你的账户 ID 显示在概览页面的右侧边栏，或在 URL 中：`https://dash.cloudflare.com/<account-id>`

#### 如何获取 Cloudflare API Token

`CLOUDFLARE_API_TOKEN` 需要以下权限：
- **Edit Cloudflare Workers**：部署 Worker 必需
- **Edit D1**：数据库迁移和备份必需
- **Edit KV**：附件存储必需（如果使用 KV）

1. 访问 [https://dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens)
2. 点击 **Create Token**
3. 使用 **Edit Cloudflare Workers** 模板
4. 在 `Permissions` 下添加 **Account** → **D1**
5. 选择 `Account Resources` 和 `Zone Resources`
6. 点击 **Continue to Summary** 然后 **Create Token**

### 可选变量

#### Web Vault 前端版本

你可以通过 GitHub Actions 变量固定/覆盖捆绑的 Web Vault（bw_web_builds）版本：

| 变量 | 适用于 | 默认 | 示例 | 说明 |
|----------|------------|---------|---------|-------|
| `BW_WEB_VERSION` | 生产（`main/uat/release*`） | `v2026.3.1` | `v2026.3.1` | 设置为 `latest` 以跟随上游最新发布 |
| `BW_WEB_VERSION_DEV` | 开发（`dev`） | `v2026.3.1` | `v2026.3.1` | 设置为 `latest` 以跟随上游最新发布 |

#### 全局等效域名

Bitwarden 客户端使用 `globalEquivalentDomains` 进行知名域名组的 URI 匹配。

为避免将大型 JSON 文件捆绑到 Worker 中，数据集可以存储在 D1 中并在部署期间种子化。

| 变量 | 适用于 | 默认 | 示例 | 说明 |
|----------|------------|---------|---------|-------|
| `SEED_GLOBAL_DOMAINS` | 生产 + 开发 | `true` | `false` | 设置为 `false` 以跳过种子化（API 返回空列表） |
| `GLOBAL_DOMAINS_URL` | 生产 | （空） | raw GitHub URL | 可选：固定特定的 Vaultwarden 标签/提交以实现可复现部署 |
| `GLOBAL_DOMAINS_URL_DEV` | 开发 | （空） | raw GitHub URL | 与生产相同，但用于开发工作流 |

如果你跳过种子化，`/api/settings/domains` 和 `/api/sync` 将返回 `globalEquivalentDomains: []`。

### 使用步骤

#### 1. Fork 或克隆仓库到你的 GitHub 账户

#### 2. 在仓库设置中配置所需的 secrets

#### 3. （可选）启用 R2 存储桶用于附件：

Warden 默认使用 KV 进行附件存储。如果你想使用 R2 作为存储后端：

1. **在运行 Action 之前在 Cloudflare Dashboard 中创建 R2 存储桶：**
   - 进入 **Storage & databases** → **R2** → **Create bucket**
   - 创建生产存储桶（例如 `warden-attachments`）

2. **将存储桶名称添加为 GitHub Action secrets：**
   - `R2_NAME` → 生产存储桶名称

当这些 secrets 存在时，工作流将自动将 `ATTACHMENTS_BUCKET` 绑定追加到 `wrangler.toml` — 无需在 Cloudflare 控制台中手动绑定。

#### 4. 从仓库的 GitHub Actions 标签页手动触发 `Build` Action

#### 5. 在仓库的 Actions 标签页监控部署

#### 6. 在 Cloudflare 仪表板中将环境变量设置为 `secret`（按照命令行部署步骤）：
- `ALLOWED_EMAILS` your-email@example.com（支持 glob 模式如 `*@example.com`，逗号分隔）
- `JWT_SECRET` 一个长随机字符串
- `JWT_REFRESH_SECRET` 一个长随机字符串
- 可选移动端推送设置：
  `PUSH_ENABLED=true`、`PUSH_RELAY_URI`、`PUSH_IDENTITY_URI`、`PUSH_INSTALLATION_ID`、`PUSH_INSTALLATION_KEY`。
  详情请参阅[移动端推送通知](#移动端推送通知可选)。

> [!IMPORTANT]
> 服务器没有这三个环境变量无法工作。如果你忘记设置它们，服务器将崩溃。

如果你想在前端显示"创建账户"按钮，可以添加 `DISABLE_USER_REGISTRATION` 作为 `text` 并将其设置为 `false`。更多详情请查看[环境变量](#环境变量)。

默认情况下，`*.workers.dev` 域名被禁用，因为它可能会抛出 1101 错误。强烈建议使用自定义域名；详情请参阅[配置自定义域名](#配置自定义域名可选)。

---

## 常见问题

### Q: 免费套餐有什么限制？

Cloudflare Workers 免费套餐包括：
- 每天 100,000 个请求
- 每个请求最多 50ms CPU 时间（使用 Durable Objects 可增加到 30 秒）
- D1 数据库：每天 500 万次读取，10 万次写入

对于个人密码管理器使用，这些限制通常足够了。

### Q: 我的数据安全吗？

是的。Warden 使用与 Bitwarden 相同的加密方式：
- 所有数据在发送到服务器之前都在客户端加密
- 服务器只存储加密后的数据
- 你的主密码永远不会发送到服务器

### Q: 如何备份我的数据？

请参阅 [docs/db-backup-recovery.md](docs/db-backup-recovery.md) 了解自动备份和手动恢复步骤。

### Q: 我可以从 Vaultwarden 迁移吗？

可以。你可以从 Vaultwarden 导出加密备份，然后导入到 Warden。请注意 Warden 不支持 Vaultwarden 的所有功能，请参阅[当前状态](#当前状态)。
