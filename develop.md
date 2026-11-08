# 插件开发指南

从[插件模板](https://github.com/geraldpeng6/wonton-plugin-template)新建自己的仓库，把示例换成你的应用。本指南集中说明开发、检查和发布流程；清单字段、权限和 Compose 规则以[插件规范 v0.8](plugin-spec.md)为准，不在这里重复。插件仓库的 README 只介绍这个插件自己的用途和操作。

在云吞内让 Agent 写插件时，使用 [Agent Skill](skills/wonton-plugin/SKILL.md) 的工具流程；下面的脚本面向有 Docker 的开发主机，不要求环境内的 Agent 安装 Docker。

## 准备

- Docker Engine 28 或更新版本，带 Compose 插件（`docker compose`）。Linux Docker Engine、macOS Docker Desktop 和 OrbStack 均可。
- `git`、`curl`（7.76 或更新）、`python3`。
- Docker 与仓库位于同一台主机：开发环境的数据目录使用仓库内的路径，网页使用该主机的端口。Docker 在别的机器时，按[在远程主机上开发](#在远程主机上开发)操作。

不用安装云吞，不要用 root 执行脚本。套件使用发布的核心镜像 `ghcr.io/geraldpeng6/wonton-core` 检查插件和启动开发环境。

## 先跑起来

在自己的插件仓库根目录执行：

```bash
scripts/dev.sh up
scripts/dev.sh install
```

`up` 启动一套云吞并打印地址，例如 `http://template3f2a.wonton.localhost:21234/`。浏览器打开后，用以下命令输出的用户令牌登录：

```bash
scripts/dev.sh token
```

`install` 先预览插件权限，确认后安装并打印插件网页地址。Chrome、Firefox 和 curl 会把 `*.localhost` 解析到本机，不用改 hosts。

## 仓库里有什么

| 路径 | 用途 |
| --- | --- |
| `docker-compose.yml` | 插件的 Compose 和 `x-wonton` 清单 |
| 图标、网页、配置等引用文件 | 随插件包交给核心，文件范围见[插件包](plugin-spec.md#插件包) |
| `screenshots/` | 商店截图 |
| `scripts/check.sh` | 检查插件包 |
| `scripts/dev.sh` | 启停开发环境、安装插件、查看日志和冒烟 |
| `scripts/smoke-probes.sh` | 插件自己的冒烟探测钩子，不属于开发套件 |
| `scripts/publish.sh` | 检查、发布；有自建镜像时先构建并推送 |
| `scripts/kit.files`、`scripts/KIT` | 套件文件清单和来源提交 |
| `.github/workflows/check.yml` | 调用模板的可复用检查工作流 |
| `.github/workflows/publish.yml` | 在 GitHub Actions 中手动发布 |
| `AGENTS.md` | 这个仓库给人和 AI 编程代理的工作约定 |
| `LICENSE`、`NOTICE` | 插件包许可证与素材许可声明 |
| `.wonton/` | 本地开发设置、密钥和数据，不提交 |

## 写插件

逐步替换模板的示例内容，每一步先检查，再装进开发环境看效果：

```bash
scripts/check.sh
scripts/dev.sh reinstall
```

新建仓库后先执行 `scripts/dev.sh kit-update`，把套件和 CI 固定到一个提交。

按下面的顺序查规范、改文件：

1. **确定身份与需求。** 修改应用身份和版本，按[字段](plugin-spec.md#字段)、[对宿主的要求](plugin-spec.md#对宿主的要求)选择清单内容；商店文案按[商店信息](plugin-spec.md#商店信息)准备。
2. **替换服务和镜像。** 先选上游官方镜像，上游没有时按[镜像](plugin-spec.md#镜像)的规矩自建，查[镜像](plugin-spec.md#镜像)和 [Compose 子集](plugin-spec.md#compose-子集)，再改服务。`image:` 不用 YAML 锚点，见 [Compose 子集](plugin-spec.md#compose-子集)。查询多架构镜像摘要可用 `docker buildx imagetools inspect <镜像:标签>`，查看输出开头的 `Digest:`。自建镜像的操作见[发布](#发布)。
3. **接入网页和登录。** 按[网页入口与单点登录](plugin-spec.md#网页入口与单点登录)选择接入方式；需要时再查 [OIDC](plugin-spec.md#oidc) 或[网关代登录](plugin-spec.md#网关代登录sso-proxy)。用[安装时注入的变量](plugin-spec.md#安装时注入的变量)配置应用，不猜地址、令牌或用户身份。
4. **安排数据与配置。** 按 [Compose 子集](plugin-spec.md#compose-子集)安排持久化，按[配置文件](plugin-spec.md#配置文件configs)交付文件。需要首次写入应用配置时，查[一次性服务与首次写配置](plugin-spec.md#一次性服务与首次写配置)；需要凭据时，查[插件密钥](plugin-spec.md#插件密钥secrets)。开发环境中的插件数据可在 `.wonton/data/apps/<slug>/` 查看。
5. **核对网络和权限。** 一起阅读[权限](plugin-spec.md#权限)、[网络](plugin-spec.md#网络)、[局域网](plugin-spec.md#局域网networklan)与 [Compose 子集](plugin-spec.md#compose-子集)，核对应用需要的能力；需要显卡时另查[显卡](plugin-spec.md#显卡devicesgpu)。
6. **补上健康探测。** 按[健康检查](plugin-spec.md#健康检查)编写并实际观察启动结果，再运行[冒烟和插件探针](#冒烟和插件探针)。
7. **准备商店素材。** 按[商店信息](plugin-spec.md#商店信息)、[图标和截图](plugin-spec.md#图标和截图)、[图标用什么](plugin-spec.md#图标用什么)更新文案、截图、许可证和 `NOTICE`；从模板新建时更换 `LICENSE` 的作者和年份。插件 README 的图标说明链接到自己的 `NOTICE`。
8. **按需接入其他能力。** 查 [Matrix Bot](plugin-spec.md#matrix-bot)、[多个 Bot](plugin-spec.md#一个插件多个-botbots)、[Agent 插件](plugin-spec.md#agent-插件)、[模型网关](plugin-spec.md#模型网关)、[工具与 MCP 总线](plugin-spec.md#工具与-mcp-总线)。工具结果含文件时查[提供工具](plugin-spec.md#提供工具mcp)。

### 界面

布局、顺序和图标已经说明的内容，不再加副标题、说明文字或标签重复；让布局说话。错误、权限和来源提示不算多余，仍要清楚。

开发页面时，按[底栏页面](plugin-spec.md#底栏页面tabs)、[全屏](plugin-spec.md#全屏)、[主题](plugin-spec.md#主题)、[和 App 通信](plugin-spec.md#和-app-通信)接入 App；需要跨源隔离时查[跨源隔离](plugin-spec.md#跨源隔离)。

## 开发环境

| 命令 | 用途 |
| --- | --- |
| `scripts/dev.sh up` | 启动并等待健康；再次执行会拉最新核心镜像并重启，`WONTON_PULL=never` 时不拉 |
| `scripts/dev.sh install [--yes] [<git 引用>]` | 预览并确认安装；引用可为 tag、分支或提交，通过本地 git 导出，私有仓库也能用 |
| `scripts/dev.sh reinstall [--yes]` | 先卸载并保留数据，再安装工作区内容 |
| `scripts/dev.sh rebuild` | 强制重建核心栈，再保数据重装工作区插件；其他插件的容器由核心管理，不重建 |
| `scripts/dev.sh uninstall [--purge]` | 卸载插件；`--purge` 同时删除插件数据 |
| `scripts/dev.sh status` | 查看开发环境服务和已装插件 |
| `scripts/dev.sh logs [<服务>]` | 跟踪核心日志；给插件服务名则查看该服务日志 |
| `scripts/dev.sh token` | 输出用户令牌 |
| `scripts/dev.sh session [--new]` | 建立或重建用户会话，cookie 保存在 `.wonton/session.cookies` |
| `scripts/dev.sh web [--jar <文件>] [--fresh] [--sso] [<路径> [<curl 参数>…]]` | 以用户身份请求插件网页，默认复用会话；`--sso` 跟随重定向并提交 JS 自动表单的隐藏字段 |
| `scripts/dev.sh api <方法> <路径> [<curl 参数>…]` | 携带用户令牌调用核心管理接口 |
| `scripts/dev.sh egress <地址>` | 读取当前出口代理设置，保留返回的 `restarts` 和直连网段后设置新地址 |
| `scripts/dev.sh mcp list` | 在 Hermes gateway 容器中使用官方 SDK 列出 Bot 经 MCP 总线可调用的工具 |
| `scripts/dev.sh mcp call <工具> [<JSON>]` | 经 MCP 总线调用工具，走 Bot 权限、确认和审计；参数默认 `{}`。敏感工具等待卡片答复，另开终端运行 `approve` |
| `scripts/dev.sh mcp --direct list` | 在插件网络内的一次性客户端中，按工作区清单的 MCP 服务、端口和路径列出原生工具 |
| `scripts/dev.sh mcp --direct call <工具> [<JSON>]` | 直连调用原生工具名，不加 `<slug>__` 前缀；不走总线权限、确认和审计，只调试插件本身 |
| `scripts/dev.sh approve [--watch]` | 用主人令牌批准待答确认卡片；`--watch` 持续批准，Ctrl-C 停止，只在专用开发实例调试时开启 |
| `scripts/dev.sh probes [探针名…]` | 对装着的工作区插件跑冒烟探针（`web_assert`、`identity_check`、`mcp`、`data_write`、`data_check`），不起新实例；给了名字就只跑那几项 |
| `scripts/dev.sh smoke [--upgrade-from <tag>]` | 运行一次性端到端冒烟：安装、网页、身份、Bot 工具、升级保数据、重装保数据；默认从最近的 tag 升级 |
| `scripts/dev.sh pair` | 输出供 App 扫描的配对二维码 |
| `scripts/dev.sh down` | 停止核心、身份提供方和 Matrix，保留数据 |
| `scripts/dev.sh reset` | 清理当前实例的插件容器和网络，保留核心和数据，不动其他实例 |
| `scripts/dev.sh purge [--yes]` | 先删本实例插件和测试客户端，再拆核心栈、网络和卷；有残留就列出、非零退出并保留设置和数据。非交互必须带 `--yes` |

开发环境和正式安装使用同一份 Compose：核心、身份提供方 Ory Hydra、Matrix 服务器 Tuwunel；开发时换用独立的项目名、核心容器名和实例名。核心启动后同样预装模型网关 magpie（`ai.usemagpie.magpie`）。第一次 `up` 的设置保存到 `.wonton/.env`。

`up` 保留 `.env` 已有的键、注释和额外行，只补缺项并更新解析得到的 `WONTON_CORE_IMAGE`。可手改 `WONTON_PUID`、
`WONTON_PGID`、`WONTON_LAN_ADDRESSES`、出口代理、`TZ`、`RUST_LOG` 等，完整清单见文件头；实例名、基础域名和身份不要改。
`WONTON_INSTALL_TIMEOUT_SECS` 没有由核心 Compose 传入，写进 `.env` 不会生效。

修改设置后先 `up`；需要让核心栈和工作区插件的容器采用这些设置时执行 `rebuild`。工作区插件重装前照常预览、确认权限，
保留数据的规则与 `reinstall` 相同；其他已装插件不卸载、不重装。核心的 `/start` 接口只启动现有容器，不能用它代替重建。
核心重启后需要用 `session --new` 重建网页会话。

MCP 两条路径都复用 Hermes 官方镜像内的 [Python MCP SDK（MIT）](https://github.com/modelcontextprotocol/python-sdk/blob/main/LICENSE)。
直连客户端使用本实例 Hermes 的不可变镜像 ID，不在插件网络里下载依赖，不改 Hermes 容器；要求 Hermes 已安装、目标插件有独立网络。
客户端带实例标签，`--rm` 退出即删。清单没有 `mcp` 时明确报错；模板 nginx 只提供网页，不含 MCP 工具。
`approve` 读取和回答失败会报错，不当作“没有卡片”；要求核心支持 `GET /_wonton/api/confirmations` 和主人令牌回答接口。

### 实例隔离

- 实例名 `WONTON_INSTANCE` 默认是目录名去掉 `wonton-plugin-` 前缀后加 4 位随机数。核心创建的容器、网络和卷都带实例名与 `wonton.instance` 标签，例如 `wonton-template3f2a--app-example-hello--web`。共用 Docker 时，每人或每个 Agent 在自己的目录里起一套，同一个插件可以同时装在多套实例中。
- 核心地址是 `http://<实例名>.wonton.localhost:<端口>/`，插件地址是 `http://<slug>.<实例名>.wonton.localhost:<端口>/`。网页端口随机选一个空闲端口，隧道 UDP 使用下一个。
- 实例名和基础域名定下后不要改；基础域名也是 Matrix 服务器名。需要换时先清理旧实例，再创建新实例。
- `down` 不停止由核心管理的插件容器；不再需要插件时用 `uninstall`，不再需要这套环境时用 `purge`。
- 只用上述脚本起停和清理自己的实例。不要运行 `docker system prune`、`docker volume prune`，也不要操作其他实例的容器、网络或卷。
- `purge` 只匹配精确实例标签、`wonton-<实例>--` 名字前缀和完整 Compose 项目名。按实例建立的出口容器与网络一起清理；
  不带实例名的共享 `wonton-egress` 不自动删，无其他网络端点时只提示手动命令，执行前仍要确认没有实例在使用。
- 每套环境至少使用 3 个 Docker 网络：核心、magpie、开发的插件各一个。共用主机出现 “all predefined address pools have been fully subnetted” 时，由主机维护者调整 Docker `daemon.json` 的 `default-address-pools`，例如使用更细的 `/24` 子网；不要清理其他人的网络来腾空间。

### 选择核心镜像

默认核心镜像是 `ghcr.io/geraldpeng6/wonton-core:beta`。第一次启动时可指定：

```bash
WONTON_CORE_IMAGE=<核心镜像> scripts/dev.sh up
```

使用本机构建的镜像时加 `WONTON_PULL=never`。已经创建的环境，修改 `.wonton/.env` 中的 `WONTON_CORE_SOURCE` 后再执行 `up`。

### 冒烟和插件探针

合并前运行：

```bash
scripts/dev.sh smoke [--upgrade-from <tag>]
```

它把工作区复制到临时目录，起一套以 `smoke` 开头的一次性实例，通过真实请求确认安装、网页、用户身份、Bot 工具调用，以及升级和重装时用户数据还在。任何 FAIL 都以非零状态退出；跑完清理该实例的容器、网络、卷和临时目录，不动已有 `.wonton`。

“升级保数据”“重装保数据”按这个顺序跑：装起始版本 → `data_write` → 装工作区版本 → `data_check` → `reinstall` → `data_check`。有升级起点时，装工作区版本走的是 reinstall：核心对已装插件的同 id 再安装一律拒绝，保数据升级就是卸载保留数据再装回。起始版本默认是 `git describe` 找到的最近 tag，可用 `--upgrade-from <tag>` 指定；本仓库没有 tag 时只测重装，“升级保数据”记 N/A。

把插件特有的验证写在 `scripts/smoke-probes.sh`，按模板中的钩子示例改成真实业务探针；未编写的探测项会 FAIL 并显示写法。写探针时先用 `scripts/dev.sh probes [探针名…]` 在开发环境里跑通（对已装的工作区插件跑，不起新实例，给了名字就只跑那几项；数据探针名是 `data_write`、`data_check`，不给名字会连数据项一起跑），都通过了再跑 smoke。这个文件属于插件，更新套件不会替换它。不要只以进程存在或 HTTP 返回成功代替身份和工具的真实验证。

数据钩子的写法（能用的东西只有这些：`SMOKE_STATE`、`SMOKE_JAR`、`SMOKE_WEB_URL`、`SMOKE_OWNER`、`SMOKE_BASE`、`SMOKE_TOKEN`、`SMOKE_INSTANCE`，助手 `web_get`、`dweb`；e2e 的 `E2E_*` 变量和套件内部函数不进钩子）：

- `data_write` 用随机标记写一条数据（走 `dweb`、`web_get` 或应用自己的接口），把标记原样写进 `$SMOKE_STATE`——它是存数据标记的文件，`data_write` 覆盖写、`data_check` 读，不是目录。自己发请求要过网关时记得带核心会话 cookie（`-b "$SMOKE_JAR"`，`web_get`/`dweb` 已带上）。
- `data_check` 从 `$SMOKE_STATE` 读回标记，去应用里查；对不上就打印看到了什么并返回非 0。
- 应用不存用户数据时写 `DATA_NA=1`，两项记 N/A；没写钩子也没有 `DATA_NA` 时两项 FAIL 并提示写法。

几个人或几个 Agent 共用一台 Docker 时，设 `WONTON_SMOKE_MAX_STACKS=<套数>` 限制同时跑的验证栈，超过就排队，
`WONTON_SMOKE_WAIT_MINUTES`（默认 45）后放弃。不设就不排队。

需要出口代理时执行：

```bash
WONTON_DEV_EGRESS_PROXY=http://<代理地址> scripts/dev.sh smoke
```

未设置时不配置出口代理。调试单项请求时，配合 `session`、`web --sso`、`mcp list` 和 `mcp call`；探测逻辑留在插件钩子中，不修改套件来绕过失败。

### 排错

- 手工 `docker compose up -d` 判断不了插件好坏：一次性服务退出 1 也不报错（compose 只在别的服务声明了
  `service_completed_successfully` 依赖时才等它）。判断插件要走核心装一遍：`scripts/dev.sh install`，
  所有一次性服务以 0 退出才算装完。
- 安装失败的错误包含原因和失败容器的最近日志。安装后用 `scripts/dev.sh logs <服务>`，或进入自己的实例容器：`docker exec -it wonton-<实例名>--<slug>--<服务> sh`。
- `scripts/dev.sh status` 中的 `running healthy` 表示长期服务的健康检查都通过；`running unknown` 表示健康尚未知；`degraded` 后附原因。核心在安装和启动时检查，两次检查之间不会持续监控，容器停止时还要查看 `docker ps`。
- 调管理接口优先用 `scripts/dev.sh api GET /_wonton/api/plugins`；也可使用 `up` 输出的核心地址直接请求：

  ```bash
  curl -H "Authorization: Bearer $(scripts/dev.sh token)" \
    http://<实例名>.wonton.localhost:<端口>/_wonton/api/plugins
  ```

### 在远程主机上开发

Docker 在另一台 Linux 主机时，把仓库克隆到那台机器，在远程仓库目录执行脚本。浏览器仍用自己的电脑，通过 SSH 转发 `up` 打印的网页端口，例如在本机执行：

```bash
ssh -L 21234:127.0.0.1:21234 <远程主机>
```

把 `21234` 换成实际端口，打开 `up` 输出的同一个地址。`*.localhost` 在本机解析，再经转发到远程核心，域名原样传到网关。不要只把 Docker context 指向远程而把仓库和数据留在本机。

### 在 App 里看

```bash
scripts/dev.sh pair
```

用云吞 App 扫终端里的二维码连接这套开发环境。手机在局域网时，需要能访问开发主机的 UDP 隧道端口（`.wonton/.env` 的 `WONTON_TUNNEL_PORT`）；在外面时经中继连接。网页的 SSH 转发不代替这条 UDP 通路。

## 检查

```bash
scripts/check.sh
scripts/check.sh --official
```

脚本在核心镜像里调用 `wonton-core plugin check`，不连接正在运行的开发核心；检查、打包和核心能力探测的临时容器都用 `--network none`，不依赖 Docker 默认 bridge。所需镜像须在本地或先拉取。输出问题、插件包文件和摘要；退出码、提醒与官方索引要求见[开发与校验](plugin-spec.md#开发与校验)。默认镜像不能读取现行 spec 时，根据错误中的版本信息用 `WONTON_CORE_IMAGE` 选择支持该规范的镜像。

插件仓库的 `.github/workflows/check.yml` 在 PR 和推送 main 时调用模板的 `plugin-check.yml`：检查 shell、执行 `scripts/check.sh`，有 `Cargo.toml` 时还运行 Rust 格式、Clippy 和测试，并检查受 Git 管理的指定源码文件行数。CI 失败时修复问题，不跳过失败。工作流版本随[开发套件](#更新开发套件)更新；需要覆盖检查使用的核心镜像时，通过可复用工作流的 `core-image` 输入指定。

PR 上不跑 `--official`：重建自家镜像（`images/` 下有 Dockerfile）的 PR 在发布前拿不到新摘要，这类 PR 会永远红着。官方索引的要求由发布流程统一查——`publish.yml` 调 `scripts/publish.sh --official`，在构建推送镜像、把摘要写回 Compose 之后执行，这时摘要才存在。

## 更新开发套件

在插件仓库执行：

```bash
scripts/dev.sh kit-update
```

可选传入模板仓库的 ref，例如 `scripts/dev.sh kit-update <ref>`；省略时使用 `main`。命令先解析来源提交、获取新的 `scripts/kit.files`，再按新清单下载全部文件；全部取到才替换。取不到时退出报错，不改原有文件，不留下半套更新。更新不需要 Docker。完成后查看命令打印的 `git diff --stat`，把套件更新与插件改动一并纳入自己的审查流程。

仓库是在套件出现以前从模板建的（没有 `scripts/KIT`）时，先从模板 main 复制 `scripts/dev.sh` 和 `scripts/lib.sh` 过来，再执行上面的命令。

`scripts/kit.files` 列出的文件由模板维护，**不要在插件仓库中修改套件文件**；插件自定义命令另写脚本。`scripts/smoke-probes.sh` 是插件自己的钩子，不属于套件，继续在插件仓库中维护。

`scripts/KIT` 用一行 40 位提交 SHA 记录套件来源。插件的 `.github/workflows/check.yml` 固定调用模板 `.github/workflows/plugin-check.yml@<同一 SHA>`；更新命令同时改写这条固定引用，使本地套件和 CI 对应同一来源提交，而不是让 CI 跟随浮动的 `main`。模板仓库是套件源头，在模板中执行更新只提示来源并退出，不执行更新；模板自己的检查使用本地工作流引用。

从模板复制的 `.github/workflows/plugin-check.yml` 属于套件，更新成功后会删除这份本地副本并打印说明；插件 CI 只调用模板维护的那份。

## 发布

上游镜像发新版时，仓库里的 `.github/dependabot.yml` 会让 Dependabot 自动开升级 PR：标签和摘要一起改。这个文件从模板来，之后归插件自己维护，`dev.sh kit-update` 不覆盖它。

compose 里有自己数据目录的数据库服务（postgres、mariadb/mysql、mongo、elasticsearch、meilisearch 等），在 `dependabot.yml` 里忽略它的大版本升级：大版本之间常要迁移数据，自动升上去会让主人的数据读不出来。大版本升级手动做，在发版说明里写清迁移步骤。

```yaml
    ignore:
      - dependency-name: "postgres"
        update-types: ["version-update:semver-major"]
```

升级 PR 的 CI 自动带端到端冒烟，两项检查都过了就合；升级 PR 里不改版本号和发版说明，攒到发版时一起改——`scripts/publish.sh` 会校验版本比上一个 tag 新、发版说明和上一个 tag 不同，没改会停下提示。

1. 改清单版本并填写更新说明，核对许可声明及引用文件，提交插件改动；字段和版本约束查[商店信息](plugin-spec.md#商店信息)、[插件仓库](plugin-spec.md#插件仓库)和[升级](plugin-spec.md#升级)。
2. 先完成检查和冒烟。公开仓库可在 GitHub Actions 中手动运行“发布”（`.github/workflows/publish.yml`，默认带 `--official`）。它用 `GITHUB_TOKEN` 推送镜像，本机无需登录镜像仓库。
3. 私有仓库或本地发布时，在插件仓库执行：

   ```bash
   scripts/publish.sh
   # 按官方商店要求发布时：
   scripts/publish.sh --official
   ```

   脚本要求工作区干净，先检查插件包；有 `images/` 时构建、推送镜像并把摘要写回提交，然后打 `v<版本>` tag，推送当前分支和 tag。仅使用上游镜像的插件直接进入打 tag 的流程。tag、预发布版本及来源的处理见[插件仓库](plugin-spec.md#插件仓库)和[来源](plugin-spec.md#来源)。
4. 想进入社区列表时，为公开仓库设置 topic `wonton-plugin`，按[社区插件](plugin-spec.md#社区插件)完成发布。申请官方收录时，按[官方索引](plugin-spec.md#官方索引)向索引仓库提 PR。私有仓库可用 `scripts/dev.sh install <tag>` 从本地 git 导出安装。

### 自建镜像

把 Dockerfile 放在 `images/<名字>/`，在 Compose 中写待发布的 `ghcr.io/<你>/<名字>:<版本>` 镜像。`publish.sh` 用 `docker buildx` 按清单的 `requirements.arch` 构建（没写时构建 `linux/amd64` 和 `linux/arm64`），推送后把 Compose 镜像引用改为新摘要并提交。在本机发布前准备支持多架构构建的 buildx，并用 `docker login` 登录镜像仓库。第一次推送后，在 GitHub 包设置里确认镜像公开，保证用户主机可拉取。

开发时可先在开发主机构建同名镜像：

```bash
docker build -t ghcr.io/<你>/<名字>:<版本> images/<名字>
```

默认拉取策略下本机已有该镜像时可直接使用。镜像引用和核心支持范围查[镜像](plugin-spec.md#镜像)与 [Compose 子集](plugin-spec.md#compose-子集)，不把本机构建步骤写进插件安装流程。

自建镜像从写好到发布分三步，每一步能证明的东西不同：

1. **本机验证。** Compose 里只写标签（`ghcr.io/<你>/<名字>:<版本>`），用上面的命令在开发主机构建，再 `scripts/dev.sh install`、`smoke`。这一步证明插件在这台机器上能装能用。`scripts/check.sh` 提醒“镜像没有按摘要固定”是正常的；`--official` 因此不合格也是正常的。
2. **交给别人审查，还没获准发布。** 别的机器拉不到你本机的镜像，审查者要自己构建。PR 说明里写清：镜像还没发布、Compose 没有摘要，构建命令（`docker build -t <镜像:标签> images/<名字>`），以及你验证时的平台（`docker version` 里的 `OS/Arch`）。不要为了让 `--official` 通过而自己推送镜像。
3. **获准发布后。** 运行 `scripts/publish.sh`（或 GitHub Actions 的“发布”）。它按 `requirements.arch` 构建、推送，把仓库返回的清单列表摘要写回 Compose，这时 `--official` 才能通过。

摘要只能来自推送到镜像仓库后返回的结果或 `docker buildx imagetools inspect`。本机的镜像 ID（`docker images` 里的 `IMAGE ID`、`docker inspect` 的 `Id`）不是镜像仓库里的摘要，不能写成 `@sha256:…`：别人按它拉不到镜像。
