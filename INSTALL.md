# Product Studio 安装与模型配置

适用于当前正式版 v0.19.5。Product Studio 的发布包是 Node.js 服务，浏览器访问工作台；它不是桌面 EXE/MSI。ZIP 包含已构建的网页、服务端和运行依赖，不含 Node.js、用户数据和模型密钥。

## 准备与校验

1. 安装 Node.js 22.13 或更高版本，运行 `node --version` 确认版本。使用 ZIP 不需要 `npm install`。
2. 从 [最新正式发布](https://github.com/RocLing26/product-studio-releases/releases/latest) 下载 `product-studio-0.19.5-intranet.zip` 和 `product-studio-0.19.5-intranet.zip.sha256`，放在同一目录。
3. 在该目录校验文件，再解压 ZIP：

```bash
# macOS
shasum -a 256 -c product-studio-0.19.5-intranet.zip.sha256

# Linux（二选一，执行这一行即可）
sha256sum -c product-studio-0.19.5-intranet.zip.sha256
```

Windows PowerShell 可运行 `Get-FileHash .\product-studio-0.19.5-intranet.zip -Algorithm SHA256`，将输出的哈希与 `.sha256` 文件第一列比较。校验失败时重新下载。

## 本机启动

解压后进入 `product-studio-0.19.5-intranet` 目录，运行：

```bash
node server/index.mjs
```

浏览器打开 `http://127.0.0.1:4310`。服务默认仅监听本机地址；按 `Ctrl+C` 停止。首次打开为演示模式，可先体验示例流程；真实模型功能按下文配置。

## 数据保存与升级

未设置 `PM_DATA_DIR` 时，数据保存在程序目录下的 `.data/`。长期使用建议让数据目录与程序目录分开，例如在 macOS/Linux 启动前：

```bash
export PM_DATA_DIR="$HOME/product-studio-data"
node server/index.mjs
```

Windows PowerShell 可在启动前设置 `$env:PM_DATA_DIR = 'C:\ProductStudio\data'`。数据目录中包含工作记录和模型配置（包括 API Key），请限制访问权限并备份整个目录。

升级时先停止服务，备份完整数据目录及内网部署使用的 `.env`；将新版 ZIP 解压到新的程序目录，用原来的 `PM_DATA_DIR` 和 `.env` 启动。不要覆盖正在运行的目录，也不要用旧版程序打开新版数据库。工作台可以检查、下载并校验正式更新，但不会自动替换程序。

### 0.19.5 的备份与恢复命令

在解压后的程序目录中运行以下命令；将占位路径替换为实际路径，备份输出放在数据目录之外：

```bash
node scripts/backup-data.mjs <数据目录> <备份输出目录> .env
node scripts/backup-data.mjs --verify <生成的备份目录>
node scripts/restore-data.mjs <生成的备份目录> <全新的恢复数据目录>
```

未使用.env的本机部署，备份命令省略最后的.env参数。备份包含SQLite在线副本、数据目录内的模型/搜索/运行/提示词配置和指定的.env，导入原件随数据库保存。恢复目标已经存在或校验不通过时拒绝恢复，不覆盖现有数据。恢复目录内的.env需按部署方式配置到程序根目录并更新PM_DATA_DIR；持续同步的外部资料目录另行保留。回退时同时使用升级前程序和对应数据备份。

### PRD、原型与设计的复核

PRD验收编号缺少定义或存在歧义时，修正正文并保存新版本，再复核和确认。原型自动检查需要管理员配置Python Playwright与Chromium；未启用时逐项记录人工检查，缺少检查不显示通过。原型、PRD或场景版本变化后重新验收，设计映射缺项按提示补齐。编号和结构检查不能替代业务规则核对。

## 内网 HTTPS 部署

当前是单一管理账号的个人工作台，同一人可以从多台设备访问同一服务；没有团队成员权限和 SSO。内网部署需要已获准运行 Node.js 的服务器、内网域名、TLS 证书，以及同机 HTTPS 反向代理。

在解压后的程序目录运行：

```bash
node scripts/configure-intranet.mjs https://pm.company.example
```

将示例域名替换为实际访问地址。此命令生成 `.env`，其中包含随机 `PM_ACCESS_TOKEN`；不会覆盖现有 `.env`。在文件中设置独立的绝对路径，例如 `PM_DATA_DIR=/var/lib/product-studio`，保持 `PM_BIND_HOST=127.0.0.1`。然后运行：

```bash
node --env-file=.env server/index.mjs
```

由管理员将 HTTPS 域名反向代理到 `http://127.0.0.1:4310`，保留原始 Host 和 Authorization 请求头；ZIP 内有 `deploy/nginx.conf.example`。浏览器访问该 HTTPS 地址，使用账号 `pm` 和 `.env` 中的 `PM_ACCESS_TOKEN` 登录。请由管理员保管 `.env` 和访问口令。

## 模型配置

真实生成需要一个兼容 OpenAI API 的生成模型服务。服务器需要能够连接该服务；模型请求会发送到你配置的地址。未配置时保持演示模式，不调用外部模型。

1. 在工作台打开「工作台配置 → 模型与搜索」，点击「添加 Provider」。
2. 填写「Provider 名称」、`API Base URL`、`API Key`。Base URL 使用服务商给出的 API 根地址，通常以 `/v1` 结尾，不要填写完整的 `/chat/completions` 地址。远程模型地址必须是 HTTPS；本机模型服务可使用 localhost HTTP。
3. 点击「添加模型」，填写准确的「模型 ID」。根据服务商限制设置输出预算等参数，点击「保存 Provider」。
4. 在「当前生成配置」选择刚添加的模型，点击「测试生成模型连接」。成功后再试用灵感探索、PRD 或原型生成。

如需配置知识检索使用的 Embedding，在 Provider 中填写「Embedding 模型 ID」；它可以与生成模型在同一 Provider，也可以使用独立 Provider。然后在「知识冲突检索服务」选择它并点击「测试 Embedding 连接」。Embedding 可选：未配置时使用本地向量召回，但知识冲突的最终判断仍需生成模型。外部产品调研还需单独配置搜索服务。

管理员也可以在 `.env` 中指定生成模型，并用 `node --env-file=.env server/index.mjs` 启动：

```dotenv
PM_MODEL_MODE=compatible
PM_MODEL_BASE_URL=https://model.example.com/v1
PM_MODEL_NAME=your-model-id
PM_MODEL_API_KEY=your-private-api-key
```

`PM_MODEL_*` 环境变量会覆盖界面中的当前生成配置；移除后重启即可改由界面选择。示例值需替换为实际服务信息，不要提交含密钥的 `.env`。如果连接测试失败，检查服务器到模型地址的网络、API Key、模型 ID，以及服务商要求的输出参数和流式支持。

## 可视化并行设置

打开 **工作台配置 → 并行执行**，所有项目共用以下上限：

| 界面名称 | 配置字段 | 范围 / 默认 |
| --- | --- | --- |
| 需求生成任务 | `maxConcurrentJobs` | 1–50 / 10 |
| 知识提炼分段 | `maxConcurrentKnowledgeChunks` | 1–8 / 3 |
| 灵感与调研 | `maxConcurrentResearchRequests` | 1–6 / 3 |
| 知识图谱提炼 | `maxConcurrentGraphExtractions` | 1–6 / 3 |

修改后点击“保存并行设置”，设为 1 可相应串行处理。Agent 运行记录页只显示当前需求上限并提供跳转入口，原编辑表单已合并。只保存修改项，旧客户端仅修改需求并行数时保留其它上限；刷新保留尚未提交的修改，保存失败保留输入。设置会保存在数据目录 runtime.json，备份时一并保留。

已开始请求继续，后续任务、工具轮次或图谱批次使用新设置。这些值分别控制工作类型，不是全部 Provider 请求的统一并发上限。文档提炼整份完成后才写入知识；图谱关系仍需人工审核；调研保留已完成请求断点、原定来源顺序和最多 6 个正文来源。证据与前后生成依赖继续校验。

## 健康检查

本机启动后打开 `http://127.0.0.1:4310/api/health`，应返回服务健康状态。内网访问出现 403 时，检查浏览器访问地址、`.env` 中的 `PM_PUBLIC_ORIGIN` 与反向代理保留的 Host 是否一致。

## 跨源冲突知识纠错

打开知识页的跨源冲突详情，在知识侧点击“编辑知识”，修改完整标题和正文后点击“保存并重新校验”。保存检查知识与来源版本，仍检出疑似冲突时自动模型复核；模型不可用会提示知识已保存及复核错误，可以再次复核。自动校验未再检出不代表模型已确认无冲突，其它关联来源继续后台校验。

0.19.5 升级到 schema v38，增加扫描规则版本，历史扫描按新规则逐步重做；人工忽略/处理决定和已有模型复核记录保留。复杂改写仍需模型与人工核对。升级前备份完整数据目录及 .env，回退同时恢复升级前数据库，禁止让旧程序打开已迁移的库。

## 图谱批量提炼与运行配置

在“知识图谱 → 批量提炼已有知识”搜索并多选/全选已生效知识，每批最多1000条。任务在后台执行，关闭面板后继续，可查看每条分段进度并重试失败项；重试/重启复用有效分段，知识正文/版本或模型配置变化时重提。自动、单条及批量共用“并行执行”的图谱模型请求上限，范围1–6，默认3。提炼结果仍需原文核对和审核。

“运行与备份”支持浏览器验收启停/Python、PDF启停/Docling程序/模型目录/可选Python与备份输出目录。填写服务运行设备上的路径，展开安装步骤后按已验证版本准备依赖，保存到数据目录operations.json并立即用于后续操作。运行中的PDF继续使用开始时的配置，后续解析使用新配置。

点击“立即备份”在线备份数据库、模型/搜索/运行/提示词/组件配置及可用.env，并校验SHA-256和数据库完整性/外键/版本；“校验最近备份”重新核验副本。部署包默认备份到版本目录之外的backups目录，可在页面修改。外部同步资料目录需单独备份。schema v38保存图谱断点；升级和回退须一起备份、恢复对应数据及配置。
