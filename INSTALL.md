# Product Studio · 安装与更新

此 ZIP 是服务端部署包，适用于 Node.js 22.13+。浏览器访问工作台，无需安装客户端。ZIP 不包含 Node.js、用户数据或访问口令。

## 首次安装

1. 核对 ZIP 同名 `.sha256` 文件中的 SHA-256。
2. 解压到独立程序目录，在目录中运行 `node scripts/configure-intranet.mjs https://你的内网域名`，生成 `.env`。配置文件中包含随机访问口令，请仅由管理员保管。
3. 在 `.env` 设置独立的数据目录 `PM_DATA_DIR`，使用 `node --env-file=.env server/index.mjs` 启动。
4. 通过公司批准的 HTTPS 反向代理访问程序。Node 服务默认仅绑定回环地址；如使用 Docker，请参考 `deploy/compose.yaml`。

工作台默认从公开的 `RocLing26/product-studio-releases` 仓库匿名检查和下载正式包。每台独立运行的电脑都不需要 GitHub token。下载仅保存新 ZIP，不会自动替换程序。

## 模型配置

首次打开为演示模式，不需要模型密钥。要使用真实模型生成、灵感探索和知识冲突判定，打开「工作台配置 → 模型与搜索」：

1. 点击「添加 Provider」，填写名称、兼容 OpenAI API 的 `API Base URL` 和 `API Key`。Base URL 通常以 `/v1` 结尾，不要填写完整的 `/chat/completions` 地址。远程模型地址须使用 HTTPS，本机模型可使用 localhost HTTP。
2. 点击「添加模型」，填写服务提供的准确模型 ID，根据服务限制设置输出预算等参数，然后保存 Provider。
3. 在「当前生成配置」选择该模型，点击「测试生成模型连接」。

如需检查模型是否实际返回工具调用与 JSON Schema 结果，可点击「探测当前模型能力」。它会向当前 Provider 各发送一次短请求；“试跑通过”只代表这次返回符合检查结构，切换模型或服务版本后需重新试跑。需求探索与调研简报会优先尝试原生函数调用；Provider 拒绝该参数时，同一轮回退到文本控制动作路径。JSON Schema 的探测结果不会自动改变对话路径。

Embedding 用于知识冲突候选召回。可在同一个或独立的 Provider 中填写「Embedding 模型 ID」，在「知识冲突检索服务」选择并点击「测试 Embedding 连接」。未配置 Embedding 时使用本地向量召回；冲突的最终判断仍需生成模型。外部产品调研还需单独配置搜索服务。

## PRD 验收编号

PRD 中引用的验收编号必须有对应定义。确认前若显示编号及行号，请编辑文档、补齐定义或修正引用，并保存新版本；同一编号出现多处不同定义时，保留一处定义，其余改成引用。存在引用问题的草稿可以继续保存，但不能确认或用于生成下游产物。编号检查仅检查文档对应关系，业务规则与历史来源仍须人工核对。

## 可选的原型浏览器验收

自动执行原型验收场景需要在服务端安装 Python Playwright 和 Chromium：`python3 -m pip install playwright`，然后运行 `python3 -m playwright install chromium`。若服务使用其它 Python 解释器，在 `.env` 设置 `PM_BROWSER_PYTHON` 为其可执行文件路径。工作台「运行与备份」可检查组件状态；不安装时仍可对每条验收项记录人工检查结果。设置 `PM_BROWSER_CHECK_ENABLED=0` 可关闭自动检查。

## 可选的复杂 PDF 本地解析

知识库导入 PDF 时，若浏览器发现图片页没有可提取文字，会暂停导入。打开“工作台配置 → 运行与备份 → PDF检测与安装说明”，可在弹窗中一键检测、安装依赖和模型并自动配置，或按Windows/macOS手动步骤准备。支持Docling 2.x（≥2.130.0），推荐已有本地回归的2.135.0。原环境变量仍兼容：`PM_DOCLING_BIN` 是程序绝对路径，`PM_DOCLING_MODEL_DIR` 是HF_HOME缓存根目录，`PM_DOCLING_ARTIFACTS_DIR` 是官方预下载模型的输出目录。参见[Docling官方安装说明](https://docling-project.github.io/docling/getting_started/installation/)和[模型准备说明](https://docling-project.github.io/docling/usage/advanced_options/#model-prefetching-and-offline-usage)。本 ZIP 不含 Python 组件或模型。 已有 Docling 环境的同目录 `python` 可用于核对 PDF 原生表格边线并补充已证明为空的合并范围；薄填充边框构成独立闭合表格、正文对应且表头字词保持时，也可纠正表头归列。子表有独立闭合边框且完全位于一个父单元格内、原解析字词保持时，可在同一来源块中恢复父子表并标出父表行列；使用包装命令时可设置 `PM_DOCLING_PYTHON` 为同一环境 Python 的绝对路径。缺少环境、边线不完整或与已有解析不一致时保留原结果，仍须核对原 PDF。

服务端只在本机离线运行该组件，一次处理一份不超过 5 MB 的 PDF，单次最多等待 180 秒；异常退出最多重试一次。配置页显示“已配置”表示CLI版本与解析接口可用；弹窗检测还会试跑合成PDF，实际文档仍须复核。解析结果会显示逐页字数、表格数和正文；操作者填写核对人、可选备注并确认后，才能作为待复核知识导入，记录可在导入历史查看。没有可定位文字的页会阻止导入。弱识别页即使有少量文字，仍须人工判断是否可靠。

管理员也可以在 `.env` 中设置 `PM_MODEL_MODE=compatible`、`PM_MODEL_BASE_URL`、`PM_MODEL_NAME` 和 `PM_MODEL_API_KEY`，再用 `node --env-file=.env server/index.mjs` 启动。`PM_MODEL_*` 会覆盖界面中的当前生成配置；移除后重启可恢复界面选择。`.env` 含有访问口令或 API Key 时不要提交到仓库。

## 0.19.6 同版本修订（2026-10-08）

本次按用户要求覆盖原0.19.6正式ZIP及同名校验文件，版本号仍为0.19.6。修正Docling主页面刷新和弹窗检测共用的短启动预算与100KB输出限制：版本/帮助检查总预算180秒，单条命令最多120秒；PDF流水线仍最多180秒。超时提示包含具体步骤，正常日志仅保留末尾，异常大量输出单独提示。弹窗复用一次接口检查和PDF试跑，已有环境检测超时时停止并显示日志，不自动重新安装。

已安装原0.19.6的服务不会因版本号相同收到升级提示，请重新下载最新ZIP和新的.sha256并按下方升级步骤停服、备份、替换程序。保留原.env、PM_DATA_DIR及operations.json；原Python环境与模型目录继续复用。不要覆盖正在运行的服务或删除数据目录。

## 升级

0.18.14起交付包包含独立备份与恢复脚本。在解压后的程序目录中运行以下命令，把`数据目录`替换为实际`PM_DATA_DIR`，备份输出目录放在数据目录之外：

```bash
node scripts/backup-data.mjs <数据目录> <备份输出目录> .env
node scripts/backup-data.mjs --verify <生成的备份目录>
node scripts/restore-data.mjs <生成的备份目录> <全新的恢复数据目录>
```

备份使用SQLite在线副本，并包含数据目录中的模型、搜索、运行和提示词配置及指定的`.env`；导入原件随数据库保存。校验失败或恢复目标已存在时拒绝恢复，不覆盖现有目录。恢复目录内的`.env`需按部署方式配置到程序根目录；持续同步的外部资料目录需另行保留。正式包不提供npm脚本，使用上述`node`入口。

停止服务，备份完整数据目录与 `.env`。将新 ZIP 解压到新的程序目录，在该目录使用原来的 `.env` 和 `PM_DATA_DIR` 启动。不要覆盖正在运行的程序目录，也不要删除数据目录。若需回退，请同时恢复升级前的数据备份。

同一数据目录只允许一个服务运行。启动时提示“数据目录已有服务运行”，请使用已有服务，或先停止它；另一个独立工作台须使用不同的数据目录。原服务异常退出后，本机确认原进程已结束才允许重新启动；未完成的模型调用仍显示结果和用量未知，重启不会自动重放对话请求。

如果提示“无法确认数据目录的服务占用状态”，先核对该目录的原服务确已停止，再移除目录中的 `.runtime-instance.sqlite` 运行占用记录后启动。使用工作台备份/恢复功能时，该临时记录会自动排除。

## 可视化并行设置

打开 **工作台配置 → 并行执行**，直接修改需求生成任务（1–50，默认 10）、知识提炼分段（1–8，默认 3）、灵感与调研（1–6，默认 3）和知识图谱提炼（1–6，默认 3）。所有项目共用；设置为 1 可相应串行执行。Agent 运行记录页仅显示当前上限并提供跳转，所有编辑集中在配置页。

修改后点击“保存并行设置”。界面提示越界/非整数，保存失败保留输入；只更新修改项，不会清空其它类型的上限。已运行请求继续，后续任务或工具轮次使用新设置。管理员仍可在数据目录 `runtime.json` 维护 `maxConcurrentJobs`、`maxConcurrentKnowledgeChunks`、`maxConcurrentResearchRequests`、`maxConcurrentGraphExtractions`，备份时一并保留。

文档提炼整份完成后才写入知识；图谱关系仍需人工审核。调研保留已完成请求断点、原定来源顺序和最多 6 个正文来源；失败或取消后不会写入迟到结果。PRD 到原型再到设计的依赖和模型输出/证据要求继续执行。

## 校验

`/api/health` 返回数据库与后台任务状态。交付包内的 `MANIFEST.sha256` 列出各运行文件的哈希值。公网交付包仅含运行所需文件，不含源码仓库、架构文档、数据库或密钥；运行文件仍可被技术人员分析，请勿将包视为不可逆向的软件。

配置原型自动检查时，可用“选择下拉项”填写选项值或显示文字；必须唯一匹配一个选项，空值可选择占位项。随后添加结果断言，检查实际交互结果。

PRD或设计文档因输出额度截断时，草稿和用量会保留；显式重试可从末尾接续。重启不会自动重发；来源变化或执行状态不明时不复用旧Markdown。重试会产生新的模型调用和用量。

选中原型元素后，自然语言修改只应用静态文字和允许的样式补丁，保留其他源码及交互。需要调整子元素、页面结构或交互时先清除选择，再描述整页修改。模型返回不支持、错误来源绑定或不完整JSON时保留原型及失败记录，不自动切换为整页重写。

分节标题中的验收项和完整编号（如 AC-FR-001-1）会进入当前 PRD 追踪索引，给定、操作和预期合为一个条目。设计追踪矩阵支持完整原文验收编号；仍须明确放在规则或验收列，并绑定真实元素。旧版索引和矩阵可保存新版本重新提取，新内容需要复核和重新验收。

功能表格中的验收定义和勾选列表也会进入当前索引；历史、示例或纯引用不新增当前验收项。正文已经定义验收项而旧索引未完整关联时，确认和下游入口会提示保存新版本重新提取、复核；读取旧记录不会自动改写正文、索引或历史检查。

PRD验收支持编号下的给定／当／那么子列表，完整条件作为一条复核；中性子标题保留上级业务规则或验收分类，历史/示例保持排除。旧索引不静默重建，需保存新版本再核对。生成设计时，模型会收到按预算保留的当前原型真实元素目录及规则/验收编号；缺失或截断映射仍须复核，目录本身不证明设计业务正确。

若历史产物提示正文含空字符，可下载完整原件，移除空字符后保存新版本并重新复核，或重新生成。旧产物不会自动改写或沿用通过状态；新生成的无效正文保留为失败草稿供下载。

损坏的历史产物会保留完整下载和版本历史，但不会参与后续检索、引用或冲突复核。修复后请保存新版本，再按原流程复核；旧正文和已记录的审阅决定保留。

## 跨源冲突知识纠错

打开知识页的跨源冲突详情，在知识侧点击“编辑知识”，修改完整标题和正文，再点击“保存并重新校验”。保存检查知识与来源版本，冲突仍存在时自动模型复核；模型不可用会提示“知识已保存”及复核错误，可再次复核。自动校验未再检出不代表模型已确认无冲突，其它关联来源继续后台检查。

0.19.4 使用 schema v37，历史跨源扫描按新规则逐步重做，保留人工处理决定和已有复核记录。单位等价、不同对象/角色/条件/生效范围不再只因表述相近触发告警。升级和回退应同时备份/恢复完整数据目录。

## 运行组件配置和一键备份（0.19.5）

进入“工作台配置 → 运行与备份”，直接调整浏览器验收启停/Python、复杂PDF启停/Docling程序/模型目录/可选Python，以及备份输出目录。路径均指服务运行设备；依赖安装方法可展开查看，保存后重新检查。配置保存在数据目录operations.json。

“立即备份”创建并校验在线数据库快照、各类配置及可用.env；“校验最近备份”复查文件哈希与数据库。外部同步资料目录需单独备份。CLI继续可用，备份和恢复已包含operations.json。schema v38保留图谱分段断点，升级前备份完整数据，回退同时恢复升级前数据。

已有知识可在“知识图谱 → 批量提炼已有知识”搜索、多选/全选并提交后台提炼，查看分段进度、失败重试。与自动提炼共用配置页的图谱模型请求上限；关闭面板后继续，重试/重启复用已验证分段，结果仍需审核。

## Windows 的 PDF 模型与浏览器组件（0.19.6）

所有步骤在运行工作台服务的 Windows 电脑执行，并使用同一账号。下面以已有的 `C:\Python314\python.exe` 为例；其它 Python/虚拟环境应替换全部路径。Python 3.14 不是本次 ENOENT 的原因：`Lib\site-packages\docling` 是包目录，不能启动。检测支持 Docling 2.x（≥2.130.0），推荐已完成本地解析回归的2.135.0；其它兼容版本需通过解析参数检查，模型完整性仍须试跑确认。

在 PowerShell 执行（需要联网）：

```powershell
& "C:\Python314\python.exe" -m pip install --upgrade "docling-slim[cli,convert-core,format-pdf,models-local,feat-ocr-rapidocr-onnx]==2.135.0"
& "C:\Python314\Scripts\docling.exe" --version
& "C:\Python314\Scripts\docling-tools.exe" models download layout tableformer rapidocr --rapidocr-backend-lang onnxruntime:ch --output-dir "C:\ProductStudioModels\docling"
& "C:\Python314\python.exe" -m pip install playwright
& "C:\Python314\python.exe" -m playwright install chromium
```

若 `Scripts\docling.exe` 不存在，使用 `& "C:\Python314\python.exe" -m pip show docling-slim` 核对安装环境；安装命令失败时先处理其错误。只安装 Playwright 包不会下载 Chromium，必须执行最后一条命令。通过浏览器访问其它电脑上运行的工作台时，应填写服务端路径。

在“工作台配置 → 运行与备份”启用组件，填写路径（字段内不添加引号或参数）：

| 字段                            | 本例填写值                                |
| ------------------------------- | ----------------------------------------- |
| Docling 可执行文件              | `C:\Python314\Scripts\docling.exe`        |
| PDF 离线模型目录（0.19.6 新增） | `C:\ProductStudioModels\docling`          |
| PDF 模型缓存目录                | 留空；旧缓存配置继续兼容 HF_HOME          |
| PDF 解析 Python（可选）         | `C:\Python314\python.exe`，或留空自动查找 |
| 浏览器验收 Python               | `C:\Python314\python.exe`                 |

“PDF 离线模型目录”对应下载命令的输出和 Docling `--artifacts-path`，不等于 Hugging Face 缓存根目录。内网离线时，在联网设备按同样版本下载，然后完整复制模型输出目录，填服务设备上的新位置。0.19.5 及以前没有独立模型目录配置，使用此流程应先升级到0.19.6。

保存并刷新：Docling 检测核对程序文件、兼容版本范围、解析参数和模型目录可访问性，实际模型完整性须上传PDF并点击“尝试增强解析”确认，输出仍需逐页复核；浏览器检测实际启动 Chromium。若出现 Playwright 缺失，说明所填 Python 环境尚未安装该包；若提示 Chromium 可执行文件不存在，说明同一环境/运行账号未完成浏览器安装。Windows 的环境变量、目录错误、依赖 stderr 和重新探测已有模拟及子进程回归，本机未运行 Windows 原机验收。

参考：[Docling 模型预下载与离线使用](https://docling-project.github.io/docling/usage/advanced_options/#model-prefetching-and-offline-usage)、[Playwright Python 安装](https://playwright.dev/python/docs/intro)。

版本检测按兼容范围`>=2.130.0 <3.0.0`及必需CLI参数判断，不再要求等于单一版本。2.130.0与2.135.0有本地合成资料解析回归，其它2.x在能力检查通过后可试用，并明确提示尚未回归；3.x和旧版暂不自动接受。安装命令锁定推荐版本用于复现环境，并非运行时只允许这一版本。解析结果核对DoclingDocument结构和已声明的1.x JSON schema，保存真实Docling版本，未知新schema拒绝导入。本机对合成中文正文/表格/扫描页做了基线和离线目录回归；Windows原机仍需试跑。

### Playwright检测中文乱码与导入失败

旧版Windows检测出现`δ��װ Python Playwright ����������`，是“未安装 Python Playwright 浏览器组件。”按GBK输出后被UTF-8读取的结果。此旧提示也可能掩盖内部依赖导入失败，不能仅凭它断言包未安装。0.19.6固定Python管道UTF-8和ASCII JSON协议，并显示实际解释器路径、区分Playwright缺失和内部导入异常。

先在PowerShell用配置页填写的同一Python执行：

```powershell
& "C:\Python314\python.exe" -c "import sys; print(sys.executable); import playwright.sync_api; print('Playwright OK')"
```

若显示找不到playwright模块，再用同一解释器安装Playwright和Chromium；若显示其它依赖错误，先处理具体错误。配置的是Python程序路径，不是Node.js/npm安装的Playwright路径。编码修复不要求用户因乱码更换Python版本。

### 组件弹窗、一键检测与安装配置（0.19.6）

“运行与备份”通过“浏览器检测与安装说明”“PDF检测与安装说明”打开弹窗；手动说明提供Windows/PowerShell与macOS/终端切换。一键检测使用指定Python，显示实际解释器、版本和分步结果；Playwright能导入不等于Chromium已安装或可启动。PDF检测还试跑合成文档，实际导入仍需逐页复核。

“安装并配置”先检查现有环境；可用时直接复用，缺少依赖时安装到选中的已有Python，不自动创建虚拟环境。浏览器流程准备Python Playwright和Chromium；PDF流程准备兼容2.x的Docling依赖、版面/表格/中文OCR模型并试跑PDF。安装需要联网、当前账号写入权限和已安装的Python3.10+；系统管理的Python禁止pip时显示原因，不绕过系统限制。失败保留原工作台配置；已执行的pip或模型下载可能留下依赖和缓存，可查看日志重试。

成功后自动回填组件路径和缓存目录，未保存的其它页面输入保持。配置变化冲突时不覆盖新的手工设置。任务在后台运行，关闭弹窗后可重新打开查看步骤和最近日志，服务重启时中断任务提示重试，不自动重新安装。

升级复用原Python、依赖、模型和浏览器缓存：不在程序启动或升级时执行pip。新下载缓存默认在版本安装目录之外的components（独立数据目录则使用其components子目录）；operations.json记录绝对路径，升级保留该文件及对应缓存位置。组件缓存不属于数据库/配置备份范围，不能只迁移operations.json到另一台机器后就认为组件可用。需要换设备时按弹窗在新设备准备并检测。

## 本地 Embedding 一键配置（0.19.7）

“工作台配置 → 模型与搜索 → Embedding 模型”点击“一键配置本地 Embedding”，先阅读说明并确认，再执行安装。默认使用 Ollama + `qwen3-embedding:0.6b`，模型约639MB；首次准备组件需联网、当前账号写入权限，建议至少8GB可用磁盘并预留运行内存。可按设备能力修改为其它本地 Embedding 模型名称，较大模型需要更多资源。

安装和模型下载在工作台**服务所在设备**执行。优先复用已有 Ollama 和模型，不使用 Python；缺少组件时从官方发布下载并校验SHA-256。新版Ollama要求Windows10 22H2+/Windows11，或macOS14+，支持x64/ARM64；旧系统可自行准备兼容版本再复用。Windows使用官方用户安装器，macOS使用官方CLI组件；设备安全提示应在服务设备处理。只监听本机11434端口。

连接试跑检查合成文本的向量后，自动添加并选中“本地 Ollama Embedding”：API地址`http://127.0.0.1:11434/v1`，模型为所选名称，API Key使用本地接口占位值`ollama`；生成模型配置保留。失败时保留原Embedding选择，已下载文件保留供重试；期间手工修改Embedding配置会阻止自动覆盖。弹窗关闭后后台继续，可重新打开查看进度和日志；重启中断的安装须重新确认，不自动重新下载。

组件与模型存放在版本目录外，升级继续复用。自行部署其它本地模型后也可用此流程检测回填，或手动添加兼容Provider。Embedding由PM_EMBEDDING_*环境变量管理时，须先移除对应覆盖并重启。Python/模型缓存与Ollama数据是不同组件；完整数据库/配置备份不包含Ollama模型，换设备需另行准备或迁移模型。

参考：[Ollama Windows](https://docs.ollama.com/windows)、[macOS](https://docs.ollama.com/macos)、[默认模型](https://ollama.com/library/qwen3-embedding:0.6b)、[兼容接口](https://docs.ollama.com/api/openai-compatibility)。
