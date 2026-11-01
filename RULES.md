# 规则维护清单

这份清单描述当前发布规则。网络历史与连接计数留在本地；频率用于安排审查顺序，具体用途和已授权工作流决定是否放行。

## 模式

四个订阅组配合使用：`baseline` 放行已审阅目标并对其余目标 Ask；`deny-telemetry` 与 `deny-derp` 拒绝已确认的非必要目标；`default-deny` 把兜底从 Ask 改成 Deny，构成白名单出口。Little Snitch 在同优先级、同具体度下按 Deny > Allow > Ask 判定，所以启用 `default-deny` 后其“任意目标拒绝”压过 baseline 的“任意目标询问”，而任何精确主机、域名或端口范围放行仍更具体、仍然生效。停用 `default-deny` 即回到 Ask。

覆盖的程序身份：

| 程序 | 签名身份 |
|---|---|
| Rex 客户端 | `identifier.AG92C3PT77/com.superlogical.rex` |
| Rex Server | `identifier.AG92C3PT77/com.superlogical.rex.server` |
| Claude Code | `identifier.Q6L2SF6YDW/com.anthropic.claude-code` |
| Codex | `identifier.2DC432GLL2/codex` |
| Orca | `identifier.6CX3WHS9HZ/com.stablyai.orca` |

Rex Server 与 Orca 是终端宿主，在其中运行的 claude、codex、gh、curl、node、agy 等子进程由 Little Snitch 归属到它们。因此终端宿主上的放行对其内所有命令生效。

## 共享工作流

下列精确主机均允许 TCP 443，范围是 Rex Server、Claude Code、Codex、Orca 及归属于它们的子进程。

| 类别 | 精确主机 | 用途 |
|---|---|---|
| 开发 | `github.com` | Git 仓库克隆、拉取和推送 |
| 开发 | `api.github.com` | gh、GitHub API 和规则仓库维护 |
| 开发 | `raw.githubusercontent.com` | 仓库原始文件、配置和规则订阅 |
| 开发 | `codeload.github.com` | GitHub 仓库归档下载（只读 CDN） |
| 开发 | `objects.githubusercontent.com` | GitHub Release 资产下载（旧地址，只读 CDN） |
| 开发 | `release-assets.githubusercontent.com` | GitHub Release 资产下载；Orca 自动更新与 Codex cask 使用 |
| 依赖安装 | `registry.npmjs.org` | npm/pnpm 包元数据与依赖下载 |
| 依赖安装 | `ghcr.io` | Homebrew 与 GitHub 包分发入口 |
| 依赖安装 | `pkg-containers.githubusercontent.com` | GitHub Packages 实际下载存储 |
| 依赖安装 | `formulae.brew.sh` | Homebrew 包目录与元数据 |
| 工具维护 | `releases.openai.com` | 历史中重复出现的发行信息访问 |
| 搜索 | `kagi.com` | 已使用的 Kagi 搜索工作流 |
| 工具文档 | `www.superlogical.com` | Rex 厂商文档 |
| 工具文档 | `www.onorca.dev` | Orca 文档工作流 |
| 工具文档 | `core.telegram.org` | Telegram 开发文档 |
| 学术 | `arxiv.org` | 论文阅读与检索；全机代理记录中的工作流候选 |
| 学术 | `scholar.google.com` | 学术搜索；全机代理记录中的工作流候选 |
| 学术 | `research.manchester.ac.uk` | 学术研究门户；代理记录不作为 agent 访问证明 |
| 学术 | `ccs.yale.edu` | 学术研究机构网站；代理记录不作为 agent 访问证明 |
| 学术检索 | `doi.org` | DOI 解析 |
| 学术检索 | `api.crossref.org` | Crossref 文献元数据 |
| 学术检索 | `api.openalex.org` | OpenAlex 文献与引用检索 |
| 学术检索 | `api.semanticscholar.org` | Semantic Scholar API |
| 学术检索 | `www.semanticscholar.org` | Semantic Scholar 页面 |
| 学术检索 | `export.arxiv.org` | arXiv API |
| 学术检索 | `api.unpaywall.org` | 开放获取版本查询 |
| 学术检索 | `core.ac.uk` | CORE 开放获取全文聚合 |
| 学术检索 | `philpapers.org` | PhilPapers 理论与哲学文献索引 |
| 学术检索 | `plato.stanford.edu` | 斯坦福哲学百科 |
| 学术检索 | `books.google.com` | 图书检索与预览 |
| 学术检索 | `pubmed.ncbi.nlm.nih.gov` | PubMed 摘要检索 |
| 学术检索 | `pmc.ncbi.nlm.nih.gov` | PubMed Central 开放全文 |
| 学术出版 | `www.jstor.org` | JSTOR |
| 学术出版 | `muse.jhu.edu` | Project MUSE |
| 学术出版 | `papers.ssrn.com` | SSRN 预印本 |
| 学术出版 | `journals.sagepub.com` | SAGE 期刊 |
| 学术出版 | `www.tandfonline.com` | Taylor & Francis 期刊 |
| 学术出版 | `onlinelibrary.wiley.com` | Wiley 期刊 |
| 学术出版 | `www.cambridge.org` | Cambridge Core |
| 学术出版 | `academic.oup.com` | Oxford Academic |
| 学术出版 | `link.springer.com` | Springer |
| 学术出版 | `www.sciencedirect.com` | Elsevier ScienceDirect |
| 学术出版 | `www.journals.uchicago.edu` | University of Chicago Press 期刊 |
| 学术出版 | `read.dukeupress.edu` | Duke University Press |
| 学术出版 | `www.annualreviews.org` | Annual Reviews |

“学术”四项来自全机代理记录；“学术检索”与“学术出版”是按学术工作需要预置的只读来源，不声称已由 agent 访问。允许匿名或第三方上传的平台（OSF、Zenodo、Wikipedia、ResearchGate）、机构登录入口和出版社 PDF 的第三方 CDN 没有纳入；DOI 跳转到名单外出版社时会被拒绝，需要时逐个补充。GitHub、包仓库与搜索服务的放行包含该主机上的读写能力，不能限定 URL、仓库、包或 HTTP 方法。

## 模型、账号与程序服务

均为精确主机 TCP 443。

| 程序范围 | 精确主机 | 用途 |
|---|---|---|
| Claude、Rex Server、Orca | `api.anthropic.com` | Claude API、WebFetch 域名安全检查、特性开关 |
| Claude、Rex Server、Orca | `mcp-proxy.anthropic.com` | Claude MCP 连接器 |
| Claude、Rex Server、Orca | `platform.claude.com` | 登录、OAuth token 交换与刷新 |
| Claude、Rex Server、Orca | `claude.ai` | claude.ai 账号认证 |
| Claude、Rex Server、Orca | `downloads.claude.ai` | Claude Code 下载与版本检查；Homebrew cask 从此下载 |
| Claude、Rex Server、Orca | `code.claude.com` | Claude Code 官方文档查询 |
| Codex、Rex Server、Orca | `chatgpt.com` | Codex ChatGPT 登录模式的模型服务 |
| Codex、Rex Server、Orca | `auth.openai.com` | Codex 登录与 token 刷新；日志未出现，按登录流程预置 |
| Rex Server、Orca | `daily-cloudcode-pa.googleapis.com` | Antigravity CLI（agy）模型服务 |
| Rex Server、Orca | `aicode.googleapis.com` | agy 服务接口 |
| Rex Server、Orca | `oauth2.googleapis.com` | agy Google 登录 token 刷新 |
| Rex Server、Orca | `www.googleapis.com` | agy 用户信息；Google 共享 API 入口，放行面较大 |
| Orca | `onorca.dev` | 更新说明与插件停用名单 |
| Orca | `login.onorca.dev` | Orca 账号登录 |
| Orca | `push.onorca.dev` | Orca 推送通知 |
| Rex Server | `acme-v02.api.letsencrypt.org` | 内置 HTTPS 证书续期 |

Orca 的 `share.onorca.dev`（分享上传）、`relay.onorca.dev`、`connector.onorca.dev` 在历史中未出现，未放行。

## 网络与远程访问

- Rex 客户端、Rex Server、Orca 对整个 `ts.net` 及其子域放行，任意端口。`ts.net` 也包含其他 tailnet 公开的 Funnel 地址；如需更严格，可在本地改为只放行自己的 tailnet 子域。
- Rex Server 放行 UDP 远端端口 1024-65535，用于内置 Tailscale 与对端设备直连、路由器端口映射和局域网发现。官方 DERP 已拒绝，直连是唯一通路。范围不含 53 和 443，所以 DNS 与 QUIC 仍走兜底拒绝；DERP 与 STUN 节点由 `deny-derp` 的精确地址规则优先拒绝。
- Tailscale 协调仅放行 `controlplane.tailscale.com` TCP 80/443，并补官方协调地址段 `192.200.0.0/24` 与 `2606:b740:49::/48` 的相同端口，覆盖只显示 IP 的记录。80 端口用于官方支持的加密协调传输。

## 自动拒绝

`deny-telemetry.lsrules` 拒绝 Tailscale 日志、Claude Datadog（遥测与错误报告两个主机）、Homebrew 统计，以及 agy 的 `play.googleapis.com` 使用统计与 `antigravity-unleash.goog` 特性开关；补齐 Rex Server 父应用归属和 Tailscale 官方日志地址段。另纳入不属于常规工作流的一次性诊断：`ipinfo.io` 公网 IP 查询；`sgp-ping.vultr.com`、`lon-gb-ping.vultr.com`、`speedtest.singapore.linode.com`、`speedtest.london.linode.com` 的 ICMP；`dynamodb.ap-southeast-1.amazonaws.com` 与 `dynamodb.ap-northeast-1.amazonaws.com` 的 TCP 80 HTTP 探测。Orca 归属的同类连接一并拒绝。这些规则在白名单模式下与兜底拒绝重叠；保留它们是为了停用 `default-deny` 回到 Ask 时仍然静默拒绝。

`deny-derp.lsrules` 继续拒绝现行官方 DERP 快照内端点；另加入官方客户端内置的 `derp12c.tailscale.com` / `149.28.119.105` 及其 IPv6 bootstrap DNS 地址。这类连接也可能用于 DNS 回退，不应一律解释成中继载荷。

`default-deny.lsrules` 对上述五个程序身份拒绝所有未被更具体规则放行的出站连接。按 2026-10-30 至 10-31 的历史回放，启用后新增拒绝的只有：Telegram 地址的 HTTP 探测与 ping/traceroute、`storage.googleapis.com`（Claude `/plugin` 安装统计）、`lh3.googleusercontent.com`（Google 头像）及对路由器的 ping。

## 本地例外

家庭局域网与个人设备例外保留在本地，不发布。发布规则均为高优先级，Little Snitch 先比较优先级再比较具体度：本地例外若不是高优先级，会被 `default-deny` 覆盖。启用白名单模式前，应把需要保留的本地例外改为高优先级。

## 已知缺口

- 遵循 macOS 系统代理的请求（Electron/Chromium 界面与内置浏览器、NSURLSession、Python urllib 等）会先连到本机 Surge 代理，再由 Surge 发出。Little Snitch 不过滤回环连接，看到的是 Surge 而不是原程序，这部分流量不受本白名单约束，需要在 Surge 侧另行限制。
- 子进程若被 Little Snitch 归为独立身份而非父应用，只受它自身规则约束。
- 主机白名单无法实现只读：放行主机上的上传、推送和 API 写入同样可行。

## 迭代方法

1. 用新的 Little Snitch CSV 统计目标、进程归属、协议端口、连接与拒绝计数、收发字节和活跃小时桶。
2. 对照当前订阅，审查白名单模式下被兜底拒绝的重复目标；同一工具链使用这一清单统一维护。
3. 为新增规则填写用途、范围和证据，保留未确认项。确认非必要的独立遥测目标加入 Deny。
4. 在原始历史上比较修改前后的覆盖情况，再更新订阅。覆盖估算不代表真实弹窗减少比例，也不能证明本机已应用。
5. 下次导出时复核被拒绝计数和工作流是否受影响。应用可能继续重试，Deny 不等于停止发起连接。

## 依据

- [Little Snitch 规则与优先级](https://help.obdev.at/littlesnitch6/concepts-rules)
- [Claude 官方网络要求](https://code.claude.com/docs/en/network-config)
- [Rex 网络用途](https://www.superlogical.com/rex/docs/reference/privacy)
- [Homebrew 分发与防火墙入口](https://docs.brew.sh/Homebrew-Package-Policies)
- [Tailscale 协调及日志网段](https://tailscale.com/kb/1082/firewall-ports)
- [Tailscale 内置 DNS 回退名单](https://github.com/tailscale/tailscale/blob/main/net/dnsfallback/dns-fallback-servers.json)
