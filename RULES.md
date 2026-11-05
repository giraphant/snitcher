# 规则维护清单

这份清单描述当前发布规则。网络历史与连接计数留在本地；频率用于安排审查顺序，具体用途和已授权工作流决定是否放行。

## 模式

四个订阅组配合使用：`baseline` 放行已审阅目标并对其余目标 Ask；`deny-telemetry` 与 `deny-derp` 拒绝已确认的非必要目标；`default-deny` 把兜底从 Ask 改成 Deny，构成白名单出口。Little Snitch 在同优先级、同具体度下按 Deny > Allow > Ask 判定，所以启用 `default-deny` 后其“任意目标拒绝”压过 baseline 的“任意目标询问”，而任何主机、域名或端口范围放行仍更具体、仍然生效。停用 `default-deny` 即回到 Ask。

规则按类别分组：每个程序每个类别一条规则，类别内的主机或域名写成列表。Little Snitch 的 `process` 字段只能写一个程序，所以同一类别会按程序重复出现。

| 程序 | 签名身份 |
|---|---|
| Rex 客户端 | `identifier.AG92C3PT77/com.superlogical.rex` |
| Rex Server | `identifier.AG92C3PT77/com.superlogical.rex.server` |
| Claude Code | `identifier.Q6L2SF6YDW/com.anthropic.claude-code` |
| Codex | `identifier.2DC432GLL2/codex` |
| Orca（含 Orca Terminal Host） | `identifier.6CX3WHS9HZ/com.stablyai.orca` |
| Antigravity CLI（agy 独立身份） | `identifier.EQHXZ8M8AV/cli` |

Rex Server 与 Orca 是终端宿主，在其中运行的 claude、codex、gh、curl、node、agy 等子进程可能由 Little Snitch 以“宿主 via 工具”归属到它们，也可能被归为独立身份。2026-11-04/05 的规则显示 agy 以独立身份连接；本机签名校验确认其开发者为 Google LLC，标识符为 `cli`。订阅同时覆盖 agy 的独立身份与终端宿主归属。宿主放行规则只在工具本身有有效开发者签名时才覆盖它；ad-hoc 签名或未签名的工具不适用，见“已知缺口”。下文“四个程序”仍指 Rex Server、Claude Code、Codex、Orca，独立 agy 未继承这些程序的其他站点类别。

## 放行：平台服务（精确主机，TCP 443）

平台的兄弟子域常有其他用途（如 `gist.github.com`、`ab.chatgpt.com`），因此只放行列出的主机。

| 类别 | 程序 | 主机 |
|---|---|---|
| GitHub | 四个程序 | `github.com`、`api.github.com`、`raw.githubusercontent.com`、`codeload.github.com`、`objects.githubusercontent.com`、`release-assets.githubusercontent.com` |
| 包管理 | 四个程序 | `registry.npmjs.org`、`ghcr.io`、`pkg-containers.githubusercontent.com`、`formulae.brew.sh` |
| Anthropic | Claude、Rex Server、Orca | `api.anthropic.com`、`mcp-proxy.anthropic.com`、`platform.claude.com`、`claude.ai`、`downloads.claude.ai` |
| OpenAI | Codex、Rex Server、Orca | `chatgpt.com`、`auth.openai.com`、`api.openai.com` |
| 终端工具服务 | Rex Server、Orca | `daily-cloudcode-pa.googleapis.com`、`aicode.googleapis.com`、`oauth2.googleapis.com`、`www.googleapis.com`（Antigravity CLI）；`api.linear.app`（Linear） |
| Antigravity CLI | agy 独立身份 | `daily-cloudcode-pa.googleapis.com`、`aicode.googleapis.com`、`oauth2.googleapis.com`、`www.googleapis.com`（模型、登录与用户信息） |
| Orca 服务 | Orca | `onorca.dev`、`login.onorca.dev`、`push.onorca.dev` |
| Rex 内置 Tailscale 与证书 | Rex Server | `controlplane.tailscale.com`、`acme-v02.api.letsencrypt.org` |

`www.googleapis.com` 是 Google 共享 API 入口，放行面较大，因 agy 需要而保留。Orca 的 `share.onorca.dev`（分享上传）、`relay.onorca.dev`、`connector.onorca.dev` 在历史中未出现，未放行。

## 放行：终端里的 Homebrew 工具（via 规则，TCP 443）

这些工具只有 ad-hoc 签名，Rex Server 与 Orca 的宿主放行规则不覆盖它们，所以各自单独放行。平台服务沿用精确主机，内容站点沿用域名及子域范围。只列已观测工作流对应的工具与类别。

| 工具 | via | 类别 |
|---|---|---|
| gh | `path.opt/homebrew/Cellar/gh/*/bin/gh` | GitHub |
| Homebrew curl | `path.opt/homebrew/Cellar/curl/*/bin/curl` | GitHub、包管理、学术检索与参考 |
| Homebrew node | `path.opt/homebrew/Cellar/node/*/bin/node` | 包管理、GitHub、OpenAI、文档与工具 |

路径写法来自本机 Little Snitch 创建的规则：`*` 代表 `Cellar/<formula>/` 后的版本目录。远程订阅是否接受并正确匹配这类 via 路径仍需本机验证；更新后先确认规则组没有报错、新 Allow 已启用，再临时停用对应本地 Allow 测试。正常命中订阅后再删除本地规则。

## 放行：内容站点（域名及子域，TCP 443）

只写到具体主机的条目（如 `scholar.google.com`）只覆盖该主机及其自身子域。

| 类别 | 程序 | 域名 |
|---|---|---|
| 搜索 | 四个程序 | `kagi.com`、`duckduckgo.com`、`www.google.com`、`www.bing.com` |
| 文档与工具 | 四个程序 | `code.claude.com`、`developers.openai.com`、`releases.openai.com`、`www.superlogical.com`、`www.onorca.dev`、`core.telegram.org`、`pi.dev`（Pi 编程助手官网） |
| 学术检索与参考 | 四个程序 | `doi.org`、`crossref.org`、`openalex.org`、`semanticscholar.org`、`arxiv.org`、`scholar.google.com`、`books.google.com`、`api.unpaywall.org`、`core.ac.uk`、`philpapers.org`、`plato.stanford.edu`、`anthroencyclopedia.com`、`pubmed.ncbi.nlm.nih.gov`、`pmc.ncbi.nlm.nih.gov`、`www.researchgate.net`、`research.manchester.ac.uk`、`ccs.yale.edu`、`britsoc.co.uk`（英国社会学协会） |
| 学术出版商 | 四个程序 | `jstor.org`、`muse.jhu.edu`、`ssrn.com`、`sagepub.com`、`tandfonline.com`、`wiley.com`、`cambridge.org`、`oup.com`、`springer.com`、`sciencedirect.com`、`journals.uchicago.edu`、`dukeupress.edu`、`annualreviews.org` |
| 中文学术与图书 | 四个程序 | `cnki.net`、`ncpssd.cn`、`xueshu.baidu.com`、`book.douban.com`、`search.douban.com` |
| 图书与全文 | 四个程序 | `annas-archive.gd`、`annas-archive.pk`、`sci-hub.ru`、`wtgadtq.org`（影子图书馆全文下载）；`thestorygraph.com`（StoryGraph 书目） |

综合平台只放行具体主机：`google.com`、`googleapis.com`、`googleusercontent.com` 含 Forms、Apps Script、云存储桶与用户内容，`baidu.com` 含网盘，`douban.com` 含社区发帖，均可作为外传通道。允许匿名或第三方上传的平台（OSF、Zenodo、Wikipedia）、机构登录入口和出版社 PDF 的第三方 CDN 没有纳入。只开放 443：`http://` 旧链接（如 `http://dx.doi.org`）需改用 `https://`；Claude 的 WebFetch 会自动升级。GitHub、包仓库、Linear 与搜索服务的放行包含该主机上的读写能力，不能限定 URL、仓库、包或 HTTP 方法。

## 放行：网络与远程访问

- Rex 客户端、Rex Server、Orca 对整个 `ts.net` 及其子域放行，任意端口。`ts.net` 也包含其他 tailnet 公开的 Funnel 地址；如需更严格，可在本地改为只放行自己的 tailnet 子域。直接连 tailnet IP（`100.x`）不匹配此规则。
- Rex Server 放行 UDP 远端端口 1024-65535，用于内置 Tailscale 与对端设备直连、路由器端口映射和局域网发现。官方 DERP 已拒绝，直连是唯一通路。范围不含 53 和 443，DNS 与 QUIC 仍走兜底；DERP 与 STUN 节点由 `deny-derp` 的精确地址规则优先拒绝。
- Tailscale 协调另放行 `controlplane.tailscale.com` TCP 80，以及官方协调地址段 `192.200.0.0/24`、`2606:b740:49::/48` 的 TCP 80/443，覆盖只显示 IP 的记录。80 端口用于官方支持的加密协调传输。

## 自动拒绝

`deny-telemetry.lsrules`：遥测与一次性探测条目覆盖四个程序；agy 独立身份另有其自身遥测和非必要连接规则，范围如下。

| 类别 | 目标 |
|---|---|
| 遥测与特性开关 | `http-intake.logs.us5.datadoghq.com`、`browser-intake-us5-datadoghq.com`（Claude）、`analytics.brew.sh`（Homebrew）、`ab.chatgpt.com`（Codex Statsig）、`play.googleapis.com`、`antigravity-unleash.goog`（agy）、`log.tailscale.com`、`ipinfo.io` |
| 一次性 ICMP 测速 | `sgp-ping.vultr.com`、`lon-gb-ping.vultr.com`、`speedtest.singapore.linode.com`、`speedtest.london.linode.com` |
| agy 独立身份的遥测 | `play.googleapis.com`（Clearcut 使用统计）、`antigravity-unleash.goog`（特性开关） |
| agy 非必要连接（Rex Server、Orca、agy 独立身份） | `lh3.googleusercontent.com`（账号头像）、`www.gstatic.com`（曾观测到插件市场目录请求失败后使用缓存；规则拒绝整个主机）、`antigravity-cli-auto-updater-974169037036.us-central1.run.app`（自动更新检查） |
| 一次性 HTTP 探测（仅 TCP 80） | `dynamodb.ap-southeast-1.amazonaws.com`、`dynamodb.ap-northeast-1.amazonaws.com` |

Rex Server 另拒绝 Tailscale 官方日志地址段 `199.165.136.0/24`、`2606:b740:1::/48`。这些规则在白名单模式下与兜底拒绝重叠；保留它们是为了停用 `default-deny` 回到 Ask 时仍然静默拒绝。

`deny-derp.lsrules`，Rex 客户端、Rex Server、Claude、Codex：一条地址列表、一条主机名列表，合并官方 DERP map（2026-11-03 更新，保留已审阅旧节点）与 Tailscale 客户端内置的 bootstrap DNS 回退名单（2026-11-01 快照，24 个节点，含 `derp12c.tailscale.com`）。本次新增官方赫尔辛基节点 `derp28e.tailscale.com`：`46.62.141.44`、`2a01:4f9:c014:f18e::1`。与这些地址冲突的本地单 IP Allow 更具体，需要删除。回退名单包含现行 DERP map 之外的节点，客户端在常规 DNS 失败时用它们解析协调服务器；2026-11-01 曾观测到 Rex Server 连续尝试其中 14 个。拒绝后，系统 DNS 正常时不受影响。

`default-deny.lsrules` 对六个程序身份（含独立 agy）拒绝所有未被更具体规则放行的出站连接。独立 agy 的 baseline 有兜底 Ask；启用本组后，四个核心主机的 TCP 443 仍允许，其余未覆盖目标改为拒绝。更新前先启用 agy 的新增 Allow，避免先收到兜底拒绝而影响模型或登录。

早期 2026-10-30 至 10-31 日志的离线回放曾列出 Telegram HTTP 探测、ping/traceroute、`storage.googleapis.com`、Google 头像和路由器 ping 等新增拒绝；该旧回放未覆盖本次独立 agy 身份。`storage.googleapis.com` 是共享下载服务，未整站放行。agy 自动更新日志与更新时间支持它用于更新下载的推断，但尚未捕获具体下载 URL。

## 本地例外

家庭局域网、个人设备、个人服务与个人用途站点，以及自行编译工具的 via 规则，保留在本地，不发布。发布规则均为高优先级，Little Snitch 先比较优先级再比较具体度：本地例外若不是高优先级，会被 `default-deny` 覆盖。启用白名单模式前，应把需要保留的本地例外改为高优先级。列表形式的拒绝规则不如本地单条规则具体，与订阅冲突的本地单条放行需要删除。

## 已知缺口

- 遵循 macOS 系统代理的请求（Electron/Chromium 界面与内置浏览器、NSURLSession、Python urllib 等）会先连到本机 Surge 代理，再由 Surge 发出。Little Snitch 不过滤回环连接，看到的是 Surge 而不是原程序，这部分流量不受本白名单约束，需要在 Surge 侧另行限制。
- 终端宿主里的 ad-hoc 签名或未签名工具不受宿主规则覆盖。Homebrew 安装的工具（gh、uv，以及 brew 使用的 Homebrew curl、node、python 等）和自行编译的程序只有 ad-hoc 签名，没有开发者身份；Little Snitch 不对它们套用“宿主、任意 via”的规则，提示 “an existing rule for any via-process was not applied”。询问模式下逐个弹窗，白名单模式下被拒绝。需要为“宿主 via 工具”建单独的规则：gh、Homebrew curl、node 已由订阅提供，其他工具出现时先在本地放行，再补进订阅。
- 子进程若被 Little Snitch 归为独立身份而非父应用，只受它自身规则约束。
- 主机白名单无法实现只读：放行主机上的上传、推送和 API 写入同样可行。

## 迭代方法

1. 用新的 Little Snitch CSV 统计目标、进程归属、协议端口、连接与拒绝计数、收发字节和活跃小时桶。
2. 对照当前订阅，审查落到 Ask 或兜底拒绝的重复目标，以及弹窗新增的本地规则。
3. 新目标归入已有类别的列表；确实无法归类时才新建类别。填写用途、范围和证据，确认非必要的独立遥测目标加入拒绝列表。
4. 在原始历史和规则中出现的全部目标上比较修改前后的判定，再更新订阅。覆盖估算不代表真实弹窗减少比例，也不能证明本机已应用。
5. 下次导出时复核被拒绝计数和工作流是否受影响。应用可能继续重试，Deny 不等于停止发起连接。

## 依据

- [Little Snitch 规则与优先级](https://help.obdev.at/littlesnitch6/concepts-rules)
- [Little Snitch 规则文件格式](https://help.obdev.at/littlesnitch6/adv-lsrules-file-format)
- [Claude 官方网络要求](https://code.claude.com/docs/en/network-config)
- [Rex 网络用途](https://www.superlogical.com/rex/docs/reference/privacy)
- [Homebrew 分发与防火墙入口](https://docs.brew.sh/Homebrew-Package-Policies)
- [Tailscale 协调及日志网段](https://tailscale.com/kb/1082/firewall-ports)
- [Tailscale 官方 DERP map](https://controlplane.tailscale.com/derpmap/default)
- [Tailscale 内置 DNS 回退名单](https://github.com/tailscale/tailscale/blob/main/net/dnsfallback/dns-fallback-servers.json)
- [英国社会学协会](https://www.britsoc.co.uk/)
- [Pi 编程助手](https://pi.dev/)
- [Codex Statsig 连接](https://github.com/openai/codex/issues/12188)
