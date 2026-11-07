# 插件规范 `x-wonton`（草案 v0.6）

> 草案，字段可能调整。正式发布之前规范直接改，不保留旧写法：核心只认当前的写法，插件跟着升。

一个插件就是一个 `docker-compose.yml`，在顶层加一个 `x-wonton` 扩展字段描述它在 Wonton 里的身份、
入口、能力和权限。compose 引用的配置文件、图标和设置表单与它放在同一个目录里，合称插件包。
没有 `x-wonton` 的 compose 文件可以由用户手动导入，但必须先补全权限声明。一个插件一个 Git 仓库，仓库根目录
就是插件包，发布就是打 tag（见“发布与来源”）。可运行的示例见
[插件模板](https://github.com/geraldpeng6/wonton-plugin-template)，官方插件的列表见
[官方索引](https://github.com/geraldpeng6/wonton-store)。

## 示例

```yaml
name: immich
services:
  server:
    image: ghcr.io/immich-app/immich-server:v2
    volumes:
      - ${WONTON_APP_DATA}/upload:/usr/src/app/upload
      - ${WONTON_SHARE_PHOTOS}:/photos:ro
    healthcheck:
      test: ["CMD", "immich-healthcheck"]
  mcp:
    image: ghcr.io/example/immich-mcp:1
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://127.0.0.1:8080/healthz"]

x-wonton:
  spec: "0.6"
  id: app.immich.photos
  version: 2.0.0
  kind: app
  title: { en: Immich }
  description: { zh: 备份手机里的照片和视频，按人脸和内容搜索, en: "Back up your photos and videos, search by face and content" }
  icon: icon.png
  screenshots: [screenshots/timeline.png, screenshots/search.png]
  author: { name: Example, url: https://example.com }
  homepage: https://immich.app
  license: AGPL-3.0-only
  tags: { zh: [相册, AI], en: [Photos, AI] }
  about:
    zh: |
      手机里的照片自动备份到家里的服务器。

      按人脸、地点和画面内容搜索照片。
    en: |
      Back up the photos on your phone to the server at home.

      Search photos by face, place and what is in them.
  release_notes: { zh: 升级到 Immich 2.0。, en: Upgrades to Immich 2.0. }
  webui:
    service: server
    port: 2283
    sso: oidc
    oidc:
      redirect_paths: [/auth/login, /user-settings]
  mcp:
    service: mcp
    port: 8080
    transport: streamable-http
  permissions:
    - storage.share.photos:read
  settings: settings.schema.json
  requirements:
    arch: [amd64, arm64]
    memory: 2G
```

## 字段

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `spec` | 是 | 清单按哪一版规范写成，见“商店信息”。 |
| `id` | 是 | 反向域名形式，全局唯一，安装后不可修改。至少两段，每段由小写字母、数字和单个连字符组成，总长不超过 63。 |
| `version` | 是 | 插件包版本（语义化版本），和镜像版本无关。 |
| `kind` | 是 | `app`（普通应用）、`agent`（Agent 运行时）、`channel`（第三方 IM 渠道）、`model`（模型服务）、`system`（系统插件，仅官方，只能由安装器预装）。 |
| `title`、`description` | 是 | 多语言文本，至少写一种语言。怎么写见“商店信息”。 |
| `icon` | 否 | 插件包里的 PNG 或 SVG，见“商店信息”。官方索引要求必填。 |
| `screenshots`、`author`、`homepage`、`license`、`tags`、`about`、`release_notes` | 否 | 商店信息，见“商店信息”。官方索引要求有 `tags`、`author` 和 `license`。 |
| `notices` | 否 | 插件包里的许可和声明文件（UTF-8 文本）的路径列表，见“插件包”。官方索引要求必填。 |
| `webui` | 否 | 作为小程序打开的入口：`service`、`port`、`path`（默认 `/`）、`sso`（`oidc`、`proxy`、`none`，默认 `none`）、`oidc`、`proxy`（见“网页入口与单点登录”）、`display`（见“全屏”）、`cross_origin_isolated`（见“跨源隔离”）。 |
| `tabs` | 否 | 能放到 App 底栏的页面，要和 `webui` 一起声明，见“底栏页面”。 |
| `mcp` | 否 | 给 Agent 的工具入口：`service`、`port`、`path`（默认 `/mcp`）、`transport`（`streamable-http`），可选 `headers`（见“提供工具”）、`tool_idle_seconds`、`tool_total_seconds`、`tools`（只提供列出的工具），见“工具与 MCP 总线”。 |
| `bot` | 否 | Bot 的显示名（`display_name`，多语言）、静态头像（`avatar`）和状态头像（`avatars`：`idle`、`working`、`waiting`、`offline` 四个状态各一个文件），都是插件包里的文件。只能和 `matrix.bot` 权限一起声明，见“Matrix Bot”。 |
| `bots` | 否 | 一个插件多个 Bot：`services` 列出每个 Bot 一份的服务（见“一个插件多个 Bot”）。 |
| `agent` | `kind=agent` 时必填 | `protocol`：`matrix`（Agent 自己登录 Matrix，须声明 `matrix.bot`）或 `acp`（经 ACP 桥接入）。可选 `login`，见“Agent 插件”。 |
| `provides` | 否 | 向其他插件提供的能力，v0.1 只有 `models`（见“模型网关”）。只有 `kind: system` 的插件可以声明。 |
| `gpu` | 否 | 拿到显卡的服务（`services`）。要和 `devices.gpu` 权限一起声明，见“显卡”。 |
| `lan` | 否 | 在局域网上开放的端口（`ports`）。要和 `network.lan` 权限一起声明，见“局域网”。 |
| `secrets` | 否 | 插件自己的密钥：核心生成后注入，或插件运行中生成、交给核心保管（见“插件密钥”）。 |
| `permissions` | 是 | 安装时由用户确认的权限，见下表。没有权限时写空列表。 |
| `settings` | 否 | 插件包里的 JSON Schema 文件，留给 App 用原生表单渲染（预留）；在那之前插件用自己的网页做设置。 |
| `requirements` | 否 | 对宿主的要求：`arch`、`memory`、`gpu`，核心按这台主机判断，见“对宿主的要求”。 |

`x-wonton` 里出现未知字段时安装失败；`spec` 比核心支持的版本新时，先报规范版本（见“商店信息”）。

## 插件包

插件包是一个目录：`docker-compose.yml`，加上它引用的文件——`icon`、`settings`、`bot.avatar`、`bot.avatars`、`notices` 和
顶层 `configs` 的 `file`。`screenshots` 不算在内（见“商店信息”）。

- 引用的路径相对 compose 文件所在的目录，不能是绝对路径、不能含 `..`、不能用变量，也不能经过软链接；
  单个文件不超过 2 MiB，整个插件包（文件大小相加）不超过 16 MiB。引用的文件不存在时安装失败。
- 安装接口收 compose 文本，或者打包好的插件包目录：tar（如 `tar -C <目录> -cf - .` 或 `wonton-core plugin pack` 的输出）、
  tar.gz 和 zip，按内容认格式，不看扩展名和 `Content-Type`。compose 引用了包里的文件时只能传压缩包。
- `docker-compose.yml` 可以在压缩包的根目录，也可以在唯一的顶层目录里（如 GitHub 的 “Download ZIP”）。核心只读 compose
  引用的文件，其余的不解压也不保存，不受上面单文件大小的限制。被引用的路径上不能有软链接、硬链接和设备。压缩包最多
  16 MiB，解压后合计最多 64 MiB，最多 1024 项。
- 核心只保存被引用的文件。安装后整个包（compose 原文和这些文件）存在核心自己的数据目录里，供审计、
  恢复和升级比对；插件的容器访问不到这个目录。
- 卷不能用相对路径挂载包里的文件（见“Compose 子集”）；容器要用的包内文件一律经 `configs` 交给它。
- 包里的第三方素材（如上游的标志）要求随分发附上版权和许可时，写进 `notices` 引用的文件。README 不进包，核心重新编码图片时
  也不留元数据，这两处的声明到不了用户那里。`notices` 只随包保存和分发，容器拿不到；容器要读的文件用 `configs`。

### 配置文件（`configs`）

写法和 compose 相同：

```yaml
services:
  app:
    configs:
      - source: app-config            # 长语法；短语法只写名字，放到 /<名字>
        target: /etc/app/config.yaml
        uid: "1000"
        gid: "1000"
        mode: 0440
configs:
  app-config:
    file: ./config.yaml               # 插件包里的文件
  motd:
    content: |                        # 内联内容，按注入的变量插值
      model=${WONTON_MODEL_DEFAULT}
```

- 顶层 `configs` 只支持 `file` 和 `content`；`external`、`environment`、`name` 报错。
- 核心把配置交给容器，`read_only: true` 的服务也能用：这样的服务以只读单文件绑定挂载读核心数据目录里
  的文件，每次部署都重新写入，升级重建容器后就能读到新内容。其余服务在创建容器之后、启动之前用 Docker
  的归档接口把文件写进容器，每次创建容器都重新写入。两种做法里 `uid`、`gid`、`mode` 对 `file` 也生效
  （docker compose 对 `file` 用绑定挂载，这三项只对 `content` 生效）。应用要自己改写的配置，用下面的
  一次性服务复制进数据目录。
- `target` 是容器里的绝对路径，不能和这个服务的挂载点（卷、绑定挂载、`tmpfs`）重叠：写进挂载点的文件
  会留在卷或数据目录里，`tmpfs` 则在启动时把它盖住。
- 没写 `uid`、`gid` 时，属主取服务的 `user`：数字形式的 `uid:gid` 就是它，只有 uid 时组是 0，没写
  `user` 或写的是用户名时是 root。`mode` 按八进制理解（`0440`、`"0440"`、`0o440` 都是 0440），默认
  `0444`，写权限位被忽略，不能超过 `0777`。默认所有用户可读，收紧了 `mode` 才需要关心属主。

### 一次性服务与首次写配置

写了 `restart: "no"` 的服务是一次性服务，不论有没有人依赖它，都不需要健康检查。被其他服务以
`condition: service_completed_successfully` 依赖的服务也必须写 `restart: "no"`，没写或写了别的取值时安装
失败：否则 Docker 会在它退出后反复重启它。

- 核心按依赖顺序创建并启动容器，一个服务要等它的每个依赖达到声明的条件才启动。一次性服务也可以依赖长期
  服务的 `service_healthy`：先等应用健康，再做初始化。
- 安装要等所有一次性服务以 0 退出才算完成，装好之后才能做的收尾（比如等主服务健康后建账号）没有人依赖也
  照等。等待中任何一个依赖已经失败——一次性服务非 0 退出、长期服务退出或健康检查判失败——安装立即失败，
  错误里是它的名字、退出码和日志，不等其余依赖到超时。
- 核心每次创建插件的容器（安装、升级、重建）时都会重新运行一次性服务，所以它必须是幂等的。
- 安装后、核心重启后检查状态时，“已退出、退出码 0”是它的正常状态（正被重新运行着也算正常），以其他退出码
  退出或容器不见了时插件标为异常。
- 不需要网络时写 `network_mode: none`。用应用自己的命令行改配置的（如登记 OIDC 认证源），这类命令常要先有主服务建好的配置和数据目录，还会连发现文档：依赖主服务 `service_healthy`，不写 `network_mode: none`。

应用会自己改写配置文件时（如 Hermes 的 `config.yaml`），由一次性服务在首次启动前把包里的配置复制进
数据目录，已存在就不覆盖。这和 Kubernetes 的 init 容器是同一种做法：

```yaml
services:
  init:
    image: ghcr.io/example/hello:1.2.3@sha256:…
    entrypoint: ["/bin/sh", "-c", "test -f /data/config.yaml || cp /seed/config.yaml /data/config.yaml"]
    volumes:
      - ${WONTON_APP_DATA}:/data
    configs:
      - source: config
        target: /seed/config.yaml
    network_mode: none
    restart: "no"
  app:
    # 同一镜像各写一遍，不用锚点别名（见“Compose 子集”）。
    image: ghcr.io/example/hello:1.2.3@sha256:…
    depends_on:
      init:
        condition: service_completed_successfully
```

其余常见的三种写法。

启动前准备目录（应用要求的子目录、权限，应用起来之前就绪）：

```yaml
services:
  init:
    image: busybox
    command: ["sh", "-c", "mkdir -p /data/library /data/tmp"]
    volumes: ["${WONTON_APP_DATA}:/data"]
    network_mode: none
    restart: "no"
  app:
    image: *image
    volumes: ["${WONTON_APP_DATA}:/data"]
    depends_on:
      init: {condition: service_completed_successfully}
```

等应用健康后经 HTTP 初始化（建管理员账号、导首批数据这类要应用先起来的活）：

```yaml
services:
  app:
    image: *image
    healthcheck: {test: ["CMD", "wget", "-qO-", "http://127.0.0.1:8080/healthz"]}
  init:
    image: curlimages/curl
    command: ["-fsS", "-d", "user=admin", "http://app:8080/setup"]
    depends_on:
      app: {condition: service_healthy}
    restart: "no"
```

每次启动覆盖会变的值（时区、PUID 这些跟着核心走的值会变；它每次重建都会运行，必须幂等，直接覆盖写
最新的，不要判断存在）：

```yaml
services:
  init:
    image: busybox
    command: ["sh", "-c", "printf 'tz=%s\\npuid=%s\\n' \"$TZ\" \"$PUID\" > /data/owner.env"]
    environment: ["TZ=${TZ}", "PUID=${PUID}"]
    volumes: ["${WONTON_APP_DATA}:/data"]
    network_mode: none
    restart: "no"
  app:
    image: *image
    volumes: ["${WONTON_APP_DATA}:/data"]
    depends_on:
      init: {condition: service_completed_successfully}
```

## 商店信息

商店、安装确认页和 App 主屏用到的信息写在清单里，随插件包按提交固定。给用户看的文字都按语言写，语言是键，至少写一种。键是 BCP 47 语言标签：`en`（英文）、`zh`（简体中文）、`zh-Hant`（繁体中文）、`ja`（日文）等。App 按用户在用的语言挑：先找完全一样的，再按文字系统找，`zh-TW`、`zh-HK` 用 `zh-Hant`，没写 `zh-Hant` 时用 `zh`；都没有时回退到 `en`，再回退到写了的任意一种。繁体中文和日文的用户看到的是你写的 `zh-Hant`、`ja`，没写就是简体或英文，建议补上。官方索引要求 `description`、`tags`、`about`、`release_notes` 都写 `zh` 和 `en`。

| 字段 | 说明 |
| --- | --- |
| `spec` | 必填。清单按哪一版规范写成，字符串，如 `"0.6"`（YAML 里要加引号，不然读成数字）。没写时安装失败。核心先读它：比核心支持的版本新时安装失败（`unsupported`），提示先升级核心，不再逐项报未知字段。 |
| `icon` | 图标，见下节。 |
| `screenshots` | 截图：仓库里的 PNG 或 JPEG（路径规则同插件包里的文件），1 到 6 张，按顺序显示。截图不算插件包的一部分：不参与插件包摘要，安装时不取，上传插件包时忽略；商店的详情页要显示时，核心按同一个提交去取。 |
| `author` | 插件包的作者：`name`（必填）、`url`（HTTPS，可选）。不一定是应用的作者。 |
| `homepage` | 应用的主页，HTTPS。 |
| `license` | 应用的许可证，SPDX 表达式，如 `AGPL-3.0-only`、`MIT OR Apache-2.0`。插件包自己的许可证写在仓库的 LICENSE 里。 |
| `tags` | 标签，插件自己写，按语言分开，如 `{ zh: [相册, AI], en: [Photos, AI] }`。每种语言最多 8 个，每个 1 到 24 个字符，不能重复。商店按出现过的标签筛选，所以写用途，不写技术词；也用来搜索。 |
| `about` | 详细介绍：多语言，纯文本，空行分段，每种语言不超过 4000 个字符。没写时详情页只显示 `description`。 |
| `release_notes` | 这个版本的更新说明：多语言，纯文本，每种语言不超过 2000 个字符，升级提示里显示。数据迁移不可逆时要写明。 |

标签不是固定的一组，下面这些是推荐的词，大家用同一个词，筛选时才归得到一起。没有合适的可以自己写。

| `zh` | `en` | 例子 |
| --- | --- | --- |
| AI | AI | Open WebUI、Vane |
| 相册 | Photos | Immich |
| 音乐 | Music | Navidrome |
| 视频 | Video | Jellyfin |
| 文件 | Files | File Browser、Samba |
| 同步 | Sync | Syncthing |
| 文档 | Documents | Paperless-ngx |
| 笔记 | Notes | Memos |
| 阅读 | Reading | Miniflux |
| 记账 | Finance | Actual |
| 效率 | Productivity | 待办、日历 |
| 通讯 | Communication | 第三方 IM 渠道 |
| 智能家居 | Home | Home Assistant |
| 网络 | Network | DNS、代理 |
| 监控 | Monitoring | Uptime Kuma |
| 安全 | Security | 密码管理 |
| 开发 | Developer | Git 服务 |
| 下载 | Downloads | BT 下载 |
| 工具 | Tools | 其他小工具 |

`title` 写产品原名，如 Immich、Syncthing，不要翻译成用途，用途交给 `tags`。原名不译时只写 `en`，如 `title: { en: Immich }`。插件自己就是产品、没有上游时才用功能名，`zh` 和 `en` 都写。

`description` 和 `about` 写给普通用户：`description` 一句话说清它能帮你做什么，不写技术实现，读者称呼“你”。

### 图标和截图

- 图标是插件包里的 PNG 或 SVG，正方形：
  - PNG 至少 256×256，建议 512×512，不超过 512 KiB；SVG 的 `viewBox` 是正方形，不超过 128 KiB。
  - 满幅：画面铺满整个正方形，不留透明边，不画圆角和阴影。App 统一裁成圆角方形，四角会被裁掉，重要内容放在中间 80%
    的范围里。
  - SVG 里的文字不渲染（要用文字就转成路径），外部资源（图片、字体、样式表）和内嵌的位图都不加载，脚本和动画不执行。
  - SVG 要是 UTF-8 文本，根元素是 SVG 命名空间里的 `<svg>`，不能有 `<!DOCTYPE>`，不收压缩过的 SVG（svgz）。
  - `<use>` 只有一个 `href` 或 `xlink:href`，写成 `#` 加 id，id 里没有空白；展开以后不超过 20000 个元素、64 层，不能循环
    引用。
  - `<style>` 里只用不带组合符的选择器：`.cls-1`、`#a`、`rect.a` 可以，`g rect`、`g > rect`、`a + b` 不行；最多 256 条
    规则。渲染器匹配后代选择器时逐层回溯，几百字节的样式表就能算上几分钟。
  - 不能用 `<pattern>`、`<marker>` 和 `<filter>`（阴影和模糊本来就不该画）；`<clipPath>` 的 `clipPathUnits` 只能是
    `userSpaceOnUse`（默认值），`<mask>` 要写 `maskUnits="userSpaceOnUse"`；每个渐变最多 64 个 `<stop>`。渲染器会按引用的
    次数复制这些功能的内容，不加限制时几 KB 的文件就能占掉几 GB 内存。
  - 不接受 HTTPS 地址：图标随提交固定，App 也不向第三方取图。
- 截图是 PNG 或 JPEG，每张不超过 1 MiB，建议长边至少 1280，横屏和竖屏的截图不要混在一个插件里。
- 核心把图片解码后重新编码再交给 App：图标是 192×192 和 512×512 的 PNG，截图是长边不超过 1280 的 JPEG，都不带原文件的
  元数据。解码和渲染在单独的进程里做，每张图最多用 512 MiB 内存、10 秒（很细的虚线这类写法渲染起来很慢）。解码不了的、
  解码后超过 4096×4096 像素的、处理时超出内存或时间的图片，和不是正方形的图标都不合法：图标不合法时安装失败，截图不合法
  时详情页里不显示它。`wonton-core plugin check` 按同样的规则、同样在单独的进程里检查。
- `bot.avatar` 的格式不变（见“Matrix Bot”）。

#### 图标用什么

官方插件照这一节做，第三方插件建议照做。`wonton-core plugin check` 不检查这些，官方插件在代码审查时看。

上游有标志时用上游的官方标志：

- 原样使用，不改色、不变形，不和别的图形拼在一起。透明或不是正方形的标志，等比放在纯色底上，用上游规定的底色；没有规定时用白色，标志是白色或很浅时用 `#111111`。
- 上游的 SVG 含文字、内嵌位图或滤镜时改用 PNG 版，上面的 SVG 规则不收这些内容。
- 不暗示 Wonton 是对方官方出品，也不暗示得到了对方背书。
- README 写明标志的来源，并注明它是对方的商标。
- 标志的来源、版权、许可全文和商标说明放进一个文件（如 `NOTICE`），用 `notices` 引用。

图标放好后缩到 62×62（主屏的显示尺寸）看一眼。

上游没有标志时（如示例插件）按下面的线条风格画，官方插件的 `notices` 写插件包自己的许可：

| 项目 | 规矩 |
| --- | --- |
| 文件 | 优先 `icon.svg`，所有尺寸都清晰；`viewBox="0 0 512 512"` |
| 底色 | 满幅 `#F4F4F4`，不画圆角和阴影 |
| 线条 | `#111111`，线宽 18，圆头圆角，全图同一种线宽；小实心点只用 `#111111` |
| 色块 | 一种浅色，平铺无渐变，形状跟随主体，比线条往右下错开 14（`translate(14 14)`），放在线条下面 |
| 范围 | 主体放在中间 112 到 400 的范围里，约占画面 55% |
| 内容 | 一个主体，笔画尽量少，不放文字和字母 |

色块从下面八种里选一种。图标是固定的图片，颜色不随用户在 App 里选的重点色变化。

| 粉 | 橙 | 黄 | 绿 | 青 | 蓝 | 紫 | 灰 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `#FFBEBD` | `#FEC89C` | `#F5DF8A` | `#B2E4B2` | `#8FE7E1` | `#A5DBFF` | `#D8CAFF` | `#D4D4D4` |

模板：

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512">
  <rect width="512" height="512" fill="#F4F4F4"/>
  <circle cx="270" cy="270" r="132" fill="#A5DBFF"/>
  <g fill="none" stroke="#111111" stroke-width="18" stroke-linecap="round" stroke-linejoin="round">
    <circle cx="256" cy="256" r="132"/>
    <circle cx="256" cy="256" r="34"/>
    <path d="M256 170A86 86 0 0 1 342 256"/>
    <path d="M256 342A86 86 0 0 1 170 256"/>
  </g>
</svg>
```

形状难画时可以先让图像模型出稿找方向，再照着写成 SVG。模型的线宽和底色不够准，它的位图不直接当图标。提示词：

```text
Minimal app icon, flat vector. Solid plain light grey background #F4F4F4 filling the whole square edge to edge, no rounded corners, no border, no shadow, no gradient, no text, no letters. One centered subject occupying the central 55 percent, drawn only with a uniform thick black monoline (#111111) with round caps and round joins, about 3.5 percent of the canvas width, in the style of Phosphor icons. Behind the lines, one flat solid {色块的颜色} shape following the subject, printed slightly misaligned toward the lower right like a riso print. No other colors, no shading. Subject: {用英文描述形状}.
```

线条在 62×62 上糊成一团就减笔画。

## 网页入口与单点登录

插件网页的地址是 `<scheme>://<slug>.<基础域名>[:端口]`，`slug` 是把 `id` 里的 `.` 换成 `-`
（`app.immich.photos` → `app-immich-photos`）。所有请求经核心网关，网关先校验用户的会话，未登录的请求
不会到达插件；转发时去掉网关自己的 Cookie 和客户端伪造的 `X-Forwarded-*`，重新设置 `X-Forwarded-For`、
`X-Forwarded-Host`、`X-Forwarded-Proto`，保留原始 `Host`，支持 WebSocket。

`webui.path`（默认 `/`）是入口路径：App 和状态页打开插件网页时，把它拼在网页地址后面，写
`/vnc.html?autoconnect=1&resize=remote` 就直接落到应用的远程桌面页。网关按域名转发，代理不因此改路由，
应用收到的请求路径也不变——入口路径只决定打开时落在哪一页。

入口路径的写法（`tabs` 的 `path` 同一条规则，`plugin check` 校验）：

- 以 `/` 开头的绝对路径，可以带 `?query`。
- 不能有 `#`、空白和控制字符。
- URL 里不允许原样出现的字符（非 ASCII、`"`、`<`、`>` 等）必须已经百分号编码（如 `%20`）。核心原样
  拼接，不再编码。

`webui.sso` 决定用户怎样登进应用：

| 写法 | 什么时候用 |
| --- | --- |
| `none` | 默认，新插件都从它开始。网关只校验会话，应用用自己的登录，或者本来不需要登录（下一节）。 |
| `oidc` | 应用官方支持 OIDC，由核心的身份提供方登录（见“OIDC”）。 |
| `proxy` | 应用官方支持反向代理传来的用户名，或者认一个固定的密钥，由网关代为带上（见“网关代登录”）。 |

声明 `oidc` 或 `proxy` 要同时满足：应用官方支持这种接法；接上后打开网页不会再看到应用的登录页，角色也对；不用脚本
往应用里预建账号，不用改应用的数据，不用每次启动核对。做不到就用 `none`。

### 应用自带的登录（`sso: none`）

用户装好后第一次打开网页时，在应用里自己注册管理员。网关挡在前面，只有用户打得开注册页。

- 不要带默认账号和默认密码。应用非要预建管理员时，用 `secrets` 生成密码，不要写死。
- 应用自己在第一次启动时建一个固定的默认账号（如 Shiori 的 `shiori`/`gopher`）：有配置项能改初始密码的，用 `secrets` 生成的值；
  没有的，照常 `sso: none`，`about` 里写明默认账号和“第一次登录后改密码”。网关挡着，改密码之前只有用户打得开这个页面。
- 注册页只在 `webui` 端口上。`lan.ports` 列出的别的端口装好就开在局域网上，不能提供未认证的管理员注册。`webui` 的端口列进 `lan.ports` 时默认不开，确认页和插件设置里有开关 `lan_webui`，默认关，旁边写明“管理员注册之前，局域网里的设备和这台主机上能出网的插件可能抢先注册”。
- 应用能关自助注册时，默认关掉第一个账号以外的注册。
- 应用的登录页关不掉就留着，局域网浏览器里用户登两次。不要为了去掉它去改应用。
- 插件自己的服务要访问应用（比如 MCP 服务调应用的接口）时，用 `secrets` 里的密钥，不要用用户的账号。
- 要用户的用户名或邮箱时（比如拿用户名建系统账号），申请 `owner.identity`，不要为此声明 `sso: proxy`。
- 忘了密码按应用自己的办法找回。不删数据重装还是原来的账号，删了数据就要重新注册。

### OIDC

`sso: oidc` 的插件由核心的身份提供方（Ory Hydra）登录，用户不会看到第二个
登录页。清单要写明回调路径：

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `webui.oidc.redirect_paths` | 是 | 应用登录回调的路径（以 `/` 开头，相对插件网页地址），核心据此登记客户端的回调地址。 |
| `webui.oidc.post_logout_paths` | 否 | 额外的登出回跳路径，默认 `[]`；与 `redirect_paths` 一样每项以 `/` 开头。核心把路径拼在插件网页地址后，登记到 `post_logout_redirect_uris`。 |
| `webui.oidc.token_endpoint_auth_method` | 否 | 应用在令牌端点出示密钥的方式：`client_secret_basic`（默认）或 `client_secret_post`。 |
| `webui.oidc.owner_groups` | 否 | 用户是家庭的主人时，`groups` 在 `owner` 之外再加这些组名，按组名对角色的应用（如 BookStack）用它把主人对到管理员等角色。名字 1–64 个字符，只含 ASCII 字母、数字和 `_-.:`，不重复，最多 8 个。 |
| `webui.oidc.requires_email` | 否 | 应用用 `email` 声明认人或必填邮箱时写 `true`：用户没填邮箱就装不上，安装页让用户先填，不用等到登录才失败。 |

客户端可申请的 scope：`openid`、`profile`（`name`、`preferred_username`）、`email`（`email`、
`email_verified`）、`groups`（`owner`，声明了 `owner_groups` 再加那些组名）、`offline_access`。
`owner_groups` 只发给声明它的这个客户端，别的插件看不到。客户端 ID 就是插件 `id`，密钥在安装时
生成，卸载时删除客户端。

认人用 `sub`（`preferred_username` 同值），邮箱只当资料：应用有用户名概念就用 `sub` 或
`preferred_username` 找账号，不要拿 `email` 认人，邮箱会变。应用确实只有邮箱一种登录名时才用 `email`
认人，这时写 `requires_email`。
保留数据卸载后重装，密钥是新的：应用把密钥存进自己数据库的（如 Forgejo 的认证源），要在每次创建容器时用 `WONTON_OIDC_CLIENT_SECRET` 更新已有的那条，不能“已存在就跳过”。

按组名对角色的应用，声明 `owner_groups` 并让应用同步组：

```yaml
services:
  bookstack:
    image: ghcr.io/bookstackapp/bookstack:<版本标签或摘要>
    environment:
      OIDC_USER_TO_GROUPS: "true"
      OIDC_GROUPS_CLAIM: groups
x-wonton:
  webui:
    service: bookstack
    port: 8080
    sso: oidc
    oidc:
      redirect_paths: [/oidc/callback]
      owner_groups: [admin]
      requires_email: true
```

BookStack 把 `groups` 里的名字对到角色：先按角色显示名（大小写不敏感），再按角色设置里的“外部认证
ID”；内置管理员角色的显示名是 `Admin`，`admin` 就对得上。BookStack 还要求 `https` 的签发方地址
（本节末的选型项），核心现在的 `http` 签发方它不接受：环境变量名作对照用，换成同类应用时以各自文档
为准。应用没有组同步功能时不用 `owner_groups`，按它认角色的方式在文档里写清楚怎么给主人管理员。

应用不能自己注册管理员、要插件预建账号时，申请 `owner.identity`，按应用官方的引导机制用
`WONTON_OWNER_NAME` 和 `WONTON_OWNER_EMAIL` 建（多数应用有建管理员的官方环境变量），不要装完再改
应用的数据补邮箱：预建账号的值和 OIDC 登录给的 `sub`、`email` 声明一致，登录才对得上同一个账号。
声明了 `requires_email` 的插件，用户装之前就填了邮箱。

- 选型时先看应用接不接受 `http` 的 issuer：有的强制 `https`（如 BookStack），核心的 OIDC 是 `http`，
  这类应用登录走不通。
- `webui.path` 可以指向应用的 OIDC 起点：App 打开网页时把它拼在地址后面，核心给出的网页地址
  （`WONTON_APP_URL`、安装时的探测）不带它，省掉用户先点登录按钮。
- 核心始终登记插件根地址（有、无末尾 `/`）和 `webui.path` 作为登出回跳地址；`post_logout_paths` 是追加，不替换默认地址，重复项只登记一次。
  应用登出固定回 `/auth` 等其他路径时，应声明如 `post_logout_paths: [/auth]`。地址必须与应用发出的
  `post_logout_redirect_uri` 整串相同，包括尾斜杠和查询参数，不是路径前缀；请求的 `state` 参数由身份提供方在回跳时附加。
- 应用走身份提供方的 end-session 时，须发送 `id_token_hint`；有回跳地址或 `state` 而没有 hint 会被拒绝。
  没发 `post_logout_redirect_uri` 时回核心首页，不自动使用客户端白名单中的地址。
- 核心启动恢复已装插件、插件重新启动或升级时，会按保存的清单和原密钥重新登记客户端，不需要卸载重装。
  新增的应用内回跳路径仍需更新插件清单；核心不会猜测路径。含新字段的插件发布前，须先升级到支持它的核心。

### 网关代登录（`sso: proxy`）

网关认出用户后，在转发给插件的请求上加上清单声明的请求头或 Cookie。应用认反向代理传来的用户名时
（Navidrome、Paperless-ngx、Grafana 的 auth proxy 等）：

```yaml
webui:
  service: navidrome
  port: 4533
  sso: proxy
  proxy:
    headers:
      Remote-User: "{user}"
```

应用用自己的固定密钥登录时，比如 magpie 的网页认名为 `magpie_web_<端口>` 的 Cookie，值就是网页密钥：

```yaml
webui:
  service: magpie
  port: 3430
  sso: proxy
  proxy:
    cookies:
      magpie_web_3430: "{secret:web_key}"
```

| 字段 | 说明 |
| --- | --- |
| `webui.proxy.headers` | 请求头名 → 值模板。不能是 `Host`、`Cookie`、`Forwarded`、`X-Real-IP`、`X-Forwarded-*` 和逐跳头部。 |
| `webui.proxy.cookies` | Cookie 名 → 值模板。名字不能以 `wonton_` 开头。和浏览器发来的其他 Cookie 合并成一个 `Cookie` 头。 |

`sso: proxy` 至少写一项头或 Cookie。
值模板里可以用 `{user}`（用户名，同 OIDC 的 `preferred_username`）、`{name}`、`{email}`，以及
`{secret:<名字>}`，其余文字原样保留，如 `"Bearer {secret:api_token}"`；`{`、`}` 只能用来写占位符。用户没填邮箱时，
用了 `{email}` 的插件装不上。

应用认人用 `{user}`：它装好后不变。只有应用确实只能按邮箱认人、没有用户名概念时，才用 `{email}` 认人；
邮箱会变。

上游没有可信代理白名单（对所有来源都信任 `Remote-User` 这类头）时，申请了 `network.egress` 的插件
过不了 `plugin check --official`，这种应用改用 OIDC。

- `{secret:…}` 只能引用插件自己的 `secrets` 里由核心生成（`generate`）、至少 16 字节的一项：引用不到别的插件的
  密钥，也引用不到插件交给核心保管（`collect`）的秘密。网关按值从响应里找出并去掉它（见下），值太短会误伤。
- Cookie 的值只能用 `{secret:…}` 和固定文字，文字里不能有空白、引号、逗号、分号和反斜杠：用户名、显示名和邮箱
  可能有 Cookie 里不能出现的字符。请求头的值里不能有控制字符。不合规的清单在预览和安装时就被拒绝。
- 只有网关认出用户的请求才加凭据，规则和其他插件网页一样：局域网浏览器要有这个插件子域名的会话，经隧道的
  已配对设备看设备身份。没认出用户的请求不转发：打开页面跳到核心登录，其余请求 401。
- 插件子域名彼此同站，一个插件的页面能让浏览器带着会话向另一个插件发请求。所以网关按 Fetch Metadata
  的资源隔离策略，只认这些请求的登录状态：`Sec-Fetch-Site` 为 `same-origin` 或 `none`；顶层导航
  （`Sec-Fetch-Mode: navigate`，方法为 `GET` 或 `HEAD`）；没有 `Sec-Fetch-Site` 时，没有 `Origin`
  或 `Origin` 就是插件地址。其余请求按没登录处理，同样不转发。凭据只加在这个插件自己的子域名上。
- 网关先删掉客户端发来的同名请求头和 Cookie（名字不分大小写）再加上声明的值，客户端无法伪造或覆盖。
- 凭据不回到浏览器：插件响应里设置这些 Cookie 的 `Set-Cookie`（名字不分大小写，包括没写名字、浏览器会读成
  同名的）由网关去掉；值里带着 `{secret:…}` 原文的响应头（比如插件把密钥写回别的 Cookie 或跳转地址）也去掉。
  网关不改响应体，也不认识应用的接口。插件自己写的页面和服务不要把 `{secret:…}` 引用的值写进响应。应用自己的页面
  在用户登录后显示这个凭据（Syncthing 的设置页显示 API key，有的应用的配置接口原样返回配置），是应用本来的行为，
  不算违规：看得到的只有打得开这个网页的人。插件不为此改应用；`about` 里写一句哪里看得到它，用户自己判断。
- 用户的显示名这类值写不进请求头时（比如带换行），网关如实报错（500 `proxy_credentials`），不转发。
- 用请求头认用户的应用，“信任的代理地址”只写核心的地址，不写 `0.0.0.0/0` 和整个网段：
  出网插件的容器和接在 `--lan` 网络上的服务，宿主上的进程和经宿主代理转发的连接都到得了，别的插件能借此冒充用户。

  ```yaml
  environment:
    ND_REVERSEPROXYUSERHEADER: Remote-User
    ND_REVERSEPROXYWHITELIST: ${WONTON_CORE_ADDRESS}/32
  ```

  这条只管 `proxy.headers` 的模板用了 `{user}`、`{name}` 或 `{email}` 的插件。只信任核心，指 `webui.service` 的环境变量引用了 `${WONTON_CORE_ADDRESS}`，值里也没有 `0.0.0.0/0`、`::/0`。做不到时（应用没有这个选项，或者作者放宽了），按 `webui.service` 怎么开：

  | 插件 | `plugin check` | 核心 |
  | --- | --- | --- |
  | 出网 | `--official` 判不合格，改用 OIDC 或 `{secret:…}` | `webui.service` 的局域网端口同下一行（第三方插件） |
  | 不出网，开了局域网端口 | 提醒，不判不合格 | `webui.service` 的这些端口默认不开，服务也不接 `--lan` 网络。确认页和插件设置里有开关 `lan_webui`，默认关，旁边写明“局域网里的设备，以及这台主机上能出网的插件，不用密码就能以你的身份打开它”，用户打开后才开 |
  | 不出网，也不开局域网端口 | 在索引的审核里登记后通过：它的网络里只有核心和本插件的服务，宿主也连不上 | 不提示 |

  不走单点登录的局域网服务（比如 Ollama 的 API）不适用这一节的任何规则，用不用密钥由用户决定。
- `webui` 指向宿主网络（`network_mode: host`）的服务时不能用 `sso: proxy`：身份头只有从核心经插件网络转发的
  请求才可信（应用按核心在插件网络里的地址认它），而宿主网络的服务直接开在宿主的网卡上，本机的其他进程和
  局域网里的设备都能直连它，照样子伪造身份头冒充用户。这类应用用 `sso: oidc` 或 `sso: none`。
- 用 `{secret:…}` 的插件分不出是谁在用：每个能打开这个小程序的人都以同一个应用身份登录。

## 底栏页面（`tabs`）

有 `webui` 的插件可以声明能放到 App 底栏的页面。用户决定放哪些、什么顺序，插件和 Bot 都改不了。App 不支持这些字段时忽略它们，插件照常从“应用”打开。没声明 `tabs` 的插件也能放，入口是 `webui.path`，用插件自己的标题和图标。

```yaml
x-wonton:
  webui:
    service: web
    port: 8080
  tabs:
    - id: feed
      title: { zh: 动态, en: Feed }
      icon: tabs/feed.svg
      icon_selected: tabs/feed-fill.svg
      path: /feed
      layout: edge-to-edge
      display: standard
```

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `id` | 是 | 插件内唯一，由小写字母、数字和单个连字符组成。App 按它记住用户放了哪些页面，升级时不要改。 |
| `title` | 是 | 多语言，`zh` 和 `en` 都要写。建议 `zh` 不超过 4 个字，`en` 不超过 8 个字母，超了 `plugin check` 提醒。 |
| `icon` | 是 | 单色图标，插件包里的 PNG 或 SVG。只取透明通道，颜色由 App 上色，全透明或全不透明的图标不合格。SVG 的规则见“图标和截图”。 |
| `icon_selected` | 否 | 选中时的图标，规则相同。没写时选中只靠颜色区分。 |
| `path` | 是 | 入口路径，写法和 `webui.path` 相同（见“网页入口与单点登录”）。 |
| `layout` | 否 | `inset`（默认）：页面夹在 App 的栏中间。`edge-to-edge`：页面铺满，从栏下面滚过去，作者按 `--wonton-inset-*` 留位置（见“主题”）。 |
| `display` | 否 | `standard`（默认）或 `fullscreen`，见“全屏”。 |

- 一个插件最多 4 项。不申请新权限，页面能做的事不变。
- 插件停了、升级中、出错时，App 显示状态页，页面不用处理。升级后 App 重新加载已打开的页面。

### 全屏

带栏时页面在 App 的标题栏和底栏之间，不铺到底栏后面。全屏时 App 隐藏标题栏和底栏，右上角只留一个胶囊（“···”和退出全屏），状态栏竖屏照常显示。自带底栏的应用全屏后不会有两行底栏。

胶囊会自己藏起来：打开、加载完、转屏时出现，3 秒没碰就淡出；用户在页面上往下拉一下就出来，往上推就藏起。藏起来时不挡点按，点到的是页面。页面不用为它留位置，也隐藏不了它。

清单写 `display: fullscreen` 时打开就全屏：`tabs` 的每一项写在该项上，只有 `webui` 的写在 `webui` 上。默认 `standard`。用户能在 App 里按页面改，用户的选择优先于清单。

用户点胶囊的退出全屏：如果是手动进的全屏，回到带栏的样子，页面不重新加载；打开就全屏的页面回到进来之前的标签。

全屏时谁来躲开状态栏、Home 条（导航栏），按 `layout` 分：

| `layout` | 页面内容 |
| --- | --- |
| `inset` | 从状态栏下面开始，底边到 Home 条（导航栏）为止或画到它后面、由 App 让出这一块。页面什么都不用做，`env(safe-area-inset-*)` 是 0。状态栏那一条是 App 的底色 |
| `edge-to-edge` | 铺满屏幕。写 `viewport-fit=cover`，按 `--wonton-inset-top`、`--wonton-inset-bottom`、`--wonton-inset-left`、`--wonton-inset-right` 留位置 |

- 给 `html` 设背景色：页面之外露出来的地方（比如 Home 条后面）画的是它。
- 要画到状态栏底下，用 `edge-to-edge`，按 `--wonton-inset-top` 排自己的标题。全屏时它是状态栏下沿（横屏为 0），不含胶囊；别把它当成固定值，按变量算。
- 用 `--wonton-inset-*`，不要用 `env()`：一些 Android WebView 版本的 `env()` 值不可靠。
- 返回：iOS 从左边缘滑动，Android 按返回键，都是先退网页历史，到头才退出全屏。页面自己用横滑手势时，别从屏幕边缘开始。

### 主题

App 在文档开始前把下面的 CSS 变量写到根元素，同时设置 `color-scheme`。明暗和重点色变了不重新加载，变量当场更新。

| 变量 | 是什么 |
| --- | --- |
| `--color-background-primary`、`--color-background-secondary` | 页面背景、次一级的背景 |
| `--color-text-primary`、`--color-text-secondary` | 文字、次要文字 |
| `--color-border-primary` | 分隔线 |
| `--color-text-danger`、`--color-text-success`、`--color-text-warning`，`background`、`border`、`ring` 同理 | 状态色 |
| `--font-sans` | 系统字体 |
| `--border-radius-xs`、`--border-radius-md`、`--border-radius-lg`、`--border-radius-full` | 圆角 |
| `--wonton-accent-fill`、`--wonton-accent-ink`、`--wonton-accent-ink-on-fill` | 重点色的底色、文字和图标、放在重点色上的文字 |
| `--wonton-inset-top`、`--wonton-inset-bottom` | 栏占的高度，含安全区；全屏时见“全屏”。`inset` 布局下是 0 |
| `--wonton-inset-left`、`--wonton-inset-right` | 左右安全区，横屏时躲刘海和圆角。`inset` 布局下是 0 |

变量名只增不改。页面也可能在浏览器里打开，所以写上后备值：

```css
:root { color-scheme: light dark; }
body {
  margin: 0;
  background: var(--color-background-primary, Canvas);
  color: var(--color-text-primary, CanvasText);
  font-family: var(--font-sans, system-ui, sans-serif);
  padding-top: var(--wonton-inset-top, 0);
  padding-bottom: var(--wonton-inset-bottom, 0);
}
```

### 和 App 通信

只读 CSS 变量的页面不用通信。要通信时用 [MCP Apps](https://modelcontextprotocol.io/docs/extensions/apps) 的方法名，可以直接用官方 SDK `@modelcontextprotocol/ext-apps` 的 `App` 类。只收插件自己主框架发的消息：

| 方法 | 说明 |
| --- | --- |
| `ui/initialize`、`ui/notifications/initialized` | 握手。`hostCapabilities` 是 `openLinks` 和 `message.text`。`hostContext` 有 `theme`、`styles`、`locale`（App 在用的语言，如 `zh-Hant`；页面自己的 `navigator.language` 是用户要的语言）、`timeZone`、`platform: mobile`、`displayMode`（带栏 `inline`，全屏 `fullscreen`）、`availableDisplayModes`、`safeAreaInsets`。App 收到握手或页面加载完就撤掉骨架屏，先到先算 |
| `ui/notifications/host-context-changed` | 明暗、重点色、字号、全屏状态变了 |
| `ui/request-display-mode` | `mode` 是 `inline` 或 `fullscreen`，回实际生效的模式。用户把这个页面设成不全屏时不生效 |
| `ui/open-link` | 系统浏览器打开，只收 `https` |
| `ui/message` | `role: user`，`content` 只收文字块。切到聊天，把文字放进输入框，用户点发送才发 |

其余方法回 JSON-RPC 错误 `-32601`，`tools/call` 和 `resources/read` 没有：页面用自己的后端。

桥只给主题、语言、时区和栏高，不给用户身份和设备信息。要用户的身份用 `owner.identity`。

`window` 上另有两个事件，用 `addEventListener` 收，不走 SDK：

| 事件 | 什么时候 |
| --- | --- |
| `wonton:tab-reselected` | 用户再点一次当前标签。App 会把文档滚回顶部，页面有内层滚动时自己处理 |
| `wonton:connection` | 手机和家里服务器的连接变了，`detail.connected` 是布尔值。`navigator.onLine` 看的是手机联网，不是服务器 |

### 跨源隔离

页面要用 `SharedArrayBuffer` 这类要跨源隔离的能力时，在 `webui` 上写 `cross_origin_isolated: true`。默认 `false`，用不到别写。

```yaml
x-wonton:
  webui:
    service: web
    port: 8080
    cross_origin_isolated: true
```

- 网关给这个插件的网页响应加 `Document-Isolation-Policy: isolate-and-credentialless`，应用自己带的同名头会被替换。网关自己的错误页不加。
- 应用仍要自己发 `Cross-Origin-Opener-Policy: same-origin` 和 `Cross-Origin-Embedder-Policy: require-corp`（或 `credentialless`）：iOS 和 Safari 只看这两个头。
- Android 的 App 在 WebView 156 及以上为声明了的插件打开隔离，没声明的插件不受影响。更旧的 WebView 给不了，App 打开前提示用户，用户可以选“仍然打开”，这时页面没有 `SharedArrayBuffer`。应用最好有不用它的降级。
- 这一项放宽了 Android 上的安全保障：页面可能和别的网页共用渲染进程。插件详情页会写明；Agent 写的插件，安装和更新的确认卡片上也会写明，更新时新加这一项要主人确认。

## 权限

| 权限 | 风险级别 | 含义 |
| --- | --- | --- |
| `storage.share.<名称>:read` / `:write` | — | 读或写某个共享文件夹，如 `photos`、`documents`、`music`、`downloads`。设备上传的文件在 `transfers` 里。 |
| `storage.home:read` / `:write` | — | 读或写当前用户的私人空间。 |
| `models` | — | 有模型网关就用它：没接上时照样安装，见“模型网关”。 |
| `tools` | — | 经核心的 MCP 总线调用核心和其他插件的工具，见“工具与 MCP 总线”。 |
| `plugins.health` | — | 读所有插件的健康状态，见“插件健康探测”。 |
| `matrix.bot` | — | 以 Bot 身份收发消息，见“Matrix Bot”。 |
| `notify` | — | 给用户发通知，见“通知”。 |
| `owner.identity` | — | 知道用户的用户名和邮箱，见“安装时注入的变量”。 |
| `network.egress` | — | 能访问外网，也能访问用户的局域网，安装确认按这句写。见“网络”。 |
| `network.lan` | 高 | 在局域网上开放 `lan.ports` 列出的端口，见“局域网”。 |
| `egress.proxy` | 高 | 做出网代理：选它以后，所有出网插件的请求都经过它，它能看到这些插件访问了哪些网站，也能改写不加密的内容。和 `provides.egress_proxy` 成对，见“网络”的“代理插件”。 |
| `devices.gpu` | 高 | 使用显卡，见“显卡”。 |
| `devices.<路径>` | 高 | 访问宿主设备，路径只能是 `/dev/net/tun` 或 `/dev/fuse`，如 `devices./dev/net/tun`。 |
| `network.host` | 很高 | 使用宿主网络：能访问本机所有端口，开的端口直接开在本机的网卡上，开在哪些网卡上由应用决定。 |
| `cap.<能力>` | 很高或最高 | 给服务增加 Linux capability，如 `cap.NET_ADMIN`。`NET_ADMIN`、`NET_RAW`、`NET_BIND_SERVICE`、`SYS_TIME`、`SYS_NICE`、`IPC_LOCK` 是很高，其余都是最高。 |
| `host.pid`、`host.ipc` | 最高 | 共享宿主的进程空间或 IPC。 |
| `security.opt` | 最高 | 修改容器的安全选项。 |
| `privileged` | 最高 | 以 Docker 特权模式运行服务。 |
| `docker.socket` | 最高 | 访问 Docker socket，等于把主机的 root 交给插件。 |

- 高权限和对应的 Compose 字段必须成对出现（见“Compose 子集”）：只写一边时清单不合法，错误写明缺少哪一项。插件不声明的能力一律不给。
- 风险级别和给用户看的提示由核心决定，安装确认时醒目展示。插件改不了。
- `kind: agent`，或者声明了 `matrix.bot`、`bots` 的插件，不能申请很高和最高级权限。
- 安装请求必须逐项确认清单里的全部权限，少一项或多一项都拒绝安装。

核心兑现上表除了 `storage.home:*` 的全部权限。`storage.home:*` 清单里可以声明，但安装时如实报“尚未支持”。共享文件夹在数据目录的 `shares/<名称>` 下，只授权读时必须以 `:ro` 挂载，而且只能整个挂载，不能挂其中的子路径。

## 安装时注入的变量

| 变量 | 何时注入 | 说明 |
| --- | --- | --- |
| `WONTON_APP_ID` | 总是 | 插件 `id`。 |
| `WONTON_APP_DATA` | 总是 | 插件私有数据目录（宿主路径）。绑定挂载的子目录由核心逐级预先创建，属主见下文；路径上有软链接或不是目录时拒绝安装。 |
| `WONTON_SERVICE_DOMAIN` | 总是 | 这个插件在插件网络里的域名 `<slug>.wonton.internal`，如 `app-immich-photos.wonton.internal`。服务的稳定地址名是 `<服务名>.${WONTON_SERVICE_DOMAIN}`：核心连插件的网页、MCP 和内网资源都用它当 Host，上游服务做精确 Host 校验（如 Playwright MCP 的 `--allowed-hosts`）时把它写进白名单，见“网络”。 |
| `WONTON_HOST_NAME` | 总是 | 宿主主机名（核心从 Docker daemon 的 `/info` 取，如 `server-01`），可填服务的 `hostname:` 或应用里的主机设置。它是用户的主机名，可能是不想公开的信息，截图里要遮住。取不到时是空字符串。 |
| `WONTON_SHARE_<名称>` | 授权了该共享文件夹 | 共享文件夹路径（大写，`-` 换成 `_`，如 `WONTON_SHARE_PHOTOS`）。 |
| `TZ`、`PUID`、`PGID` | 总是 | 时区与运行用户。`TZ` 是 IANA 时区名，如 `Asia/Shanghai`、`UTC`。 |
| `WONTON_APP_URL` | 声明了 `webui` | 插件网页的公开地址，不带末尾 `/`。应用生成回调地址、外部链接时使用。 |
| `WONTON_OIDC_ISSUER`、`WONTON_OIDC_CLIENT_ID`、`WONTON_OIDC_CLIENT_SECRET` | `webui.sso: oidc` | OIDC 客户端配置。 |
| `WONTON_OIDC_AUTH_URL`、`WONTON_OIDC_TOKEN_URL`、`WONTON_OIDC_USERINFO_URL` | `webui.sso: oidc` | 各端点地址，给不支持发现文档的应用（如 Grafana）。 |
| `WONTON_CORE_ADDRESS` | `webui.sso: proxy` | 核心在这个插件网络里的地址（网段的最后一个可用地址，如 `172.18.255.254`），核心容器重建后不变。写进应用的“信任的代理地址”，见“网关代登录”。 |
| `WONTON_OWNER_NAME` | `owner.identity` | 用户的用户名，装好后不变，和 OIDC 的 `sub`、`preferred_username` 以及代登录模板的 `{user}` 同值，见下文。 |
| `WONTON_OWNER_EMAIL` | 同上，且用户填了邮箱 | 用户的邮箱，和 OIDC 的 `email`、代登录模板的 `{email}` 同值。值是建容器时的快照，见下文。 |
| `WONTON_MODEL_BASE_URL` | `models` | 模型网关 OpenAI 兼容接口的地址，以 `/v1` 结尾，如 `http://models.wonton.internal:8080/v1`。没接上网关时是空字符串。 |
| `WONTON_MODEL_ANTHROPIC_BASE_URL`、`WONTON_MODEL_GEMINI_BASE_URL` | `models`，且网关提供该接口 | Anthropic Messages、Gemini 接口的地址，按各自 SDK 的约定不带 `/v1`。没接上网关时是空字符串。 |
| `WONTON_MODEL_API_KEY` | `models` | 只属于本插件（多 Bot 时只属于这个 Bot）的网关凭据。没接上网关时是空字符串。 |
| `WONTON_MODEL_DEFAULT` | `models` | 默认模型名，由网关决定（magpie 是路由组 `group/wonton-default`）。没接上网关时是空字符串。 |
| `WONTON_MCP_URL` | `tools` | MCP 总线的地址（Streamable HTTP）：核心在插件网络里的别名，如 `http://mcp.wonton.internal:8080/mcp`。 |
| `WONTON_NOTIFY_URL` | `notify` | 发通知的地址：核心在插件网络里的别名，如 `http://notify.wonton.internal:8080/notices`，见“通知”。 |
| `WONTON_MATRIX_HOMESERVER` | `matrix.bot` | Matrix 客户端接口的地址：核心在插件网络里的别名，如 `http://matrix.wonton.internal:8080`。 |
| `WONTON_MATRIX_USER_ID`、`WONTON_MATRIX_ACCESS_TOKEN` | `matrix.bot` | Bot 的 Matrix 用户 ID 和访问令牌。 |
| `WONTON_MATRIX_OWNER` | `matrix.bot` | Bot 用户的 Matrix 用户 ID，用作白名单：私人 Bot 只接受用户的指令。 |
| `WONTON_MATRIX_HOME_ROOM` | `matrix.bot` | Bot 的主房间：核心替用户建好的、和 Bot 的加密私聊，定时任务的结果发到这里。 |
| `WONTON_SECRET_<名字>` | `secrets` 里由核心生成的项 | 插件密钥，名字转成大写，如 `WONTON_SECRET_WEB_KEY`。 |
| `WONTON_BOT_NAME`、`WONTON_BOT_DATA` | `bots.services` 里的服务 | Bot 名和这个 Bot 的数据目录（宿主路径）。 |
| `WONTON_LAN_ADDRESSES` | `network.lan` | 局域网端口开在哪些地址上，逗号分隔，如 `192.0.2.10`。地址变化时的重建规则见“局域网”。 |
| `WONTON_PROXY_URL` | `network.egress` | 出口容器的地址 `http://egress.wonton.internal:3128`，用户没配代理时是空串。给有自己代理设置、不读 `HTTP_PROXY` 的程序用，见“网络”。 |

- 这些变量只用于 compose 插值（`${VAR}`、`${VAR:-默认值}` 等，`$$` 表示字面量 `$`），不会自动进入
  容器环境；需要的变量在对应服务的 `environment` 里显式引用，密钥只给用得到它的容器。
- 引用了没有注入的变量且没有默认值时，安装失败，而不是像 docker compose 那样替换成空串。例如没有声明
  `sso: oidc` 却引用 `WONTON_OIDC_CLIENT_SECRET`，或者模型网关不提供 Anthropic 接口却引用
  `WONTON_MODEL_ANTHROPIC_BASE_URL`。核心不把自己的环境变量传给插件。
- 地址和令牌以注入值为准，插件不要自己拼：主机名、端口和格式都可能变。
- 用户的用户名和邮箱同样以注入值为准，不要写死或按规则拼。没申请 `owner.identity` 的插件拿不到它们，安装不多给一份
  个人信息，引用了就装不上，错误写明“插件没申请 `owner.identity`”。应用登录时从 OIDC 或代登录模板拿到的身份不归这项权限管，
- 用户名是固定 ID，装好后不变，插件不用处理它变化的情况。它只含小写字母、数字、`_` 和 `-`，以字母开头，最长 32 个
  字符，不是常见的系统用户名，可以直接用来建系统账号；镜像里自带的用户（`postgres`、`node`）可能重名，要插件自己
  处理。应用认人一律用 OIDC 的 `sub`（和 `preferred_username` 同值）或代登录的 `{user}`，不要按邮箱认：
  **邮箱只是资料，会变**。应用确实只有邮箱一种登录名、没有用户名概念时，才用 `email` 声明或 `{email}`。
- 显示名和邮箱由用户在 App 里填，填过以后能改，不能改回没有。显示名没填时是用户名，所以 `name` 声明总有值；要显示
  用户的名字用 `name`，不要用 `preferred_username`。
- 用户没填邮箱时不注入 `WONTON_OWNER_EMAIL`，OIDC 也没有 `email` 声明。应用非要邮箱时写 `${WONTON_OWNER_EMAIL:?}`，
  App 的安装页这时让用户先填邮箱，填好才能装：OIDC 登录要 `email` 声明的应用声明
  `webui.oidc.requires_email`，不要等到登录时才失败。
  能没有邮箱的写 `${WONTON_OWNER_EMAIL:-}`。
- 用户改了邮箱或显示名之后，OIDC 声明和代登录的请求头马上换成新值，容器里的 `WONTON_OWNER_EMAIL`
  还是旧值：核心重启只核对插件容器，不重建。容器重建时才拿到新值；升级或重装（可以不删数据）会重建，局域网地址变化
  按“局域网”的规则只重建受影响的服务，未重建的服务保留旧值。应用里已有账号的资料由用户在应用里改，插件不写对账代码。
- 核心不注入 `WONTON_OWNER_ID`：OIDC 的 `sub` 就是用户的用户名，另起一个变量只是同一个值。

**数据目录属主**：核心预先创建的绑定挂载目录（`WONTON_APP_DATA` 下）交给谁，看挂载它的服务的
`user`：

- 没写 `user`，或 `user` 是 root（`root`、`0`、`0:0`）：交给 `PUID:PGID`。以 root 启动、再按
  `PUID`/`PGID` 降权的镜像（如 LinuxServer.io 的镜像）就是这种情况。
- `user` 是数字形式的 `uid:gid`：交给这个 uid:gid。镜像固定以某个 uid 运行、不认 `PUID`/`PGID` 时
  （如 magpie 的 65532），就写上它：`user: "65532:65532"`。
- `user` 是 root 以外的用户名，或只有 uid：安装失败，提示改成 `uid:gid`。几个服务以不同的属主挂载同一个
  目录时也安装失败。

应用数据目录本身和路径上的中间各级目录交给 `PUID:PGID`。核心只改这些目录本身的属主，不递归改里面已有的
文件。

能以任意 uid 运行的镜像优先写 `user: "${PUID}:${PGID}"`，宿主上的文件就归用户的用户。这里不另设清单
字段：compose 的 `user` 已经说明了容器以谁运行。

## 预装插件

核心镜像里带着两类插件包，核心启动后在后台装上，授予清单声明的全部权限：装核心就是用户对它们的确认。目录里
每个子目录一个插件包，一个装完再装下一个；先装系统插件，再装默认插件（默认 Agent 要用模型网关）。

| | 目录 | 什么时候装 | 用户卸掉以后 |
| --- | --- | --- | --- |
| 系统插件（`kind: system`，如模型网关） | `WONTON_SYSTEM_PLUGINS` | 每次启动，没装就装 | 下次启动装回来 |
| 默认插件（如默认 Agent） | `WONTON_DEFAULT_PLUGINS` | 只装一次 | 不再自动装 |

- 系统插件只能这样装，API 装不了。默认插件是普通插件（不能是 `kind: system`），用户可以照常卸载、重装。
- 已经装着的照常核对，不重装。系统插件目录里的版本比装着的新时，等核心自己的升级结束后，按“升级”的步骤升级它，装核心就是对它
  的确认。默认插件只在第一次从目录装，之后和别的官方插件一样由官方索引提示升级、用户确认，核心不随镜像升级它。
- 默认插件装好、或者用户卸掉它以后，核心在自己的数据目录里记下（`core/plugins/preinstalled/<id>.json`），此后不再
  自动装：用户换掉默认 Agent，核心不会把它装回来。
- 装不上时如实标为异常、写明原因，不影响核心启动：
  - 插件包本身的问题（`invalid_input`、`unsupported`，如不支持本机架构）、本机没有可用的显卡（`gpu_unavailable`）和
    同名的网络或卷不是这个插件的（`conflict`）重试也一样，不重试，下次启动再试；
  - 其余原因（Docker、身份提供方、Matrix，核心处理图片的子进程起不来，拉镜像失败，没通过健康检查）
    自动重试，间隔逐次加长，最长 10 分钟；等待时原因里写着试了几次、会自动重试。拉不下镜像时原因写成“拉取镜像失败
    （镜像）：Docker 的原话”（错误类别 `image_pull_failed`）；
  - 等待重试时用户卸掉这个异常的插件，就不再重试；默认插件同样记下，不再自动装。卸载开始了就算，到点时卸载还没
    做完也不再重试。
- 局域网状态页逐个列出这些插件里没在运行的（安装中，或者异常和原因的第一行）：它们每台主机都有，列出来不泄露用户
  装了什么。原因后面的容器日志可能有聊天内容，只在用户登录后的插件列表里给。其他插件只给个数。

## 模型网关

模型请求只经模型网关（默认 magpie）。网关是一个声明了 `provides.models` 的系统插件（见“预装插件”）；
核心把声明了 `models` 的插件接到它上面，给每个插件单独的凭据。

### 使用方

- 地址指向核心在插件网络里的别名 `models.wonton.internal`，由核心转发给网关（见“网络”），如
  `http://models.wonton.internal:8080/v1`。插件和网关不在同一个网络里，只能经这个别名访问；网关按来源地址认出
  请求来自哪个插件网络，没申请 `models` 的插件得到 403 `permission_denied`，网关没在运行时得到 503
  `models_unavailable`。请求和响应都流式转发，不缓冲，也不设总超时。主机名一定带点：
  Hermes 把不带点的主机名当成本机模型服务，会去探测 Ollama 接口，而 magpie 正好兼容这组接口，结果每个
  请求都被加上 Ollama 专用的参数。
- 请求 `WONTON_MODEL_DEFAULT`，不写死 provider 的模型名：换 provider 或模型只在网关里改。默认模型还没
  配好时，签发照常成功；安装预览给出不阻断安装的提示。使用方的请求仍得到网关自己的错误，插件如实显示，
  不换用别的模型，核心不拦截或改写这些错误。
- 凭据默认可以用网关里的所有模型：插件和网关在同一台机器上，能访问就该能用。凭据由核心自动签发和注入，
  插件不用配置，作用是按插件记账和单独吊销。要限定某个插件能用的模型、设额度或限定账号，用户在网关自己的
  网页里改；核心不同步这些设置。
- **网关可选。** `models` 的意思是“有网关就用”：没装、没就绪、或核心签发凭据失败，都照样安装，
  `WONTON_MODEL_*` 照样定义、值为空字符串，用户用自己的登录或 key 设置（详情页有“没接上”的轻提示）。
  不加“必需”档。插件在启动时按变量有没有值选路：有值走网关，没值走自己的设置。
- 网关之后就绪（装上、起来、恢复）时，核心给没接上的使用方签发凭据并重建它们的容器，插件的启动选路在下一次
  启动时生效；网关卸掉或坏了，使用方不动，提示变回“没接上”。
- 安装预览只请求 `/models`，不发送可能计费的对话请求。`kind: agent` 即使没有申请 `models` 也给出默认路由提示，
  但不影响安装和状态：使用订阅账号的 ACP Agent 可能不经过模型网关。
- 接上网关之后，核心按默认路由的探测结果给详情页的轻提示（不是错误，状态照常，见“凭据的签发与吊销”的记法）：
  约 30 秒一次的巡检只读 `GET <apis.openai>/models`，所有运行中的使用方共用一次结果；默认模型不在 `data` 里
  就给 `models_no_default_model`。模型已列出时，核心在核心启动首轮、使用方刚安装、刚启动时各发一次最小对话
  请求（一条用户消息、`max_tokens: 1`、非流式；凭据是核心自己的 `<网关插件 id>/health`，不记在使用方名下），
  被服务商拒绝或失败就给 `models_provider_error`，原因由核心按 App 的语言写；服务商出错后每 5 分钟复查，
  恢复即停。健康稳态不发补全：每 30 秒一轮补全是每天约 2880 次请求，会把健康检查混进用量。网关连不上或
  模型列表拿不到归 `models_not_connected`。

> **已定：凭据默认可以用所有模型**。没选的方案是只允许默认模型：最小权限、花费
> 可控，但需要特定模型（如视觉模型）的插件要用户去网关里放开。

### 提供方（`provides.models`）

```yaml
x-wonton:
  kind: system
  provides:
    models:
      service: magpie
      port: 3425
      apis:                           # 接口 → 基础路径
        openai: /v1
        anthropic: /
        gemini: /
      default_model: group/wonton-default
      credentials:
        issue: [sh, /wonton/credentials.sh, issue, "{name}"]
        revoke: [sh, /wonton/credentials.sh, revoke, "{name}"]
      providers:
        list: [sh, /wonton/providers.sh, list]
        save: [sh, /wonton/providers.sh, save, "{id}"]
        test: [sh, /wonton/providers.sh, test, "{id}"]
        custom:
          save: [sh, /wonton/providers.sh, custom-save]
          test: [sh, /wonton/providers.sh, custom-test]
```

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `service`、`port` | 是 | 网关监听的服务和端口。核心只转发到这个端口，网关容器的其他端口（如网页）使用方访问不到。 |
| `apis` | 是 | 提供的接口及其基础路径。必须有 `openai`，可选 `anthropic`、`gemini`；决定注入哪些地址变量，注入的地址都不带末尾的 `/`。 |
| `default_model` | 是 | 注入给使用方的 `WONTON_MODEL_DEFAULT`。默认路由有成员时，网关必须在标准 OpenAI `GET <apis.openai>/models` 的 `data` 中以 `id` 列出它；没有成员时不列出。核心只按这个声明检查，不认识网关内部的模型配置。 |
| `credentials.issue`、`credentials.revoke` | 是 | 签发和吊销凭据的命令（argv，不经 shell），见下节。 |
| `credentials.service` | 否 | 在哪个服务的容器里执行命令，默认是 `service`。 |
| `providers.list`、`providers.save`、`providers.test` | 声明 `providers` 时必填 | 列出、保存和只测试模型服务商的命令（非空 argv，不经 shell），三个要一起写，`save` 和 `test` 都要用 `{id}`。见“模型服务商”。没声明 `providers` 时，核心只给 App 网关的网页地址，用户在网页里配置。 |
| `providers.custom` | 否 | 自定义模型服务商的 `save`、`test` 命令（非空 argv，不经 shell），两个要一起写，且都不能含占位符。见“模型服务商”。 |

命令都在 `credentials.service`（默认 `service`）的容器里执行，以容器默认用户运行。核心不经 shell 拼接或解释 argv。

同一台服务器同时只能有一个插件提供 `models`，装第二个时安装失败（`conflict`）。提供方看得到全部模型请求和
provider 的 key，所以只有 `kind: system` 的官方插件可以声明。

`/models` 和 `providers.list.configured` 只说明模型能力和已配置服务商，不证明 key 仍有效。最小补全会计费，
不能每轮巡检发送（30 秒一次约每天 2880 次）。仅在核心启动后首轮、使用方刚安装或刚启动时各检查一次；
服务商错误后每 5 分钟复查，恢复即停止。安装已经完成首轮检查时不重复请求；缺模型以及缺模型后首次列出默认组时不发补全。
请求发到 `<apis.openai>/chat/completions`：`model` 为 `default_model`，一条用户消息，`max_tokens: 1`、
`stream: false`。预览只查模型列表，不发补全。不新增服务商专用命令，健康检查凭据沿用 `credentials.issue`、
`credentials.revoke` 合同，与使用方凭据和用量分开。

### 凭据的签发与吊销

参照 Docker 凭据助手（credential helper）的约定：核心在网关的容器里执行声明的命令，秘密值走标准输入，
以退出码判断结果，不解析输出。

- 占位符：`{name}` 是凭据名。没有 `bots` 时只有插件级的凭据，名字是插件 `id`；有 `bots` 时 Bot 服务用这个 Bot 的
  凭据（`<插件 id>/<Bot 名>`，默认 Bot 是 `default`），还有全插件一份的服务时再签发一个插件级的。核心逐项替换 argv，
  不经 shell，在 `credentials.service`（默认 `service`）的容器里以容器的默认用户执行。
  默认路由探测另用 `<网关插件 id>/health`，不注入使用方；该凭据的用量独立记录，网关重装后按同一值重新签发。
- `issue`：标准输入给出凭据的值（一行）。网关里没有这个名字的凭据时，新建一个，不限定模型；
  已有时不改动它（用户可能调整过）。退出码 0 表示成功。
- `revoke`：删除这个名字的凭据；本来就没有也算成功。
- 凭据的值由核心生成（32 字节随机数的十六进制）并存在密钥库。网关的数据丢了或重装了，核心用同一个值
  重新签发，使用方不用重启。
- 时机：
  - 安装使用方时签发，再用这个凭据请求一次 `GET <openai 基础地址>/models`，返回 200 才算成功。返回 401
    说明网关里的同名凭据不是这个值，核心先 `revoke` 再 `issue` 一次；还是不行就报错。
  - 网关每次启动后（安装、升级、核心重启后的恢复），核心按同样的步骤为所有使用方对一次账；对账失败的使用方
    标为异常、写明原因。
  - 卸载使用方（删不删数据都一样）或删掉一个 Bot 时吊销，再从密钥库删除；重装是一个新值。网关已经卸载了就没有
    可吊销的；网关装着但没在运行时吊销不了，卸载如实失败。
- 命令超时 30 秒。失败时错误里附上命令的标准错误输出，并区分“网关未就绪”和“网关拒绝”。

### 模型服务商

用户在 App 里选服务商、贴 key。核心不认识网关，也不存 key，只在网关的容器里执行声明的命令。

`list` 超时 30 秒，退出码 0，在标准输出写一个 JSON：

```json
{
  "presets": [{ "id": "deepseek", "title": "DeepSeek" }],
  "configured": ["deepseek"]
}
```

- `presets` 是网关能配的服务商。`id` 是服务商的标识，官方索引的推荐项按它对应；`title` 是厂商的名字，不分语言。
- `configured` 是已经配好的服务商。App 只显示 `presets` 里的项，所以配好的服务商也要出现在 `presets` 里。
- 退出码不是 0 或者输出不是这个 JSON 时，错误是 `models_unavailable`，错误里带标准错误输出。

`save` 和 `test` 超时 60 秒。`{id}` 必须在 `list` 的 `presets` 里，否则核心不执行，直接报 `invalid_input`。
标准输入都是 key 加一个换行；key 不能为空、不超过 4 KiB、不含回车或换行，不合格时核心不执行命令。
已有服务商时 `save` 换 key，没有时新建，然后测试一次；`test` 只测试，成功或失败都不改变已有配置，也不留下临时配置。
key 只经标准输入传递，不进入 argv、日志、审计或错误信息。按退出码判断结果，不解析输出：

| 退出码 | 含义 | 核心的错误 |
| --- | --- | --- |
| 0 | 操作成功（`save` 保存并测试通过，`test` 测试通过） | |
| 3 | 服务商拒绝了 key | `invalid_input`；`save` 不保存，`test` 不改变配置 |
| 4 | 连不上服务商 | `upstream_unavailable`；`save` 的说明是“key 已保存”，`test` 的说明是“什么都没保存” |
| 其他 | 网关自己出错 | `models_unavailable`，错误里只有退出码 |

可选的 `custom.save`、`custom.test` 用来接 OpenAI 兼容的自定义服务商，超时 60 秒，不使用占位符。
标准输入是一行紧凑 JSON 加一个换行，例如：

```json
{"base_url":"http://192.0.2.10:8080/v1","model":"example-model","key":"example-key"}
```

`custom.save` 用这些参数保存服务商并测试；名字由网关定，保存后出现在 `list` 的 `configured` 和 `presets` 里，
使用方请求 `WONTON_MODEL_DEFAULT` 时应能用到这个模型。`custom.test` 只测试，不改变已有配置，也不留下临时配置。
退出码和保存语义同上表，另有退出码 2 表示地址或模型名不合法（`invalid_input`）。连不上时，保存操作说明“key 已保存”，
测试操作说明“什么都没保存”。秘密输入和错误信息的限制与 `save`、`test` 相同。

核心接口：

| 接口 | 说明 |
| --- | --- |
| `GET /_wonton/api/models/providers` | 返回按官方推荐清单筛选的服务商、拿 key 的地址、网关已配置的 `configured`（包括自定义服务商）和网页地址；`custom` 表示网关是否声明了 `providers.custom`。这不是命令 `list` 的原始 JSON。 |
| `PUT /_wonton/api/models/providers/{id}` | 请求体 `{"key":…}`，转给 `save`。 |
| `POST /_wonton/api/models/providers/{id}/test` | 请求体 `{"key":…}`，转给 `test`，什么都不保存。 |
| `PUT /_wonton/api/models/custom-provider` | 请求体包含 `base_url`、`model`、`key`，转给 `custom.save`。 |
| `POST /_wonton/api/models/custom-provider/test` | 请求体同上，转给 `custom.test`，什么都不保存。 |

保存和测试成功（退出码 0）时返回 HTTP 204，没有响应体。鉴权与 `PUT /_wonton/api/models/providers/{id}` 相同，
这些写接口的请求体上限都是 8 KiB。核心先校验自定义输入：`base_url` 不超过 2048 字节、没有 Unicode 空白，
且能解析为 `http` 或 `https` URL；`model` 非空、不超过 200 字节、没有控制字符和 Unicode 空白；
key 非空、不超过 4096 字节、没有 CR 或 LF。不合格是 HTTP 400 `invalid_input`，不执行命令。
key 被拒（退出码 3）以及自定义地址或模型名不合法（退出码 2）是 HTTP 400 `invalid_input`；
连不上服务商（退出码 4）是 HTTP 502 `upstream_unavailable`；其他命令失败是 HTTP 503 `models_unavailable`。
网关未就绪时也报 HTTP 503 `models_unavailable`。未声明 `custom` 时，列表的 `custom` 为 `false`，
自定义操作报 HTTP 503 `models_unavailable`。核心不保存 key。

## Matrix Bot

Bot 是一个 Matrix 身份加上驱动它的插件。核心没有接入 Matrix 时，声明了 `matrix.bot` 的插件
安装失败（`matrix_unavailable`）。安装时核心：

1. 分配 Bot 的用户名：插件 `id` 的最后一段（`agent.echo` 是 `echo`）。这个名字的账号已经存在（家人、别的插件的
   Bot、停用的账号），或者和用户、核心的管理账号重名时，依次换成 `<名字>-2`、`<名字>-3`……JWT 登录对已有的账号
   是直接登录进去，所以核心不登录任何已有的账号，只有这个插件记录在案的 Bot 账号能沿用。插件以
   `WONTON_MATRIX_USER_ID` 为准。
2. 用 JWT 代 Bot 登录一台新设备，首次登录即建号；不申请刷新令牌。Tuwunel 的 `access_token_ttl`
   只作用于申请了刷新令牌的客户端，这样拿到的访问令牌不会过期，而 Agent 多半不会自己刷新令牌。插件只拿到
   令牌，Bot 账号也没有密码。
3. 按 `bot` 设置显示名（取中文，没有时取英文；没写 `bot.display_name` 时用 `title`）和静态头像（png、jpg、gif 或
   webp）。只在新建账号时设置：沿用的账号（不删数据重装、升级、安装失败后重试）不重设，用户或 Agent 后来改过的
   名字和头像不会被冲掉。见下面的“名字和头像”。
4. 准备 Bot 的主房间：Bot 建一个加密房间（`trusted_private_chat`，用户和 Bot 都是管理员），先不邀请用户。没有主房间
   时 Hermes 会在每个新会话的第一条回复里插一段设置提示。
5. 注入 `WONTON_MATRIX_*` 变量。
6. 插件装好、健康检查都通过后，把主房间交给用户：核心用 JWT 临时登录用户的一台设备（用户的账号在第一次登录时才建，
   所以先登录再邀请），Bot 邀请用户，用户加入，双方的 `m.direct` 都记着它，然后退出登录。私聊这时才出现在用户的
   聊天列表里：早交出去的话，用户在 Bot 启动前发的话它收不到。装不上时用户那边不留私聊；交出之后才失败的（如写
   安装记录失败），回滚时用户和 Bot 都退出这个房间。

- `WONTON_MATRIX_HOMESERVER` 是 `http://matrix.wonton.internal:<网关端口>`，核心在声明了 `matrix.bot` 的插件网络里的
  别名。网关按连接的来源地址认出请求来自哪个插件网络（不算网桥自己的地址，那是宿主那一侧），只转发声明了
  `matrix.bot` 的插件的请求，而且只转发 Matrix 客户端接口（`/_matrix/client/`、`/_matrix/media/`），管理接口和
  联邦接口不转；其他来源得到 403 `M_FORBIDDEN`。插件不和 Tuwunel 共用网络。
- 账号和主房间建好后一直沿用：不删数据卸载后重装、安装失败后重试，都是同一个账号。Bot 还在主房间里，并且用户也在、
  或者还没交给过用户（上次安装没装完）时沿用它；用户退出过就新建一个。设备每次安装新建，安装失败回滚时吊销。
- 插件的容器启动时，第一次安装的用户还不在主房间里。要往主房间里发消息的插件，等用户加入（`m.room.member` 事件）
  之后再发：之前发的消息，房间密钥没有发给用户的设备，用户解不开。
- 令牌对应一台设备，核心不主动轮换。吊销设备、删掉 Bot 时令牌立即失效。核心重启时核对 Bot 的
  设备还在，被吊销了就把插件标为异常，重装（可以不删数据）重新签发；Bot 换了一台设备，要重新建加密身份。
- 重建容器时沿用设备和令牌（升级；第一次配上或取消出网代理）。核心不单独保存令牌，它只在 Docker 的容器配置里，所以从旧容器读回：
  1. 用占位值渲染一次，找出哪些服务的哪些环境变量引用了 `WONTON_MATRIX_ACCESS_TOKEN`。
  2. 从旧容器读出这些变量。全部读到之后才删旧容器。
  3. 旧容器不在，或者令牌写在 `command`、`configs` 里读不回来：不删、不重建，把插件标为异常，提示重装。不悄悄签新设备，换设备要重新建加密身份。
- Agent 自己生成的加密恢复密钥之类，用 `secrets` 的 `collect` 交给核心保管（见“插件密钥”）。
- 卸载插件时吊销 Bot 的所有设备；选择删除数据时再停用 Bot 账号，账号随之退出所有房间。停用不可恢复，所以
  保留数据重装时沿用原账号，房间和历史都还在；删除数据后重装是一个新账号。卸载结果的 `bot` 字段写着 Bot 的
  用户 ID、吊销的设备数和是否停用。

### 名字和头像

名字和静态头像就是 Bot 的 Matrix 全局资料（`displayname`、`avatar_url`），所有 Matrix 客户端都看得到。每改一次，
Bot 在的每个房间都会多一条成员事件，App 把它显示成“某某 改名为 某某”一行。

状态头像不进房间：四个状态的头像整组写在 Bot 资料的自定义字段 `app.wonton.avatars` 里，不产生成员事件，
别的客户端看不到，换一整套也只是一次资料写入：

```json
{
  "states": {
    "idle":    { "url": "mxc://…", "mimetype": "video/mp4", "w": 480, "h": 480, "size": 812345, "duration_ms": 3000, "poster": "mxc://…" },
    "working": { "url": "mxc://…", "mimetype": "image/webp", "w": 512, "h": 512, "size": 402311, "duration_ms": 2000 }
  }
}
```

`states` 的键是状态，各字段：`url` 是 Matrix 媒体库里的 `mxc://` 地址，`mimetype`、`w`、`h`、`size` 照文件写，
`duration_ms` 只有动图和视频有，`poster` 是视频的静态首帧（`mxc://`），可以不写。键只认 `idle`、`working`、
`waiting`、`offline` 这四个，认不出的键原样保留。没写的状态用 `idle` 的，`idle` 也没有就显示静态头像。
状态头像经 `wonton__set_profile`（见“工具与 MCP 总线”）改；直接写这个字段会绕开核心的媒体检查和每小时
10 次的改动上限，规范不提倡。

四个状态什么时候出现，由核心和 App 判断（插件在运行、请求和工具调用数这些事实，加上 Matrix 的“正在输入”），
插件不用上报：

- `offline` 离线：插件没在运行——正在安装、升级、启动，或者停了、出错了。
- `waiting` 等你确认：插件在运行，有这个 Bot 发起的确认卡片在等用户回答。
- `working` 工作中：有模型请求或工具调用在进行，或者 Bot 正在输入。
- `idle` 待机：其余时候。

**默认素材**

清单的 `bot` 可以带 `avatars`，每个状态指向插件包里的一个文件：

```yaml
x-wonton:
  bot:
    display_name: { zh: Hermes, en: Hermes }
    avatar: avatar.png                 # 静态头像
    avatars:
      idle: avatars/idle.mp4
      working: avatars/working.webp
      waiting: avatars/waiting.mp4
      offline: avatars/offline.mp4
```

视频的首帧放在旁边：同名换成 `.jpg` 后缀的文件（如 `avatars/idle.jpg`），核心有就一起传成首帧，可以省略。

核心在安装、升级时，只在 Bot 资料里还没有 `app.wonton.avatars` 字段时按清单传上去写进去；字段已经在（用户或
Agent 设过，包括清成了空表）就不碰。所以升级到带素材的版本也能拿到默认形象，用户或 Agent 设过的不会被冲掉。

**限制**

- 方形，边长不超过 640 像素。
- 每个文件不超过 2 MiB；动图和视频不超过 6 秒。
- 格式：静态头像只能是 PNG、JPEG、WebP 的静态图；状态头像另可 GIF、动图 WebP、H.264 的 MP4（不要声音）。
- 核心按文件内容判断格式和尺寸，不看扩展名。清单的静态头像 `bot.avatar` 例外：安装时只按扩展名认
  png、jpg、gif、webp（见上面安装的第 3 步）。

### 一个插件多个 Bot（`bots`）

核心只发默认 Bot，家人的私人 Bot 留待以后。格式先定下，到时插件不用改结构。

```yaml
x-wonton:
  bot:
    display_name: { zh: Hermes, en: Hermes }   # 新 Bot 的默认显示名
  bots:
    services: [init, gateway]                  # 这些服务每个 Bot 一份
```

- 每个 Bot 有自己的账号、令牌、主房间和模型凭据（凭据名 `<插件 id>/<Bot 名>`）。安装时核心建默认 Bot
  `default`；用户新建 Bot 时，核心按 `bots.services` 为它再建一组容器；删掉 Bot 就删掉这组容器并吊销它的
  凭据。其余服务全插件一份。
- 在这些服务里，`WONTON_MATRIX_*`、`WONTON_MODEL_API_KEY` 是这个 Bot 的值；`WONTON_BOT_NAME` 是 Bot 名
  （小写字母、数字和单个 `-`），`WONTON_BOT_DATA` 是 `${WONTON_APP_DATA}/bots/<Bot 名>`。
- 隔离：Bot 服务挂载应用数据目录时只能挂自己的 `${WONTON_BOT_DATA}`，也不能用命名卷，这样一个 Bot 读不到
  别的 Bot 的记忆和凭据。
- 依赖：Bot 服务可以依赖同一个 Bot 的服务和全插件一份的服务；全插件一份的服务不能依赖 Bot 服务。
  `mcp`、`provides` 不能指向 Bot 服务。`webui` 可以，连的是默认 Bot（`default`）那一组的容器；应用只监听回环地址时，
  在同一个 Bot 里用 `network_mode: service:<服务名>` 配一个反向代理对外，`webui` 写被借用的服务。
- Bot 服务在插件网络里的别名是 `<服务名>-<Bot 名>`，同一个 Bot 的服务之间也用它互访，如
  `http://sidecar-${WONTON_BOT_NAME}:8080`。
- 全插件一份的服务引用 `WONTON_MATRIX_*` 时安装失败；引用 `WONTON_MODEL_API_KEY` 时拿到插件级的凭据
  （凭据名是插件 `id`）。
- 没有 `bots` 时插件只有一个 Bot，变量对所有服务都一样。

> **已定：每个 Bot 一组容器**，即上面的写法，沿用单 Bot 的变量名。隔离最好：
> Hermes 的文档写明 profile 不隔离文件系统，同一个容器里的 Bot 能读到彼此的记忆和凭据，而家人的私人
> Bot 必须互不可见。代价是内存，Hermes 空闲时约 210 MB 一个。没选的方案是一个容器跑多个 Bot（Hermes
> 的多 profile）：容器级的环境变量只作用于 Hermes 的默认 profile，其他 profile 只读自己的
> `profiles/<名字>/.env`，核心要按插件声明的模板把每个 Bot 的凭据写成文件；省内存，但只适合同一个人的
> 多个 Bot。

## 通知（`notify`）

申请了 `notify` 的插件把要告诉用户的事交给核心，核心以核心账号发进和用户的“Wonton”私聊：安装时注入
`WONTON_NOTIFY_URL`，插件在插件网络里 `POST` 一条 JSON，不用令牌。监控、下载、备份这类插件的告警走这里，
不用自己再接一套推送渠道。

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `title` | 是 | 一句话，最多 120 个字符 |
| `body` | 否 | 纯文本，最多 2000 个字符 |
| `level` | 否 | `info`（默认）、`warning`、`critical` |
| `path` | 否 | 插件网页里的一页（以 `/` 开头，不能以 `//` 开头，不能有反斜杠和控制字符，可带 query），用户点通知时应用打开这一页；插件没有 `webui` 时不能写 |

最短例子（`wget` 和 `curl` 都行）：

```sh
curl -s -o /dev/null -w '%{http_code}\n' -X POST "$WONTON_NOTIFY_URL" \
  -H 'Content-Type: application/json' \
  -d '{"title":"证书 7 天后到期","body":"app.example.com 的证书快到期了","level":"warning","path":"/certs"}'
```

- 级别决定推不推送：`info` 发成 Matrix 的 `m.notice`，按 Matrix 的默认推送规则不响铃，只进聊天；`warning`、
  `critical` 发成 `m.text`，照常推送。App 里插件通知单独一个类别，用户可以只关插件通知。
- 通知以核心账号发出，正文是“插件名：标题”加换行加正文，给其他 Matrix 客户端看；结构化的字段（插件、级别、
  `path`）App 自己认。
- 频率上限：每个插件每分钟 10 条、滚动 24 小时 200 条。超了的直接丢弃；第一次超限时核心在私聊里发一条说明，
  恢复前不再重复。计数在核心内存里，核心重启后重新计。
- 通知只发给用户本人，家人不收。
- 状态码：发成功是 201；字段不合法是 400 `invalid_input`；没申请 `notify` 是 403 `permission_denied`；
  插件网络以外是 403 `unauthenticated`；超了频率是 429 `rate_limited`；聊天服务不可用是 503
  `matrix_unavailable`。

## 工具与 MCP 总线

Agent 经核心的 MCP 总线调用工具：总线把核心自己的工具和所有在运行的插件声明的工具汇总成
一个 MCP 服务器，按调用方授权，敏感的调用先请用户确认。

### 提供工具（`mcp`）

```yaml
x-wonton:
  mcp:
    service: mcp           # 说 MCP 的服务
    port: 8080
    path: /mcp             # 默认 /mcp
    transport: streamable-http
    headers:               # 可选；{secret:…} 要引用 secrets 里声明的项，完整例子见下
      Authorization: Bearer {secret:app_token}
    tool_idle_seconds: 600    # 可选，见“工具调用的期限”
    tool_total_seconds: 1800  # 可选
    tools: [search_files, read_file]  # 可选，只提供列出的工具
```

- 核心经插件网络连这个服务（同网页入口），它只对核心可见；插件在运行时它的工具才出现在总线上。只说 stdio 的
  MCP 服务器要在镜像里自带一个转成 Streamable HTTP 的桥（如 supergateway）。
- 工具在总线上叫 `<slug>__<工具名>`，如 `app-wonton-files__search_files`。核心连插件的 MCP 服务用 `initialize` 握手
  （MCP 2025-11-25）。
- 上游要凭据时声明 `headers`：头名 → 值模板，写法和 `webui.proxy` 相同，占位符只能用 `{secret:…}`，引用本插件
  `secrets` 里声明的项（`generate` 或 `collect` 都行）。`{user}` 这类身份占位符不能用：MCP 调用不是主人的浏览器
  请求，上游认的是应用签发给核心的凭据。不能加 `Host`、`Cookie`、`Forwarded`、`X-Forwarded-*`、逐跳头部和 MCP 的
  协议头（`mcp-session-id`、`mcp-protocol-version`）。
- 值在安装、升级和核心每次核对插件时按当时的密钥解析。核心发往 MCP 服务的每个请求都带上（包括握手），值不写进
  日志、错误信息和审计。头引用的密钥在安装时就得有值：`collect` 的文件还没写下（比如生成它的一次性服务还没跑完）
  就装不上，总线的头填不出来时宁可不上线。
- 密钥的值里不能有换行或控制字符，结尾的换行也算：`echo` 写的文件末尾多一个换行就填不出头，用 `printf '%s'`
  写。核心会报错到具体的头和密钥，值本身不出现在错误里。
- `collect` 的密钥在核对时读到新值后，核心对这条上游的连接在条目更新后就断开重连，用新值（不需要重启插件）。
  上游回 401/403 时总线如实报“插件的 MCP 服务拒绝了核心的凭据”，不自动重试，也不重新取密钥——换令牌是插件的事。
- 应用只认它自己签发的令牌时（如 Memos 的 MCP 服务）：用一次性服务登录应用、签发一个个人访问令牌（PAT）写进
  文件，核心收存后经 MCP 头带给上游。下次创建容器时先用保存的令牌试，应用拒了就重新签发、写回文件；核心在
  核对时读到新值就用新的。

  ```yaml
  services:
    init:
      image: curlimages/curl
      entrypoint: ["/bin/sh", "/init.sh"]   # 登录应用、签发 PAT，用 printf '%s' 写进 /run/token（别带结尾换行）
      depends_on:
        app: {condition: service_healthy}

  x-wonton:
    secrets:
      app_token:
        collect: {service: init, path: /run/token}
    mcp:
      service: app
      port: 5230
      transport: streamable-http
      headers:
        Authorization: Bearer {secret:app_token}
  ```

- `tools` 是允许列表：写了就只提供列出的工具，名字是上游的原名（不带 `<slug>__` 前缀）；不写时全部提供。至少一项、
  不能重复，每个名字按 MCP 2025-11-25 的工具名规定校验（1–128 个字符，只有 ASCII 字母、数字、`_`、`-` 和 `.`）。
  列表里写了但上游没有的名字：其余工具照常提供，缺哪些写在 `wonton__list_plugins` 的 `tools_error` 里；调用不在
  列表里的工具和调用不存在的工具一样报错。
- 没有标 `readOnlyHint: true`（MCP 的工具注解）的工具算敏感：总线先请用户确认，确认了才转给插件。会改数据的工具
  不要标成只读。不会改数据的工具要标上，否则 Bot 每次调用都要用户确认。
- 工具返回的内容交给 Agent，Agent 把它当不可信输入。除了下面的文件引用，总线不改内容。

**列工具**

- 每个 Agent 列工具时，总线并发地问所有插件。一个插件最多 15 秒，包括没连上时的握手；最多 50 页、2000 个工具、8 MiB。超时或超限的插件，它的工具这次不出现，原因在 `wonton__list_plugins` 的 `tools_error` 里。翻页翻回同一个 `cursor` 算插件的错。

**连接**

- 一个请求的回答不对（非 2xx、`Content-Type` 不对、JSON 解析不了、回答的 `id` 不是这个请求的）算插件对这一个请求的错误，连接照用，别的请求不受影响。连接断了才重连。
- 连续 3 次带期限的请求（`tools/list`、`resources/read`、`tools/call`）没等到回答，中间也没有任何请求拿到回答，总线就当这个会话卡死了：取消它，下次新建一个，有状态的 stdio 子进程跟着丢。同时超时的算一次。有更早开始的 `tools/call` 或 `resources/read` 还在跑时，超时不算：串行的服务器做长调用时 `tools/list` 排在后面，这段时间工具不出现，但会话和调用都留着。

**工具调用的期限**

- 总线等一个 `tools/call` 有两个期限：空闲期限（默认 10 分钟，`tool_idle_seconds`），收到这个调用的 `notifications/progress` 就重新计时，总线的请求都带 `progressToken`；总期限（默认 30 分钟，`tool_total_seconds`），不管有没有进度。都可以声明得更长，最多一周。长的工具要么发进度，要么把两个期限都写长：只写总期限，空闲期限先到；空闲期限超过总期限（不写时按 30 分钟算）的清单不合法。
- 到期后总线给插件发 `notifications/cancelled`，工具要尽量停下来。这时不知道工具做到了哪一步，结果算不确定，不算失败：Agent 收到 `isError: true`，`_meta` 的 `app.wonton/failure` 写明原因（`tool_timed_out`，或者 `session_replaced`、`connection_lost`、`result_too_large`）；需要确认的调用，卡片标“结果不确定”。
- 整个回答超过 48 MiB（一条 JSON 消息的上限，和单个内容块的 1 MiB 是两回事）也算 `result_too_large`，结果不确定：大文件按“工具结果里的文件”用引用交，不要塞在回答里。
- 请求没到插件（连接已经关了、连不上插件）不算不确定，算失败。
- 只读的调用，Agent 发了 `notifications/cancelled` 或者关闭了会话，总线把取消传给插件。确认过的调用不跟着 Agent 走：它取消了或者关闭了会话都照样执行，结果在卡片上。

**工具结果里的文件**

工具结果是交给 Agent 的。Agent 没有用户的登录，打不开插件网页，也看不到共享文件夹。文件要这样返回，核心把引用换成一个编号，
Agent 用编号调 `wonton__view_file`、`wonton__share_file`，用户的设备按链接经核心的文件服务下载。

| 文件 | 返回 |
| --- | --- |
| 小图（不超过 1 MiB），Agent 看一眼就够 | `image` 块。它没有编号，分享不了；要能分享原图，再给一个下面的引用。 |
| 共享文件夹里的文件 | `resource_link`，`uri` 是 `wonton-share:<共享文件夹>/<相对路径>`，路径按 URI 规则做百分号编码。 |
| 插件自己的文件 | `resource_link`，`uri` 是插件的 MCP 资源，插件实现 `resources/read`。大文件在 `_meta` 里加内网地址，支持 `Range` 才能续传和流式播放。 |

```json
{
  "content": [
    { "type": "text", "text": "找到 2 张 7 月的照片" },
    {
      "type": "resource_link",
      "uri": "wonton-share:photos/2025/07/IMG_1234.jpg",
      "name": "IMG_1234.jpg",
      "mimeType": "image/jpeg",
      "size": 4404019
    },
    {
      "type": "resource_link",
      "uri": "immich://assets/3f2a/original",
      "name": "IMG_1235.heic",
      "mimeType": "image/heic",
      "_meta": { "app.wonton/internal_url": "http://server:2283/internal/assets/3f2a/original" }
    }
  ]
}
```

- `name`、`mimeType`、`size` 用来告诉 Agent 和显示卡片。`name` 没写时取路径的最后一段。下载时的类型由核心决定，不可信的类型一律按附件下载。
- 共享文件夹必须是这个插件申请到的（读或写都算），写别的共享文件夹不会登记。路径不能是绝对路径，不能含 `.`、`..` 或空段，路径上不能有软链接，
  必须是普通文件。第一段是共享文件夹的名字，不是容器里的挂载点：`${WONTON_SHARE_PHOTOS}:/photos:ro` 里的 `/photos/a.jpg` 写成
  `wonton-share:photos/a.jpg`。
- `app.wonton/internal_url` 只有核心会访问：主机名必须是这个插件 compose 里的服务名，协议是 `http`，不跟随跳转，不用 IP 字面量。不声明时核心用
  `resources/read`，最多读 32 MiB。这个键在交给 Agent 前去掉。
- `uri` 是 `http`、`https` 的 `resource_link` 不是文件引用，原样交给 Agent。
- 引用登记不了时（没有授权、路径不合法等），Agent 看到的是一句说明原因的文字，不是这个引用。
- 不要：
  - 返回宿主路径；
  - 用 `blob`、`image`、`audio` 塞大文件：超过 1 MiB 的块，总线换成一句说明原因的文字；
  - 自己往聊天里发文件：发不发给用户由 Agent 决定；
  - 一次返回几百个文件：一个结果最多登记 100 个，要分页。

### 使用工具（`tools`）

- 申请了 `tools` 的插件得到 `WONTON_MCP_URL`，以 Streamable HTTP 连接。网关按来源地址认出调用方（同“模型网关”），
  不在插件网络里的请求得到 403 `unauthenticated`，没申请 `tools` 的得到 403 `permission_denied`。
- 申请了 `tools` 的插件看到全部工具；按 Bot 的白名单放行留待以后（一个插件多个 Bot 时，每个 Bot 另发令牌）。
- 插件装上、卸掉、恢复运行时工具列表随之变化，总线给连着的客户端发 `notifications/tools/list_changed`。
- 核心自己的工具：

  | 工具 | 敏感 | 说明 |
  | --- | --- | --- |
  | `wonton__list_plugins` | 否 | 已装的插件、版本、状态和网页地址；工具列不出来时带 `tools_error`。 |
  | `wonton__search_catalog` | 否 | 按关键词在官方索引里找插件：ID、名字、说明和要申请的权限。 |
  | `wonton__install_plugin` | 是 | `id` 和 `dir` 必须且只能给一个。`id`：按官方索引里的 ID 安装，授予它声明的全部权限。`dir`：装或更新自己写的插件，见“Agent 写插件”。高权限也走确认卡片，醒目标出；最高级权限多一张确认卡，两张都过才装。 |
  | `wonton__uninstall_plugin` | 是 | 卸载，`delete_data` 为真时同时删除它的数据。 |
  | `wonton__view_file` | 否 | 按文件编号看文件：图片返回缩小的预览；文本文件（按内容判断：能按 UTF-8 解、没有 NUL 字节，不只看 MIME）返回开头最多 64 KiB 的内容，截断时写明文件总大小。PDF 等其他二进制报错，这类读不了内容。 |
  | `wonton__share_file` | 否 | 把编号指向的文件分享给用户，返回文件链接，原样放进回复。 |
  | `wonton__list_shares` | 否 | 列出用户的共享文件夹的名字。 |
  | `wonton__plugin_guide` | 否 | 写插件的说明和两个最小例子。返回 `spec_version`（这台核心支持的规范版本）、`guide` 和 `examples`。 |
  | `wonton__check_plugin` | 否 | `dir` 是调用方 outbox 里的插件目录（相对路径），和 `wonton-core plugin check` 同一套校验，不装也不改。返回 `id`、`version`、`digest`、`files`（`path` 和 `bytes`）、`problems`、`warnings`；解析不了时只有 `problems`。 |
  | `wonton__plugin_status` | 否 | `id` 是已装的插件。返回 `status`、`url`，有的话带 `health`；有网页时带探测的状态码 `web_status`，探测不了时是 `web_error`。调用方写的插件还带 `logs`：每个服务（`service`）最近的日志（`text`），最多 40 行、8 KiB；取不到时是 `logs_error`。 |
  | `wonton__save_file` | 按路径授权 | 把 outbox 里的文件存进共享文件夹，见下面的“Bot 存文件”。 |
  | `wonton__get_profile` | 否 | 读你自己的 Bot 的资料：实际名字、有没有静态头像、哪些状态设了状态头像，和“名字和头像”里的状态与限制。只作用于调用方自己的 Bot，不接受参数。 |
  | `wonton__set_profile` | 否 | 改你自己的 Bot 的名字或头像，不需要用户确认。`name`：新名字，去掉首尾空白后 1 到 32 个字符，不含控制字符，不能用核心（Wonton、云吞、雲吞）或用户的名字，不分大小写。`avatar`：静态头像在 outbox 里的相对路径。`avatars`：要改的状态，每项是相对 outbox 的路径，写 `null` 清掉这个状态，没写的不动。不收 `mxc://`。成功后删掉用过的 outbox 文件，失败时保留。每个 Bot 每小时最多成功改 10 次，超了返回 `rate_limited` 和还要等的秒数。 |
  | `wonton__help` | 否 | Wonton 和 App 的帮助文档，见下面的“帮助文档”。 |

- 插件目录就是官方索引（见“发布与来源”）。Bot 能装官方索引里的插件和自己写的插件（见“Agent 写插件”），不能装社区插件。
- 敏感的调用会一直等到用户回答或者确认过期（10 分钟）。确认卡片发在核心和用户的“Wonton”私聊里，用户在 Wonton App
  里回答。用户拒绝或者过期时工具返回错误（`isError: true`），说明原因。
- 用户在 App 里点了“我来操作”时，这个插件的敏感调用先停住，不发卡片也不执行，用户交还后才继续判定。这段最多 10 分钟，
  和卡片各算各的，所以 Agent 的工具超时要超过 20 分钟。只读工具照常执行。停得太久或者停住的调用太多时返回错误，
  `_meta` 的 `app.wonton/failure` 是 `owner_operating`：调用明确没有执行，过一会可以重试。

**Bot 存文件**

Bot 把下载好的文件存进用户的共享文件夹，用户在 Navidrome、File Browser 这些插件里就能看到。只要 `tools` 权限，不需要 `storage.share.*`，也因此读不到、列不出共享文件夹里的文件。

```yaml
services:
  agent:
    volumes:
      - ${WONTON_APP_DATA}/outbox:/opt/data/outbox
```

- 插件把 `${WONTON_APP_DATA}/outbox` 挂进容器，Bot 把要存的文件放进去，再调 `wonton__save_file`。没挂的插件调用时报错。
- 参数：`files` 是 outbox 里的相对路径，1 到 100 个；`share` 是共享文件夹的名字；`dir` 是里面的子目录，可选，没有的会建出来。绝对路径、`..`、隐藏文件和软链接都拒绝。
- 第一次存进一个文件夹时，用户在确认卡片上选：允许这个文件夹，或者允许整个共享文件夹（目标是共享文件夹本身时没有这一项），或者拒绝。授权按路径记在插件上，包括子文件夹，之后不再问。用户可以在插件详情页撤销，卸载插件时一起清掉。调用会一直等到用户回答或者确认过期。
- 只新增不覆盖：同名时加序号。结果的 `saved` 写每个文件最后的路径，路径可能和请求的名字不一样。
- 卡片发出之前核心已经把文件复制走，用户确认的就是存下去的内容。存好后 outbox 里的原文件删掉；复制之后又被改过的不删。拒绝或者过期时原文件留给 Bot 处理。
- 放不下时返回 `storage_full`，磁盘要留 5% 空闲。一批文件中途放不下时停下，前面放好的不撤回，错误里写明。

**帮助文档**

- 参数都可选。`topic` 是主题；不带时 `topics` 是目录，每项有 `topic`、`lang`、`title`、`summary`。带了时返回那一页：`topic`、`lang`、`title`、`summary`、`body`。不认识的主题返回 `invalid_input`，并列出可选的。`lang` 是语言，默认用用户的语言，缺时回退规则同“商店信息”。
- 结果都带 `core_version`（读不到时是 `unknown`）、`docs_commit`、`docs_updated_days_ago`。取新文档失败时再带 `refresh_problem`，写明原因。这台核心一份文档都没有时返回 `docs_unavailable`。
- 页面声明了 `requires_core`、`requires_app` 时目录和页面都带上；核心版本不够时页面再带 `note`。段落里写了“需要核心 ≥ x”时，Agent 拿它和 `core_version` 比。App 的版本 Agent 不知道，要问用户。
- 文档的来源是公开的[用户文档](user/README.md)。核心镜像带一份，有更新时按官方索引的 `docs` 取，最多等 3 秒，没取完先用手头的回答。

### Agent 权限确认

Agent 自带的工具（shell、文件这类不经 MCP 总线的）要动手之前，Agent 插件可以请用户确认。这不是 MCP 调用：`POST` 和 `WONTON_MCP_URL` 同一个地址，路径换成 `/agent/permissions`，同样要有 `tools` 权限，同样按来源地址认出调用方。

```json
{
  "tool_call_id": "call-1",
  "title": "运行 ls",
  "description": "列出工作区的文件",
  "arguments": {"command": "ls"},
  "options": [
    {"choice_id": "a", "kind": "allow_once", "name": "这次允许"},
    {"choice_id": "b", "kind": "allow_always", "name": "总是允许"},
    {"choice_id": "c", "kind": "reject_once", "name": "拒绝"}
  ]
}
```

- `tool_call_id`、`title`、`options` 必填且不能为空，请求体不超过 1 MiB。`description` 和 `arguments` 可选。`arguments` 会进卡片和审计，秘密要先去掉。
- `choice_id` 由插件自己定，核心不解释，同一次请求内不能重复。`kind` 只能是 `allow_once`、`allow_always`、`reject_once`、`reject_always`；卡片上的按钮按 `options` 的顺序显示 `name`。
- `allow_always` 的作用域归 Agent 自己，不写进核心的工具策略，也不放行 MCP 工具。
- 没有拒绝类的选项时，核心在卡片上加一个“取消”。

这是一个长请求，用户回答或者 10 分钟到期才返回，HTTP 客户端的超时要比这长：

| 响应 | 含义 |
| --- | --- |
| `{"outcome": "selected", "choice_id": "a"}` | 用户选了这一项，包括拒绝类的选项。 |
| `{"outcome": "cancelled"}` | 用户点了核心加的“取消”。 |
| `{"outcome": "expired"}` | 用户没在期限内回答。 |

- 请求不合法（`kind` 不认识、`options` 为空、`choice_id` 重复等）时返回 400 `invalid_input`。
- 这个插件已经有 3 个确认在等用户时（和 MCP 工具的确认共用这个上限）返回 429 `too_many_pending`，卡片发不出去时返回 503 `confirmation_unavailable`。这两种都没有问到用户，不能当成同意。
- 插件提前断开请求就是撤回，卡片随之作废。

## 插件健康探测（`plugins.health`）

监控类插件（如 Uptime Kuma）申请 `plugins.health` 后，经核心在插件网络里的别名
`http://health.wonton.internal:8080` 读所有插件的健康状态，不用主人的登录凭据，也连不到任何插件的管理界面。
按来源地址认出来源插件（同“工具与 MCP 总线”）：不在插件网络里的请求得到 403 `unauthenticated`，
没申请 `plugins.health` 的得到 403 `permission_denied`。接口只读：不返回日志、配置和密钥，不触发任何操作。

| 接口 | 返回 |
| --- | --- |
| `GET /plugins` | JSON 数组，每个已安装插件一项：`{id, title, state, checked_at}`。 |
| `GET /plugins/<id>` | 同上，再加 `probe`：核心这一刻探测那个插件网页入口的结果。 |

- `state` 是核心已知的状态：`running`、`stopped`、`degraded`（已安装但异常）、`installing`、`uninstalling`、
  `upgrading`。以后核心新增的状态按 `other` 对待。
- `checked_at` 是核心最近一次核对状态的时间（Unix 秒）。
- `probe` 是核心从自己的位置请求目标插件网页入口（webui 声明的端口和路径，带插件公开地址的 Host，不带主人
  身份）的结果，每次限期 5 秒，形状是 `{reachable, http_status, latency_ms, error_kind}`。连不上时
  `error_kind` 是 `unreachable`，超时是 `timeout`，上游回了 5xx 是 `upstream_error`，三种 `reachable` 都是
  `false`；其余状态码（包括应用自己跳登录、要鉴权的响应）`reachable` 都是 `true`。没声明网页入口的插件
  `probe` 为 `null`，`probe_reason` 写明原因。
- 同一个插件每秒最多请求 5 次，超了返回 429 `too_many_requests`。

例子：Uptime Kuma 加一个 “HTTP(s) - Keyword” 类型的监控项，地址填
`http://health.wonton.internal:8080/plugins/<id>`，关键字填 `"reachable":true`。目标插件正常运行时监控是绿的；
插件停了，核心探测连不上，`reachable` 变成 `false`，监控变红。

## Agent 插件

`kind: agent` 的插件是一个 Bot 的驱动。`protocol: matrix` 的 Agent 自己登录 Matrix；`protocol: acp` 的由 ACP 桥接到聊天。

**账号登录。** 模型默认经模型网关。Agent 官方程序能用账号登录时，在 `agent.login` 里声明，核心经引导和插件详情的接口把选项交给 App：

```yaml
x-wonton:
  kind: agent
  agent:
    protocol: acp
    login:
      - id: chatgpt
        title: { zh: ChatGPT 账号, en: ChatGPT account }
  permissions: [network.egress]
```

- `id` 在插件内唯一，由小写字母、数字和单个连字符组成。`title` 是多语言文本。
- 声明了 `login` 的插件必须申请 `network.egress`：账号登录后，模型请求直接发给官方，不经过网关。
- 登录在官方程序里完成，不要把账号的凭据交给网关或别的程序。登录态放在数据目录里，容器重建后还在。
- 使用前看该 Agent 的条款。有些厂商不许经第三方转发订阅，也限制使用它的名字和标志。

**帮助 skill。** 官方 Agent 插件要带 [`skills/wonton-help`](skills/wonton-help/SKILL.md)，放进这个 Agent 读 skill 的目录。不带时用户问 Wonton 的用法，Agent 不一定想得到调 `wonton__help`：

| Agent | 目录 |
| --- | --- |
| Hermes | `$HERMES_HOME/skills` |
| Claude Code | `~/.claude/skills` |
| Codex、OpenClaw、omp、Gemini CLI、OpenCode、Kimi Code 和其他读 Agent Skills 共用目录的 | `~/.agents/skills` |

## 插件密钥（`secrets`）

```yaml
x-wonton:
  secrets:
    web_key:                          # 核心生成，注入为 WONTON_SECRET_WEB_KEY
      generate: { bytes: 32 }
    matrix_recovery_key:              # 插件运行中生成，交给核心保管
      collect: { service: gateway, path: /opt/data/platforms/matrix/recovery-key.txt }
```

名字由小写字母、数字和 `_` 组成，以字母开头。每项必须有 `generate` 或 `collect` 之一，不能两个都有。
两种密钥都存在核心的密钥库里（核心数据目录下，文件权限 0600，插件的容器访问不到）：卸载时保留数据就一起
保留，重装时沿用（数据库密码之类已经写进了数据，换了就对不上）；选择删除数据时一起删除。

**核心生成（`generate`）**

- `bytes` 是随机字节数，1 到 1024，默认 32；`encoding` 是 `hex`（默认，小写）或 `base64`（带填充、不换行），
  结果和 `openssl rand -hex <bytes>`、`openssl rand -base64 <bytes>` 的格式相同。
- 首次安装时生成，以 `WONTON_SECRET_<名字大写>` 注入；升级时保留，升级新增的项在升级时生成。已有的值
  不随 `bytes`、`encoding` 的修改重新生成。
- 做法同 Runtipi 的 `random` 字段：随机生成后保存，不像 Umbrel 的 `APP_SEED` 那样从主种子派生——只恢复了
  应用数据、没恢复种子时，派生出的值就对不上。
- v0.1 不轮换，也不在 App 里显示（待定）。

**插件生成、核心保管（`collect`）**

插件运行中自己生成、丢了就无法恢复的秘密，如 Hermes 自动建交叉签名后写下的恢复密钥。清单声明它在哪个
服务的容器里、哪个文件：

- 核心在安装时所有服务就绪后、以及核心每次启动核对插件时，用 Docker 的归档接口读取。路径在容器里解析，
  软链接出不了容器，最后一级是软链接时不跟随；只接受不超过 64 KiB 的普通文件，是目录、软链接或超过上限时
  安装失败（核心启动核对时则把插件标为异常）。文件还不存在时不算失败，下次核对时再试；读到以后每次核对，
  变了就更新。
- 核心不修改、不删除插件里的这个文件，也不注入回去。
- 声明在 Bot 服务上时，每个 Bot 各存一份。

## 对宿主的要求

`requirements` 的每一项都是核心能判断的，商店和详情页逐项显示这台主机满足、不满足还是查不出来。核心只跑在 Linux 和 Docker 上，所以没有系统这一项。

```yaml
requirements:
  arch: [amd64, arm64]
  memory: 2G
  gpu: { need: required, vendors: [nvidia, amd], memory: 8G }
```

| 字段 | 说明 | 核心怎么判断 |
| --- | --- | --- |
| `arch` | `amd64`、`arm64`。不写表示都行。插件的每个服务都要能在列出的每种架构上跑，见下。 | Docker 报告的架构。 |
| `memory` | 应用能跑起来的内存下限（官方写的最低要求，不是推荐值）。写法是数字加 `M` 或 `G`，如 `512M`、`1.5G`，1G 等于 1024M。 | 主机的总内存。标称 8G 的机器实际报出来不到 8G，所以到要求的 90% 就算够。 |
| `gpu.need` | `required`（默认）：没有符合的显卡就装不上。`optional`：照常装，不给设备，插件自己退回 CPU。 | 见“显卡”。 |
| `gpu.vendors` | 能用的厂商：`amd`、`nvidia`、`intel`。不写表示哪家都行。 | 驱动可用的显卡的厂商。 |
| `gpu.memory` | 显存下限，写法同 `memory`。 | 显卡报告的显存，到要求的 90% 就算够（标称 8G 的卡实际报 8188 MiB）。集成显卡报告的够要求就算满足，不够算查不出来：划给核显的量不是它的上限。驱动不报告的算查不出来。 |

- `arch` 管整个插件，没有“某个服务只在某种架构上装”的写法：插件在一台主机上装上，就是每个服务都在跑，工具和网页都能用。
  某个组件（常见的是 MCP 桥这类小服务）上游只发了一种架构的镜像时，自己构建多架构镜像（模板的 `scripts/publish.sh` 就是这样构建的），或者把
  `arch` 收窄到它支持的架构，或者不带这个组件。不要写成在别的架构上启动失败、或者跳过它的样子。
- 想在自己的主机上看每条要求的判断结果：`wonton-core plugin host [插件包目录]`。
- 不满足的项写明缺什么（“这台主机只有 1.5 GB 内存”）。有 `required` 的项不满足时插件装不了，商店和预览的 `blocked` 写着原因，安装接口也拒绝：架构和内存的错误类别是 `unsupported`，显卡是 `gpu_unavailable`。
- `optional` 的显卡不满足时照常装，显示会退回 CPU。
- 查不出来时如实显示，不拦安装。
- 只在安装和预览时判断。装好后主机变了，容器自己会出错。

## 资源限制

普通插件什么都不用写。核心给每个服务设下面的默认值，都是防失控用的，碰不到时没有任何影响：

| 字段 | 默认值 | 说明 |
| --- | --- | --- |
| `cpu_shares` | 512 | 只是权重：机器闲时不限速，忙时给系统让路。 |
| `mem_limit` | 主机总内存的 80%，且不低于 `requirements.memory`；读不出总内存时不设 | 超过时进程被系统结束。不用交换区。 |
| `pids_limit` | 4096 | 进程和线程的总数。 |
| `cpus` | 不限 | 硬上限，闲时也限速，一般不用写。 |

要不一样时在 compose 的服务上写这些字段，再加 `mem_reservation`（内存紧张时优先保留多少，不能大于 `mem_limit`），写法和 compose 相同。插件不能要交换区。

用户可以调整单个插件的内存和 CPU 上限，对它的每个服务生效，优先于 compose 写的值。

- 有的主机还会给所有插件设一个总量，并留一部分内存给系统。插件不能假设一定用得到 80%。总量到顶时进程直接被结束，不会先放慢，服务没到自己的上限也可能被结束。
- 内存不够被结束时，核心记下是哪个服务。容器反复重启时插件显示为异常，原因写内存；只结束了子进程或重启后好了时，插件照常运行，用户看到一行提示，24 小时后消失。有总量的主机上分不清是服务到顶还是总量到顶，两种可能都写上。
- 进程数超限时 `fork` 返回错误，由插件自己报。

## 显卡（`devices.gpu`）

要用显卡的插件（生图、生视频、本地模型）申请 `devices.gpu` 权限，并在 `gpu.services` 里写明哪些服务拿到显卡。
交给容器什么由核心按宿主实际有的显卡决定，compose 里自己写的 `devices` 一律拒绝：插件写不了宿主路径，
一份清单在 AMD 和 NVIDIA 的主机上都能装。

```yaml
name: comfyui
services:
  comfyui:
    image: ghcr.io/example/comfyui:1
    shm_size: 8g
    healthcheck:
      test: ["CMD", "curl", "-f", "http://127.0.0.1:8188/"]
  web:
    image: ghcr.io/example/comfyui-proxy:1

x-wonton:
  id: ai.example.comfyui
  version: 1.0.0
  kind: app
  title: { en: ComfyUI }
  description: { zh: 用工作流生图、生视频, en: Image and video generation with workflows }
  webui:
    service: web
    port: 8188
  gpu:
    services: [comfyui]
  permissions:
    - devices.gpu
  requirements:
    gpu: { need: required }
```

| 字段 | 说明 |
| --- | --- |
| `gpu.services` | 拿到显卡的服务，至少一项，不能重复，都要在 `services` 里。其余服务什么都不拿。 |
| `requirements.gpu` | 要什么样的显卡，见“对宿主的要求”。没写时等于 `need: required`。 |

三者缺一不可：写了 `gpu` 或 `requirements.gpu` 却没申请 `devices.gpu`，或者申请了 `devices.gpu` 却没写 `gpu.services`，
清单都不合法。

核心按宿主的显卡交给这些服务，容器里的路径和宿主相同：

| 宿主 | 交给容器什么 |
| --- | --- |
| AMD、Intel 等开源驱动的显卡（有 DRM 渲染节点） | 设备 `/dev/dri`（整个目录）；有 KFD（amdgpu 的 ROCm 计算接口）时还有 `/dev/kfd`。容器加入这些设备节点所属的组（宿主 `video`、`render` 组的 GID），以非 root 用户运行的服务才开得了设备。 |
| NVIDIA 官方驱动 | Docker 的 `DeviceRequests`（同 `docker run --gpus all`），由宿主上的 NVIDIA Container Toolkit 把全部 NVIDIA 显卡和驱动库放进容器。 |

- 两种显卡都有时都给。核心只给显卡本身；`privileged`、`cap_add`、`security_opt`、`ipc: host` 要分别声明权限；要大一点的共享内存
  写 `shm_size`。
- 有没有可用的显卡看 sysfs 里的驱动和设备节点，不看 `nvidia-smi`、`rocminfo` 这类命令装没装。`required` 的插件装不上时
  错误类别是 `gpu_unavailable`（HTTP 422），原因分清：没有找到显卡；有显卡但没有可用的驱动（没加载驱动，或者是
  nouveau）；有 NVIDIA 驱动但 Docker 没有登记 `nvidia` 运行时（没装 NVIDIA Container Toolkit）；核心在容器里却没挂载
  宿主的 `/dev`。
- 安装预览的说明（`notices`）里有一行写明拿显卡的服务，以及本机会交给它们什么（厂商、设备、组）或者为什么给不了；`required`
  而给不了时同时是装不了的原因（`blocked`）。确认卡片列出拿显卡的服务。
- 核心在容器里看不到宿主的 `/dev`，compose 把它只读、不递归地挂到 `/host/dev`（`WONTON_HOST_DEV`，`/dev/shm` 这类子挂载
  不带进来）；没挂时申请显卡的插件装不上，原因写明，核心不拿容器自己的 `/dev` 当宿主的。
- 装好后核心不再盯着显卡：显卡拔了、驱动换了，容器自己会出错。

## 局域网（`network.lan`）

要让局域网里的电脑、手机直接连上来的插件（Samba、Syncthing），申请 `network.lan` 权限，并在 `lan.ports` 里逐项写明开放哪个
服务的哪个端口。

```yaml
name: samba
services:
  samba:
    image: ghcr.io/example/samba:4
    volumes:
      - ${WONTON_SHARE_DOCUMENTS}:/shares/documents
  web:
    image: ghcr.io/example/samba-web:1
    environment:
      LAN_ADDRESSES: ${WONTON_LAN_ADDRESSES}

x-wonton:
  id: org.example.samba
  version: 1.0.0
  kind: app
  title: { zh: 文件共享, en: File sharing }
  description: { zh: 用 SMB 在局域网里共享文件夹, en: Share folders over SMB on the LAN }
  webui:
    service: web
    port: 8080
  lan:
    ports:
      - { service: samba, port: 445 }
  permissions:
    - network.lan
    - storage.share.documents:write
```

| 字段 | 说明 |
| --- | --- |
| `lan.ports[].service` | 开端口的服务，要在 `services` 里，不能写 `network_mode: none`，不能是写了 `network_mode: service:…` 的服务（写它借用的那个服务）。 |
| `lan.ports[].port` | 端口号。宿主端口和容器端口相同，不做映射。 |
| `lan.ports[].protocol` | `tcp`（默认）或 `udp`。 |

清单不合法的情况：`lan` 和 `network.lan` 只写了一个；`ports` 为空、端口为 0 或者同一端口和协议写了两次；把 `mcp`
的 TCP 端口列进来（它只经核心访问）；同时申请了 `matrix.bot`（地址变了核心要按记录重建容器，而它不保存 Bot 的访问令牌）。

- **端口开在哪里**由核心决定：默认是宿主默认路由所在网卡的私有 IPv4 主地址，用户可以在 App 的局域网设置里改选网卡或地址
  （管理接口 `/_wonton/api/lan`），或者给核心设 `WONTON_LAN_ADDRESSES` 指定。端口只发布在这些地址上，
  从不发布到 `0.0.0.0` 或 `::`。选出的地址经 `WONTON_LAN_ADDRESSES` 交给插件，插件在自己的网页上告诉用户怎么连。
- **选不出地址就装不上**：错误类别是 `lan_unavailable`（HTTP 422），原因写明是哪一种（没有默认路由、默认路由走的是虚拟网卡、
  网卡上没有私有 IPv4 地址、检测失败）。预览的 `notices` 里每个端口一行，写明开在哪块网卡的哪个地址上；端口被别的插件的
  `lan.ports` 占着、或者选不出地址时，`blocked` 写明原因。
  `lan_webui` 关着、没有要开的端口时，不检查地址，不报 `lan_unavailable`。
- **端口要和宿主上现有的东西不撞**：核心在预览时拿要开的地址和端口跟 Docker 里所有运行中的容器已发布的端口对一遍（通配的
  `0.0.0.0`/`::`、UDP 分开算，同一个插件自己的旧容器不算），撞上了 `blocked` 写明被哪个容器的哪个地址端口占着，装不上；
  装的时候再查一遍，启动容器时 Docker 也会拒绝。端口被占的类别是 `port_taken`（HTTP 409），错误里带地址和端口。
  宿主上非 Docker 的进程占的端口核心连一下试出来（连一下只判得了 TCP），只作为预览里的提示，不挡安装，探测本身失败时预览写明
  没能检查；插件挑宿主上空的端口是插件的事，核心不自动换端口。
- **发布在哪个网络上**：声明了 `network.egress` 的插件直接发布在自己的插件网络上。没声明的，核心另建 `<插件网络>--lan`
  （关掉地址伪装的桥接网络），只把开着局域网端口的服务以网关优先级 1 接进去（`webui` 的端口要看 `lan_webui`，见“网关代登录”和“应用自带的登录”），端口发布在它上面；这些服务仍在插件网络里，
  主动连外网和局域网的其他主机仍然不通。
- **地址变了自动跟上**：装了这类插件时，核心启动后和之后每 5 分钟检测一次。地址变了，重建 `lan.ports` 里的服务，以及 Compose 插值实际用到 `WONTON_LAN_ADDRESSES` 的服务（包括 `configs.content`）。`$$` 转义、没有求值的插值分支和原样写入的 `configs.file` 不算使用。
  一次性服务也按安装时的依赖顺序重新运行；无关服务和仅被重建服务依赖的服务不动。数据不动，正在用的连接会断开；选不出地址时插件标为异常，端口不改开到别的地址上，地址回来后恢复。用户改了设置也一样重建。
- **不做发现**：核心不发 mDNS、WS-Discovery、SSDP，也不转发组播和广播，用户按地址连。
- **身份由插件自己验证**：局域网的连接不经核心登录，用不用密钥、开不开鉴权由用户决定，核心和 `plugin check` 不拦。`webui` 的端口也可以列进来，这时连接绕过网关，没有 TLS；要不要用户打开开关见“网关代登录”和“应用自带的登录”。插件经 IPv4 地址看得到真实的客户端地址；用户选了 IPv6 地址时，
  那上面的连接由 docker-proxy 转发，插件看到的是网桥的网关地址。

### 第三方客户端的账号（`lan.client`）

要让用户把地址和账号填进第三方客户端（比如 Subsonic 播放器）的插件，写 `lan.client`。这样的账号由核心生成密码，用户不用先打开应用的网页。

```yaml
x-wonton:
  lan:
    ports:
      - { service: navidrome, port: 4533 }
    client:
      address: "http://{host}:4533"
      username: "{user}"
      password: "{secret:client_password}"
  secrets:
    client_password: { generate: { bytes: 24 } }
```

| 字段 | 说明 |
| --- | --- |
| `lan.client.address` | 客户端填的地址，必须写 `{host}`，核心换成端口开着的局域网地址（IPv6 带方括号）。开在几个地址上就给几个。 |
| `lan.client.username`、`password` | 值模板，写法和 `webui.proxy` 相同（`{user}`、`{email}`、`{secret:…}`，不能用 `{name}`）；`{secret:…}` 要引用核心生成、至少 16 字节的密钥。账号用 `{user}`，只有应用确实只能按邮箱认人时才写 `{email}`。 |

- 写了 `lan.client` 的插件，`lan.ports` 全部受开关 `lan_webui` 管：默认关，服务也不接 `--lan` 网络。
- 开关打开后，插件详情里显示地址和账号，可以复制。详情接口只给用户本人，Bot 和其他插件拿不到。
- 账号由插件自己在应用里建好，比如用一次性服务（见“一次性服务与首次写配置”）。用户在应用里改了密码，核心显示的不会跟着变，界面上有这句提示。

## Compose 子集

核心经 Docker API 渲染插件，只接受下面的字段，其余字段一律报错：

- 顶层：`name`、`services`、`volumes`（只支持不带参数的本地卷）、`configs`（见“插件包”）、`x-wonton`，
  以及任意 `x-` 扩展字段（如 `x-casaos`）；`version` 会被忽略。
- 服务：`image`、`command`、`entrypoint`、`environment`、`volumes`、`configs`、`restart`（未写时默认
  `unless-stopped`）、`healthcheck`、`depends_on`（含 `service_started`、`service_healthy`、
  `service_completed_successfully`）、`user`、`working_dir`、`labels`（`wonton.` 前缀保留给核心）、
  `hostname`、`expose`、`init`、`read_only`、`shm_size`、`tmpfs`、`stop_signal`、`stop_grace_period`、
  `mem_limit`、`mem_reservation`、`cpus`、`cpu_shares`、`pids_limit`（见“资源限制”）、
  `pull_policy`（默认 `missing`）、`network_mode`（`none`、`host` 或 `service:<服务名>`）、`cap_add`、`cap_drop`、`devices`、`pid`、`ipc`、
  `security_opt`、`privileged`，以及 `x-` 扩展字段。`image` 既没写
  标签也没写摘要时按 `latest` 拉取，和 compose 一致。`image:` 要直接写完整的
  `镜像:标签@sha256:…` 引用，不用 YAML 锚点或别名（`&name`、`*name`）：自动升级镜像的机器人认不出别名里的
  镜像，升级 PR 开不出来；同一镜像在几个服务里用就各写一遍。
- 卷：短语法和长语法（`bind`、`volume`、`tmpfs`）。宿主路径只能是 `${WONTON_APP_DATA}`（或其子目录）和
  已授权共享文件夹的根目录，不能含 `..`，不支持相对路径；同一插件的绑定挂载源不能嵌套（同一个目录可以挂进
  多个服务）；命名卷必须在顶层 `volumes` 声明。
- 命名卷在 Docker 里叫 `wonton-<slug>--<卷名>`。同名卷已存在时：属于本插件（上次卸载时保留的数据）就
  直接复用；属于其他插件或不是核心创建的，拒绝安装。安装失败回滚时只删除本次新建的卷，保留的卷不删。
- `network_mode: service:<服务名>` 让这个服务借用同一插件里另一个服务的网络（同一个网络命名空间，
  `127.0.0.1` 互通）。上游应用只监听回环地址时，用它配一个独立的代理容器对外，不用合并镜像、不改上游源码。
  不算高权限，不用声明权限。
  - 被借用的服务不能再写 `network_mode`：不能链式借用，不能借用 `host` 或 `none` 的网络。
  - 借用的服务不能写 `networks`、`ports`、`hostname`、`domainname`、`dns`、`dns_search`、`extra_hosts`、
    `mac_address`、`expose`，写了空列表也不行：这些都由被借用的服务决定。
  - `webui.service`、`mcp.service`、`lan.ports[].service` 写被借用的服务，端口是借用的服务在这个网络里监听的端口。
  - 借用的服务自动依赖被借用的服务，先启动后者；被借用的服务重建时（如开关局域网访问），借用的服务跟着重建。
  - 两个服务要么都是 Bot 服务，要么都不是。

- 所有插件容器默认带 `no-new-privileges:true`，核心写进 HostConfig，不需要声明。依赖 setuid 的镜像
  （容器里用 sudo 的）写 `security_opt: [no-new-privileges:false]` 退回 Docker 默认，这一项不算权限。
- 高权限字段按服务生效，必须和 `x-wonton.permissions` 里的权限一一对应：
  - `network_mode: host` 对应 `network.host`。这个服务不能出现在 `lan.ports` 里；`webui` 指向它时要写健康检查，不能用 `sso: proxy`（原因见“网关代登录”）。它连不到插件网络里的服务，包括同一个插件的：宿主不能直接访问插件网络，这是隔离承诺（见“网络”）。要和插件网络里的服务通信就反过来：让插件网络里的服务去连它开在宿主上的端口（Beszel 的做法：Agent 只在宿主网络里开 SSH 服务，Hub 主动拨号），或者整个插件都用宿主网络。`webui` 或 `mcp` 指向它时，应用要能从 `127.0.0.1:<端口>` 连上：核心给每个入口加一个中继容器（服务名 `relay-webui`、`relay-mcp`，插件自己不要用这两个名字），经 `${WONTON_APP_DATA}/relay` 里的 unix 套接字转发到这个端口，插件不要往这个目录里放东西。
  - `cap_add` 的每个值对应 `cap.<能力>`；`cap_drop` 只减不加，不需要权限。两者的值都按能力名写，小写规范成大写，`ALL` 也收。核心把插件的 `cap_drop` 和宿主网络默认去掉的 `NET_RAW` 合并去重，写了 `NET_RAW` 或 `ALL` 就不再重复。
  - `devices` 对应 `devices.<路径>`，写成 `源:目标[:权限]`，源只能是 `/dev/net/tun` 或 `/dev/fuse`，目标是绝对路径，权限由 `r`、`w`、`m` 组成。显卡不写在这里，由核心按 `gpu.services` 交给服务（见“显卡”）。
  - `pid: host`、`ipc: host` 对应 `host.pid`、`host.ipc`，这两个字段只接受 `host`。
  - `security_opt` 里的 `no-new-privileges` 写法是例外：写 `no-new-privileges` 或 `no-new-privileges:true` 等于默认的加固，写 `no-new-privileges:false` 退回 Docker 默认，都不算权限；其他写法对应 `security.opt`，可以和退回默认在同一个服务里同用。`privileged: true` 对应 `privileged`。
  - `/var/run/docker.sock:/var/run/docker.sock` 这一条卷对应 `docker.socket`，只能这样写，不能只读。
- 明确拒绝并给出原因：`ports`（局域网端口写在 `lan.ports`，见“局域网”）、`network_mode`（`none`、`host`、`service:<服务名>` 以外的取值）、`networks`、
  `container_name`、`build`、`env_file`、`extends`、`links`、`external_links`、`volumes_from`，以及顶层的
  `networks`、`secrets`。
- `environment` 里只有键没有值的写法（取宿主环境变量）不受支持。

安装时核心按 `depends_on` 顺序启动容器，等每个长期运行的容器健康（有健康检查时）或稳定运行、每个一次性
服务以 0 退出，再确认网页入口能响应，全部通过才算安装完成；任一步失败都按相反顺序回滚已创建的资源，并在
错误里附上容器最近的日志。

## 健康检查

- 长期运行的服务都要有健康检查，而且检查的是真正可用，不只是进程还活着：Hermes 要 Matrix 已经连上
  （用它自己的 `/health/detailed`），magpie 用自带的 `magpie healthcheck`。
- 写在 compose 的 `healthcheck` 里。镜像自带的 `HEALTHCHECK` 也算数，但官方索引的插件要在 compose 里
  显式写出，便于审核，也随插件版本锁定。
- 写上 `start_period` 和 `start_interval`：安装时能尽快判断就绪，启动慢时又不会误判失败。
- 检查只能用镜像里已有的命令，distroless 之类的镜像里没有 `curl`、`wget`。
- 官方索引的插件缺健康检查时不收录（`wonton-core plugin check --official` 检查）。导入的应用和社区插件
  没有健康检查时照常安装，状态如实显示为“运行中，健康未知”，不显示“健康”。
- 核心在安装、每次启动时检查，运行期间每 30 秒再看一次容器，插件列表里运行中的插件带上最近一次的结果：每个
  长期运行的服务都有健康检查而且都通过了是 `healthy`；有服务没有健康检查，或者检查还没通过，是 `unknown`（只
  知道在运行）。
- 检查失败的插件标为异常；容器好了，核心重新核对一遍，插件恢复运行。卸载中途失败、记录损坏这些和容器无关的异常
  不会自己恢复。
- 一次性服务不需要健康检查：核心只看它的退出码，镜像自带的 `HEALTHCHECK` 失败不影响它（复用主镜像
  `image: *image` 时会一直失败，不用写 `healthcheck: test: ["NONE"]` 去关）。

## 网络

- 每个插件有独立的 Docker 网络 `wonton-<slug>`，容器叫 `wonton-<slug>--<服务名>`，服务之间用服务名互访。服务名是
  合法的 DNS 标签：只能有小写字母、数字和 `-`，不能含 `--`，不能以 `-` 开头或结尾。每个服务在插件网络里还有一个
  别名 `<服务名>.<slug>.wonton.internal`（如 `mcp.app-immich-photos.wonton.internal`），核心访问插件的网页、MCP
  和内网资源时都用它当 Host；宿主网络的服务照旧走中继。别名和实例名无关：网络本来就按实例分开。核心加入所有插件网络并负责
  路由。插件之间、插件和 Tuwunel、插件和模型网关都不共用网络。开发环境的核心设了实例名（`WONTON_INSTANCE`）时，网络、
  容器和命名卷的名字在 `wonton-` 后多一段 `<实例名>--`。
- 核心以低于默认的网关优先级（`gw_priority: -1`）接入插件网络，默认路由和发布的端口始终在核心自己的网络上。
  容器接了几个网络时，Docker 让优先级最高的端点提供默认路由和发布端口，一样高时取网络名按字典序排第一的，
  而插件网络排在核心的网络前面（`wonton-` 对 `wonton_`）。不压低的话，核心的出口跟着能出网的插件网络走，
  卸载这个插件时经发布端口的连接全部断开（#90）。
- 核心在插件网络里以带点的别名提供下面几个入口，每个别名只出现在需要它的插件网络里。核心按请求从哪个
  插件网络进来判断来源，只转发它有权访问的入口。`.internal` 是 ICANN 保留给私有网络的顶级域，不会和
  公网域名冲突。

  | 别名 | 出现条件 | 转发到 |
  | --- | --- | --- |
  | `auth.<基础域名>` | `webui.sso: oidc` | 身份提供方的公开端点；浏览器和插件看到同一个签发方地址。 |
  | `models.wonton.internal` | `models` | 模型网关声明的服务和端口。 |
  | `matrix.wonton.internal` | `matrix.bot` | Tuwunel 的 Matrix 客户端接口。 |
  | `mcp.wonton.internal` | `tools` | 核心的 MCP 总线。 |
  | `health.wonton.internal` | `plugins.health` | 只读的健康接口，见“插件健康探测”。 |

  不让声明 `models` 的插件直接加入模型网关的网络：那样这些插件彼此也连通了，还能访问网关容器的所有端口。
  经核心转发只多一跳（核心网关的 p50 约 0.27 ms），却守住了插件之间的隔离，核心也能按插件
  限流和审计。Matrix 同理。
- 出网：没有 `network.egress` 的插件网络是 Docker 的内部网络（`internal: true`）：不能访问外网和局域网，
  也解析不了外部域名（Docker Engine 26 起不再为只连内部网络的容器转发外部 DNS 查询，CVE-2024-29018），
  上面的入口照常可用。网桥还设成 `com.docker.network.bridge.gateway_mode_ipv4/ipv6=isolated`（Engine 28 起）：
  网桥在宿主上没有地址，插件连宿主自己监听的服务也访问不到，宿主也不能直接访问插件，核心经它加入的插件网络
  访问插件。声明了 `network.egress` 时插件网络是普通的桥接网络。`network.egress` 是一个开关：不区分外网和局域网，
  也不按服务细分。
  - 核心直接跑在宿主上（开发时，没设 `WONTON_CORE_CONTAINER`）要经网桥访问插件，所以不设 `isolated`，
    插件还能访问宿主上监听所有地址的服务；插件经自己网络的网关访问核心的身份提供方入口（内部网络没有
    默认路由，`host-gateway` 走不通）。
  - 核心启动核对插件时检查它的网络：和清单、核心的运行方式不符时（比如核心升级前装的插件网络能出网）把
    插件标为异常，提示重装（可以不删数据）。

  > **已定：默认不能出网**。要出网的插件声明 `network.egress`，安装时列在权限里：
  > 相册、文档这类插件连不上外网，就带不走数据，符合“住在自己家里”；代价是多一项确认。
  > 没选的方案是默认能出网（Docker、CasaOS、Umbrel 都是这样）：兼容最好，但插件不声明也能把
  > 数据发出去，只能靠审核。
- 核心的地址：每个插件网络的最后一个可用地址留给核心，容器自动分配只用网段的前一半，核心容器重建后地址不变（`WONTON_CORE_ADDRESS`）。
- 走代理：要走代理才能出网的主机，用户配一次代理地址，插件不用管。出口容器 `wonton-egress` 接在每个出网插件的网络上，别名 `egress.wonton.internal`，是这些网络里除核心和插件服务之外唯一的容器。连宿主时用网关地址或局域网地址都行，和不配代理时一样。
  - 核心给声明了 `network.egress` 的插件的每个服务（写了 `network_mode: none` 或 `host` 的除外：`none` 不出网，`host` 走宿主自己的路由，不经出口容器；`service:<服务名>` 照常写）写好 `HTTP_PROXY`、`HTTPS_PROXY`、`http_proxy`、`https_proxy`、`NO_PROXY`、
    `no_proxy` 和 `NODE_USE_ENV_PROXY=1`。地址是出口容器 `http://egress.wonton.internal:3128`，插件看不到用户的代理。`NO_PROXY` 已经包含回环、
    `.wonton.internal`、`auth.<基础域名>`、本插件的服务名和 IPv4 私有网段（10/8、100.64/10、169.254/16、172.16/12、192.168/16；插件网络和
    `--lan` 网络的网段都在这些私有网段里。IPv6 网段条目会把 httpx 这类读了 `NO_PROXY` 的客户端建连接时弄崩，所以不写）。没配代理时这些变量都不写。这是核心唯一直接写进容器环境的一组变量。
  - compose 里写了同名代理变量时以 compose 为准，写成空就是这个服务不走代理。写了 `NO_PROXY` 或 `no_proxy` 时核心不覆盖，把自己名单里缺的条目并到后面：作者写的名单常常只有自己的服务，少了核心的条目，插件内部的请求会被送去用户的代理，上游解析不了插件的服务名，得到 502 一类的错。不要写具体的代理地址：官方索引不收，换一台主机也用不了。
  - 镜像自己带了代理变量（`docker inspect --format '{{.Config.Env}}' <镜像>` 能看到）时，在 compose 里把它们写成空（`HTTP_PROXY: ""` 这样，核心不注入的名字也一样），或者从镜像里去掉。不然这些变量原样进容器：没配代理、或插件没声明 `network.egress` 时核心什么都不注入，应用照着镜像里的地址走，同插件的内部请求也发给那个代理。声明了 `network.egress` 且配了代理时，核心注入的变量盖掉镜像里同名的，核心不管的名字（如 `ALL_PROXY`）原样保留。
  - 程序不读这些变量但有自己的代理设置（SearXNG 的 `outgoing.proxies`、Java 的 `-Dhttp.proxyHost`）时，在配置里引用 `${WONTON_PROXY_URL}`。
  - 健康检查不走代理：核心改写调用 wget 和 shell 的写法，包括镜像自带的 `HEALTHCHECK`。入口脚本里的 busybox wget 不看 `no_proxy`，访问内部地址要自己加 `-Y off`。
  - 插件网络不是信任边界：用户的代理放行私有地址时，别的出网插件能经它连到出网插件和 `--lan` 网络上的服务的任何端口。建议：应用支持密码或密钥的内部服务（数据库、Redis），用 `secrets` 生成并设上。不支持的（比如 Ollama）不因此拒收，`lan.ports` 里的端口也不在此列。
  - 经代理出错时：403 是出口容器拒绝了这个目标（内部或私有地址走到了代理）；503 是连不上用户的代理（立刻返回是被拒绝，等 15 秒是没响应）；CONNECT 200 之后 TLS 断开是代理连不上目标。
  - 出网失败先查代理配置。DNS 被污染时域名解析到代理或内网地址，应用自带的 SSRF 防护会拦下来，报 internal address 一类的错。
- 代理插件：没有自己代理的用户可以装一个代理插件，在出网代理的设置里选“使用已安装的代理插件”。插件这样声明：

  ```yaml
  x-wonton:
    provides:
      egress_proxy: { service: proxy, port: 7890 }
    permissions: [network.egress, egress.proxy]
  ```

  - `service` 是提供代理的服务，要在 `services` 里，不能写 `network_mode`，不能是 Bot 服务；`port` 是支持 `CONNECT` 的 HTTP 代理端口。
    `provides.egress_proxy` 和 `egress.proxy` 必须成对出现，还要申请 `network.egress`。不限 `kind`。
  - 一台服务器只能装一个，装第二个时安装失败（`conflict`）。装上不会自动被选中。
  - 选中后，出口容器把请求转给 `http://wonton-<插件>--<服务>:<port>`（插件网络里的容器名）。这个端口不用、也不要发布到宿主。
    代理插件自己的服务不写代理变量，直接出网；装、升级、重装它时不检查出网代理，没有代理也装得上，所以它的镜像要能直接拉到。
  - 代理插件没在运行时，出网插件得到 503，不会改成直连；别的出网插件装不上，错误写明代理插件不可用。正在用的代理插件
    卸载失败（`conflict`），要先在设置里换掉。
  - 管理接口一定要设密钥（`secrets.generate`）：别的出网插件也能把目标写成代理插件的容器名，经代理到达它。
  - 给局域网里的其他设备用，照“局域网”的 `lan.ports` 加 `lan.client` 开端口，默认关。经出口容器来的连接来自 Docker 的私有网段，
    不要对它们要求账号。
- 核心要求 Docker Engine 28 及以上，并且不开 `allow-direct-routing`：28 起局域网里的其他主机不能再经路由
  直接访问容器里没发布的端口（moby#48724），插件网页才真的只能经核心网关访问；`sso: proxy` 信任网关加的
  请求头，靠的就是这一点。核心启动时检查 Engine 版本，低于 28 时报错退出，不降级运行；
  `allow-direct-routing` 经 Engine API 查不到，由安装器保证。
- 插件之间默认互不可达；需要互通时在清单里声明依赖（待定）。
- 插件不向宿主暴露端口，只有 `network.lan` 的插件把 `lan.ports` 列出的端口发布在局域网地址上（见“局域网”）；
  网页一律经核心网关访问。声明 `network.host` 的服务例外：它用宿主网络，端口由应用自己开在宿主上。只监听 `127.0.0.1` 时局域网绕不过网关；监听所有网卡时局域网设备也能直连，本机开着防火墙时连不进来，要用户自己放行。网关和 MCP 总线经中继连它，不用改防火墙。核心装好后直连一次网页端口，连不上时详情页提示局域网设备可能连不进来，只监听 `127.0.0.1` 的应用也可能看到这条提示。
- 插件的容器、网络和卷都带标签 `wonton.plugin=<id>`，卸载时据此清理；卸载时选择删除数据才会删除
  命名卷、应用数据目录和插件密钥，共享文件夹永不删除。
- 安装和卸载不随请求取消：请求断了（手机切网、隧道断开），操作照样做完，结果看插件列表里的状态。同一个插件正在
  安装或卸载时，对它的新请求得到 409 `conflict`，说明正在安装或卸载。核心升级没结束时，安装、升级和卸载都得到 409
  `core_upgrading`。
- 核心在安装半路重启，或者回滚没做完，会留下没有安装记录的容器和网络。再装时核心按标签（这个插件，实例名和自己相同）
  删掉它们，吊销上次登录的 Bot 设备，然后从头装；命名卷、应用数据目录和密钥接着用。这些网络上还接着别的容器（另一套
  核心、用户的容器）时什么都不删，拒绝安装（`conflict`）。同名的网络属于其他插件或不是核心创建的，也拒绝安装，不接管
  也不删。
- 一台 Docker 上跑几套核心时，每套都要设自己的实例名（`WONTON_INSTANCE`）。没有实例名的两套，标签上分不出来：对方的
  核心还接在网络上时会被上面的检查挡住，已经停了的话，它的插件和数据都会被当成自己的。

## 发布与来源

### 插件仓库

- 一个插件一个 Git 仓库，仓库根目录就是插件包：`docker-compose.yml` 和它引用的文件。仓库里的其他文件（README、没有被
  `notices` 引用的 LICENSE、Dockerfile、源码、截图、测试）核心不取，截图只在商店的详情页里显示。
- 发布：把清单的 `version` 改成新版本，提交，打 tag `v<version>` 推上去。tag 和清单的版本不一致时装不上。带预发布后缀的
  版本（如 `v1.2.0-beta.1`）只能按 tag 明确安装，不进商店，也不作为升级提示。发布过的 tag 不要移动：已经装上的不受
  影响，但核心会提示用户这个 tag 被改过。
- 插件自己的镜像由作者构建，推到公开的镜像仓库，compose 按摘要引用（`名字:标签@sha256:…`，多架构镜像写清单列表的
  摘要）。核心不构建镜像。
- 公开仓库加上 topic `wonton-plugin`，几分钟后就出现在商店的社区插件里，标为“未审核”。想进官方商店，向官方索引提 PR
  （见“官方索引”）。

### 来源

| 写法 | 装的是 |
| --- | --- |
| `github:<owner>/<repo>@v1.2.0` | 这个 tag |
| `github:<owner>/<repo>` | 最新的正式版本：版本号最大的、不带预发布后缀的 `v<语义化版本>` tag |
| `github:<owner>/<repo>@<40 位提交号>` | 这个提交，开发时用；清单的版本照常校验，不检查升级 |
| `index+<索引地址>#<插件 ID>` | 用户添加的第三方索引里的这个插件（见“第三方索引”） |
| `upload:sha256:<插件包摘要>` | 刚上传、核心暂存着的插件包（见“上传插件包”） |

- 核心经 git 协议 v2 的 `ls-refs`（`https://github.com/<owner>/<repo>.git`）把 tag 解析成提交，之后按提交取文件：先取
  `docker-compose.yml`，再取它引用的文件。取到后和上传的插件包走同一套解析和校验。
- 私有仓库核心取不到（核心不存 GitHub 令牌），开发时从本地目录上传（见“开发与校验”）。
- 安装记录写下来源、tag、提交和插件包。

### 插件包摘要

插件包的每个文件一行 `<sha256 的十六进制>  <路径>`（两个空格），按路径的字节序排列，每行以换行结尾，即 `sha256sum`
的输出格式；路径相对包的根目录，不带 `./`，`docker-compose.yml` 也在内。这段文字的 sha256 就是插件包摘要，写成
`sha256:<十六进制>`。它只取决于文件的内容，和文件从哪里取、怎么打包无关；`wonton-core plugin check` 会打印它。

安装和升级的预览返回摘要，安装请求必须带回同一个摘要，对不上时得到 409 `changed`：用户确认的就是装上的。

### 镜像

- 官方索引的插件，所有服务的 `image` 都按摘要写。
- 用户打开镜像加速后，Docker Hub 上按摘要固定的镜像由核心从加速地址按同一个摘要拉，Docker 核对摘要，内容对不上就拉取失败；
  只写标签的镜像和别的仓库照旧直连。所以 Docker Hub 上的镜像要按摘要写才能经加速拉到。经加速拉到的镜像，容器的镜像名
  可能显示成加速地址的引用。
- 社区插件和上传的插件包可以只写标签，预览时如实提示“镜像没有按摘要固定”。升级失败回滚时用旧容器当时的镜像，不再按
  标签拉：标签可能已经被新版本的拉取移动了。

### 官方索引

公开仓库 `geraldpeng6/wonton-store` 的 `index.json`：

```json
{
  "format": 1,
  "plugins": [
    {
      "source": "github:geraldpeng6/wonton-plugin-files",
      "version": "0.2.0",
      "commit": "<40 位提交号>",
      "files": {
        "docker-compose.yml": "<sha256 的十六进制>",
        "icon.svg": "<sha256 的十六进制>"
      },
      "compose": "<docker-compose.yml 的原文>"
    }
  ],
  "onboarding": {
    "agents": ["com.nousresearch.hermes"],
    "needs": [
      {
        "id": "photos",
        "title": { "zh": "备份照片", "en": "Back up photos" },
        "plugins": ["app.immich.immich"]
      }
    ],
    "providers": {
      "zh": [{ "id": "deepseek", "key_url": "https://platform.deepseek.com/api_keys" }],
      "en": [{ "id": "openrouter", "key_url": "https://openrouter.ai/keys" }]
    },
    "prompts": [{ "zh": "你能帮我做什么", "en": "What can you help me with" }]
  },
  "docs": {
    "source": "github:geraldpeng6/wonton-docs",
    "commit": "<40 位提交号>",
    "min_core": "0.9.0",
    "files": { "user/zh/devices.md": "<sha256 的十六进制>" }
  }
}
```

- `format` 是索引格式的版本，核心读不懂的格式按取不到处理，并提示升级核心。
- 一个插件一项，`source` 和插件 ID 都不重复，商店按这里的顺序列出官方插件，标为“已审核”。`files` 是插件包的全部文件，`compose` 必须
  和 `files` 里的 `docker-compose.yml` 一致。核心按 `files` 核对取到的每个文件，所以文件可以从任何地方取（GitHub、
  jsDelivr）。
- `onboarding` 是 App 的新用户引导用的清单，四个键都要有。`agents` 是可选的 Agent，按顺序列出，不在这份索引里或者不是 `kind: agent` 的跳过；`needs` 是用户能选的需求和对应的插件，不在这份索引里的插件跳过；`providers` 是按语言推荐的模型服务商和拿 key 的地址，网关 `presets` 里没有的不显示；`prompts` 是第一次聊天时给用户点的问题。这个键读不懂只影响引导，不影响商店。
- `docs` 是帮助文档的版本。`source` 不带版本，`files` 最多 500 个文件。核心按 `commit` 取 `files` 列出的文件，全部核对通过后才换用。`min_core` 比这台核心新时不换，继续用手头的。这个键读不懂只影响帮助文档。
- 条目用 `wonton-core plugin index-entry <插件包目录> github:<owner>/<repo> <提交号>` 生成，不手写：目录是那个提交的
  检出（核心不连 Git，不核对），条目的 JSON 写到标准输出；加 `--index <索引目录>` 时直接写进本地的索引目录，同一个
  来源的旧条目换掉。
- 收录要求：`wonton-core plugin check --official` 通过，即有 `icon`、`tags`、`license`、`author`、`notices`，`description`、`tags`、`about`、`release_notes` 都写了 `zh` 和 `en`，所有服务的 `image`
  按摘要固定，长期运行的服务都有健康检查；申请的权限和插件做的事相符；镜像来自上游官方或作者可复现的构建。
- 核心不内置索引，从 GitHub 取，取不到时用磁盘上次成功的那份，并说明列表是什么时候的、为什么没更新。开发镜像和测试用本地的索引目录代替（`WONTON_STORE_INDEX`）：`index.json`
  加上按 sha256 存放的文件（`blobs/<sha256 的十六进制>`）。

### 第三方索引

任何人都可以发布自己的索引，用户添加后，其中的插件出现在商店里，标为“未审核”。

- 格式和官方索引相同，条目也用 `wonton-core plugin index-entry` 生成。顶层可以多一个 `name`（字符串），显示在商店里。
- 放在 GitHub 仓库根目录的 `index.json`，或者任何 https 地址。不支持 http。
- 核心读默认分支或这个地址上最新的内容。文件先从索引旁边的 `blobs/<sha256 的十六进制>` 取，取不到再按条目的提交从 GitHub
  取，都按 `files` 里的 sha256 核对。只放在 GitHub 上时不需要 `blobs/`。
- 从索引装的插件按这个索引里的版本提示升级。用户删掉索引或者索引不再收录时不再提示，也不改去看 GitHub 的 tag。

### 添加来源

用户在商店里填一个地址，核心判断它是插件还是索引，记下来，以后在商店的“我添加的”里列出：

| 用户填的 | 是什么 |
| --- | --- |
| `https://github.com/<owner>/<repo>`，可带 `.git`、`/tree/<tag>`、`/releases/tag/<tag>`、`/commit/<提交>` | 按下面判断 |
| `github.com/<owner>/<repo>`、`<owner>/<repo>`、`github:<owner>/<repo>[@<版本>]` | 按下面判断 |
| 仓库默认分支的根目录有 `docker-compose.yml` | 插件；没有正式版本 tag、地址里也没写版本时加不上 |
| 仓库默认分支的根目录有 `index.json` | 第三方索引 |
| 其他 https 地址 | 第三方索引，解析不了就加不上 |

添加的仓库和第三方索引都标为“未审核”。删掉来源不影响已经装上的插件。

### 上传插件包

用户可以在 App 里选一个压缩包安装（格式见“插件包”）。上传后核心校验并暂存 1 小时，预览的来源是 `upload:sha256:<摘要>`，
确认时带回；过期了要重新上传。预览显示插件包摘要，两个人可以用它核对收到的是不是同一个包。

校验和其他来源相同：清单、规范版本、权限，`kind: system` 拒绝。压缩包不带镜像，预览列出镜像来自哪些仓库。

### 社区插件

- 公开仓库、topic `wonton-plugin`、有 `v<语义化版本>` tag：核心按 GitHub 星数列出，标为“未审核”。
- 核心读最新正式版本的 `docker-compose.yml`；没有 tag、清单解析不了的仓库不列出。官方索引里的仓库、插件 ID 和官方插件
  相同的仓库也不列出。
- 列表缓存 10 分钟。用户可以关掉社区插件（`WONTON_STORE_COMMUNITY=off`）。

### 升级

- 官方插件按官方索引里的版本、社区插件和添加的仓库按仓库最新的正式版本 tag、第三方索引的插件按那个索引里的版本提示升级；
  上传的和按提交装的不提示。随核心预装的默认插件没有来源，按插件 ID 在官方索引里找，升上去之后就是官方来源。提示只升不降。
- 用户可以主动换版本：传一个新的压缩包，或者填一个来源（`POST /_wonton/api/plugins/{id}/upgrade/preview`），和升级走同一个
  确认页。比装着的旧时醒目提示，旧版本可能读不了迁移过的数据；来源变了时也醒目提示，之后按新来源提示升级，信任标签也跟着变。
  插件包摘要和来源都和装着的相同时不用换，内容相同但来源不同时可以换。
- 按 tag 从仓库装的插件，官方索引后来收录了同一个仓库的同一个提交时，核心检查新版本时把它标为“已审核”，用户不用重装。
  收录的是别的提交时保持“未审核”，用户升级到官方索引里的版本后才变。
- 用户手动刷新商店、打开升级预览时，核心不等缓存过期，重新取官方索引（一分钟内不重复取）；没取到时报错，不说“没有新版本”。
- `id` 不能变。升级保留数据、插件密钥、Bot 账号和设备、模型凭据、OIDC 客户端，只换容器。Bot 的令牌从旧容器读回（见
  “Matrix Bot”），读不到就不升，不签新设备。
- 新版本的镜像先拉好再动旧容器，拉不下来旧版本不受影响。换容器之后新版本起不来、没通过健康检查，就回滚到旧版本的容器，
  错误里写明退回了哪个版本。
- 新版本申请的权限和装着的不同时，用户要重新确认全部权限，新增的标出来。
- 数据迁移由应用自己在启动时做；不可逆的迁移要在 `release_notes` 里写明，回滚后旧版本可能读不了迁移过的数据。
- Bot 更新自己写的插件不一定要用户确认，见“Agent 写插件”。

### 开发与校验

- 校验：`wonton-core plugin check <目录>`，发布的核心镜像里就有。和安装时同一套解析和校验，加上商店信息和图片的检查，
  最后打印插件包里的文件和插件包摘要；`--official` 再按官方索引的要求检查。不连 Docker，也不联网。
  - 插件包解析不了时只报这一条（和安装时报的一样）；解析得了时列出所有问题。
  - 不合格（退出码 1）：装不上的（核心还不支持的权限、渲染不了的 compose），图标或截图处理不了。
  - 提醒（退出码 0）：镜像没有按摘要固定，长期运行的服务没有健康检查，没有图标，图标的角是透明的，截图的长边不到
    1280，横屏和竖屏的截图混在一起，`kind: system`。`--official` 时前三项也算不合格，另外要有 `tags`、`license`、
    `author`、`notices`，`description`、`tags`、`about`、`release_notes` 要写 `zh` 和 `en`。只看得到 compose 里写的健康检查，镜像自带的看不到。
  - 用法不对时退出码 2。
- 打包：`wonton-core plugin pack <目录>` 把插件包打成 tar 写到标准输出，只有 compose 文件和它引用的文件，README、截图
  这些不在里面；解析不了时和 `check` 一样报错，退出码 1。
- 开发时从本地目录安装：把 `pack` 打出的 tar 上传到安装接口（见“插件包”），或者在 App 里选这个文件。装上后不检查升级。改了以后
  上传到升级接口（`POST /_wonton/api/plugins/{id}/upgrade`），保留数据。
- 模板仓库 `geraldpeng6/wonton-plugin-template` 带开发环境和检查、发布的脚本。开发环境用发布的核心镜像在本机起一套
  核心，和正式安装是同一份 compose，换掉项目名、核心的容器名（`WONTON_CORE_CONTAINER`）和实例名
  （`WONTON_INSTANCE`），几套开发环境可以共用一台 Docker，同一个插件也能同时装在几套里。

### Agent 写插件

用户在聊天里说想要什么，能调工具的 Agent 插件（申请了 `tools`）可以写一个插件，装到这台机器上。说明在
[`skills/wonton-plugin`](skills/wonton-plugin/SKILL.md)，核心也经 `wonton__plugin_guide` 给同一份。

1. Agent 把插件目录写在自己的 outbox（`${WONTON_APP_DATA}/outbox`，见“Bot 存文件”）。
2. 调 `wonton__check_plugin`，`dir` 是 outbox 下的相对路径，改到没有 `problems`。这一步不打扰用户。
3. 调 `wonton__install_plugin`，`dir` 同上。校验不过时不发卡片，返回 `invalid_input` 和全部问题。过了的话用户在确认卡片上看全部权限和镜像，批准后才装。失败时回滚，错误里带容器日志。
4. 要改时改目录，再调一次：这次是更新，数据保留，失败回滚，版本号不用改。插件包摘要没变时返回 `invalid_input`，写明没有变化。

规则：

- 核心读 `dir` 时逐级不跟软链接，只收普通文件，校验和装的是读到的同一份，之后再改目录不影响正在确认的那一份。目录不会被删。
- 核心不构建，Agent 也没有 Docker。写法只有一种：公开镜像加 `configs`，代码在插件包里，受插件包的大小限制。
- 核心记下插件是哪个 Bot 写的。只有写它的 Bot 能更新。`dir` 里的 `id` 已被别的插件占用时返回 `invalid_input`。用户用文件升级后，插件不再归 Agent。
- 同一个 Bot 对同一个插件再次提交时，还在等的旧卡片作废，旧调用收到“被取代”的错误。

| 操作 | 确认卡片 |
| --- | --- |
| 第一次装 | 每次都发，高权限醒目标出，以前留下的数据和密钥也写明。最高级权限多一张。 |
| 更新，新增了权限、局域网端口或拿到显卡的服务 | 发，只列新增的项。 |
| 更新，申请了很高或最高级权限 | 每次都发。最高级权限多一张。 |
| 更新，用户打开了“每次都问” | 每次都发。 |
| 其他更新 | 不发，写审计。 |
| 卸载 | 每次都发。 |

换镜像不算新增。减少权限时按新清单授予。

## 待定问题

- 插件间依赖与通信的声明方式。
- 官方索引的签名（minisign），签名后索引也能经镜像和国内的静态托管分发。
- 一个仓库放多个插件（`github:<owner>/<repo>/<路径>`）。
- 快捷方式（`shortcuts`）的写法。
- `settings.schema.json` 支持的 JSON Schema 子集。
- 插件升级时的数据迁移钩子。
- 共享 Bot的白名单：多个用户，而且会在运行中变化，环境变量表达不了。
- 多 Bot 时的存储授权：共享文件夹是插件级的授权，家人的私人空间（`storage.home`）要按 Bot 的用户分别
  挂载。
- `network.lan` 能不能和 `matrix.bot` 一起申请：Bot 的令牌能从旧容器读回（见“Matrix Bot”）后，地址变了也能重建。
- 插件密钥在 App 里的查看（如应用的管理员密码）与轮换；收存的秘密在恢复数据时怎样交还插件。
- compose 的 `secrets`：以文件形式交给容器的密钥，给遵循 `*_FILE` 约定的镜像用。

