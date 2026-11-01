# 规则维护清单

这份清单描述当前发布规则。网络历史与连接计数留在本地；频率用于安排审查顺序，具体用途和已授权工作流决定是否放行。

## 核心与共享工作流

`baseline.lsrules` 保留模型服务、证书续期和 `ts.net` 规则；新增 Claude MCP 连接器及 OAuth 主机。下面的精确主机均允许 TCP 443，范围是 Rex Server、Claude Code、Codex 及 Little Snitch 归属于它们的子进程。共享工作流允许这三个程序使用同一组工具入口，不限定某个 curl、git 或 node 路径。

| 类别 | 精确主机 | 用途 |
|---|---|---|
| 开发 | `github.com` | Git 仓库克隆、拉取和推送 |
| 开发 | `api.github.com` | gh、GitHub API 和规则仓库维护 |
| 开发 | `raw.githubusercontent.com` | 仓库原始文件、配置和规则订阅 |
| 依赖安装 | `registry.npmjs.org` | npm/pnpm 包元数据与依赖下载 |
| 依赖安装 | `ghcr.io` | Homebrew 与 GitHub 包分发入口 |
| 依赖安装 | `pkg-containers.githubusercontent.com` | GitHub Packages 实际下载存储 |
| 依赖安装 | `formulae.brew.sh` | Homebrew 包目录与元数据 |
| 工具维护 | `releases.openai.com` | 历史中重复出现的发行信息访问 |
| 搜索 | `kagi.com` | 已使用的 Kagi 搜索工作流 |
| 工具文档 | `www.superlogical.com` | Rex 厂商文档 |
| 工具文档 | `www.onorca.dev` | Orca 文档工作流 |
| 工具文档 | `core.telegram.org` | Telegram 开发文档 |
| 学术 | `arxiv.org` | 论文阅读与检索；全机代理记录中的工作流候选，按已授权的学术白名单方向纳入 |
| 学术 | `scholar.google.com` | 学术搜索；全机代理记录中的工作流候选，按已授权的学术白名单方向纳入 |
| 学术 | `research.manchester.ac.uk` | 学术研究门户；代理记录不作为 agent 访问证明 |
| 学术 | `ccs.yale.edu` | 学术研究机构网站；代理记录不作为 agent 访问证明 |

学术项来自全机代理记录与已授权的学术阅读需求，不声称由 agent 直接访问。机构登录入口没有当作学术文档站自动放行。GitHub、包仓库与搜索服务的放行包含该主机上的读写能力，不能限定 URL、仓库、包或 HTTP 方法。

Tailscale 协调仅放行 `controlplane.tailscale.com` TCP 80/443，并补官方协调地址段 `192.200.0.0/24` 与 `2606:b740:49::/48` 的相同端口，覆盖只显示 IP 的记录。80 端口用于官方支持的加密协调传输。没有开放所有网站的 HTTP。

## 自动拒绝

`deny-telemetry.lsrules` 拒绝 Tailscale 日志、Claude Datadog 与 Homebrew 统计，补齐 Rex Server 父应用归属和 Tailscale 官方日志地址段。仍沿用原订阅 URL，另纳入不属于常规工作流的一次性诊断：`ipinfo.io` 公网 IP 查询；`sgp-ping.vultr.com`、`lon-gb-ping.vultr.com`、`speedtest.singapore.linode.com`、`speedtest.london.linode.com` 的 ICMP；`dynamodb.ap-southeast-1.amazonaws.com` 与 `dynamodb.ap-northeast-1.amazonaws.com` 的 TCP 80 HTTP 探测。需要专门排障时可停用对应规则；这不拒绝上述 DynamoDB 主机未来的 HTTPS API 使用，443 仍按未知目标询问。

`deny-derp.lsrules` 继续拒绝现行官方 DERP 快照内端点；另加入官方客户端内置的 `derp12c.tailscale.com` / `149.28.119.105` 及其 IPv6 bootstrap DNS 地址。这类连接也可能用于 DNS 回退，不应一律解释成中继载荷。新增精确 IP TCP 443 拒绝可压过同优先级、同 IP 的 Any Port 放行；其他本地具体规则仍需按 Little Snitch 的优先级检查。

## 保留询问

未确认用途的共享对象存储、尚未识别的公网 IP 以及未确认的个人设备地址继续 Ask。家庭局域网与个人设备例外保留在本地。没有将整个 Surge Fake IP 池、全部 UDP、公网 IP 段或 `.edu` 加入通行范围。

## 迭代方法

1. 用新的 Little Snitch CSV 统计目标、进程归属、协议端口、连接与拒绝计数、收发字节和活跃小时桶。
2. 对照当前三个订阅，优先审查仍会落到 Ask 的重复目标；同一工具链使用这一清单统一维护。
3. 为新增规则填写用途、范围和证据，保留未确认项。确认非必要的独立遥测目标加入 Deny。
4. 在原始历史上比较修改前后的覆盖情况，再更新订阅。覆盖估算不代表真实弹窗减少比例，也不能证明本机已应用。
5. 下次导出时复核实际新增目标、被拒绝计数和工作流是否受影响。应用可能继续重试，Deny 不等于停止发起连接。

## 依据

- [Little Snitch 规则与优先级](https://help.obdev.at/littlesnitch6/concepts-rules)
- [Claude 官方网络要求](https://code.claude.com/docs/en/network-config)
- [Homebrew 分发与防火墙入口](https://docs.brew.sh/Homebrew-Package-Policies)
- [Tailscale 协调及日志网段](https://tailscale.com/kb/1082/firewall-ports)
- [Tailscale 内置 DNS 回退名单](https://github.com/tailscale/tailscale/blob/main/net/dnsfallback/dns-fallback-servers.json)
