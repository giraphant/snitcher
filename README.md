# Snitcher

Rex、Claude Code、Codex、Orca 的 Little Snitch 规则订阅，按白名单出口维护：核心服务与已审阅工作流允许，已确认的非必要目标拒绝，其余连接拒绝。停用 Default Deny 组即退回询问模式。

本仓库只包含规则和使用说明，不包含通信日志、个人主机名、代理配置或凭据。

## 订阅

在 Little Snitch 的规则窗口，点击 **Rule Groups → + → Remote Rule Group…**，按需要添加以下地址：

### 1. 核心白名单与询问

```text
https://raw.githubusercontent.com/giraphant/snitcher/main/rules/baseline.lsrules
```

模型与账号服务、Claude MCP、GitHub、npm/Homebrew、Kagi、已审阅工具文档、学术检索与出版站点按工作流放行，主要限精确主机 TCP 443。Rex 客户端、Rex Server 与 Orca 对整个 `ts.net` 及其子域放行；Rex Server 另放行内置 Tailscale 直连所需的高位 UDP；Tailscale 协调按官方要求补 TCP 80 与官方协调地址段。未匹配目标 Ask。

完整主机、进程范围、理由和待确认类别见 [规则维护清单](RULES.md)。个人局域网和设备地址留在本地。

### 2. 非必要遥测与一次性探测拦截

```text
https://raw.githubusercontent.com/giraphant/snitcher/main/rules/deny-telemetry.lsrules
```

按程序范围拒绝 Tailscale 日志、Claude Datadog、Homebrew 统计、Antigravity CLI 统计，并拒绝已审阅的一次性 IP 查询、ICMP 测速及特定 HTTP 探测。详细目标和协议见规则维护清单；需要专门排障时可停用对应规则或该组。

### 3. 官方 DERP 拦截（独立可选组）

```text
https://raw.githubusercontent.com/giraphant/snitcher/main/rules/deny-derp.lsrules
```

只针对上述程序阻止官方 DERP 节点的全部协议和端口，包括 STUN。名单以 2026-10-31 的官方快照为基础，包含 IPv4、IPv6 与域名，并补入 Tailscale 客户端内置的整份 bootstrap DNS 回退名单（2026-11-01 快照）；不包含自建 DERP 或 Peer Relay。可能影响新连接建立；停用这个组即可撤销其影响。

### 4. 白名单模式：默认拒绝

```text
https://raw.githubusercontent.com/giraphant/snitcher/main/rules/default-deny.lsrules
```

对上述五个程序身份拒绝所有未被更具体规则放行的出站连接。Little Snitch 在同优先级、同具体度下 Deny 优先于 Ask，所以它会盖过第 1 组的兜底询问，而第 1 组的各项放行更具体，仍然生效。停用这个组即回到询问模式。

## 启用顺序

1. 先让第 1 组更新到最新版本，在规则组里确认新增 Allow 全部已启用（勾选 **Disable new allow rules** 时，新增 Allow 默认是禁用的）。
2. 把需要保留的本地例外（家庭局域网、个人设备）改为高优先级，否则会被第 4 组覆盖。
3. 再添加第 4 组。Deny 规则不受 “Disable new allow rules” 限制，添加后立即生效；若此时 Allow 仍处于禁用状态，相应程序会直接断网。
4. 出现异常时先停用第 4 组，确认原因后再补规则。

## 自动更新与回退

- 将更新间隔设为 **Hourly**。推送后的文件需要客户端拉取，并非实时生效。
- 检查 **Disable new allow rules**：保留勾选时，新增 Allow 需要手动审核启用；取消勾选才会自动采用维护者发布的新增 Allow。
- `rules/` 内规则为启用状态，只有添加并启用对应订阅组才会作用于设备。
- 出现问题时可单独停用相应规则组。不要通过关闭整个网络过滤器来回退。
- 维护时保留 Git 历史，更新规则时说明用途与原因；可以通过回退提交发布修正版。

## 范围和边界

规则通过签名身份匹配 Rex 客户端、Rex Server、Claude Code、Codex、Orca，没有 `Any Process` 全机规则。程序签名变化后需要重新核对。Rex Server 与 Orca 是终端宿主，其中运行的命令归属到它们，终端宿主上的放行对其内所有命令生效。

发布规则采用高优先级。Little Snitch 先比较优先级再比较具体度，普通优先级的本地规则无论多具体都会被这些规则覆盖。普通 Third Party Apps 全放行不应盖过这些高优先级规则。

子进程、Surge、浏览器或后台代理可能被归为不同进程；父应用规则不保证覆盖所有委托请求。遵循 macOS 系统代理的请求（Electron 界面与内置浏览器、NSURLSession、Python urllib 等）由本机 Surge 代发，Little Snitch 不过滤回环连接，这部分流量不受本白名单约束。主机白名单也无法限制 URL、HTTP 方法、仓库或请求内容。共享平台的放行不等于只读权限。

白名单模式下，被拒绝的连接不会弹窗，表现为程序报网络错误或超时。若某项工作流失效，先在 Little Snitch 的连接记录里查看被拒绝的目标与归属进程，确认用途后再补规则；不要为排障整体放行。

Surge 的虚拟接口及 Fake IP 需要分开审视：官方文档说明 `198.18.x.x` 可代表经 Surge 转发的不同真实域名。因此本订阅不将整个 Fake IP 地址池加入白名单；目的地址应结合本机接口与 Surge 请求记录判断。参见 [Surge 官方原理说明](https://manual.nssurge.com/book/understanding-surge/cn/)。

这些规则持续打磨，不保证覆盖所有工作流。新目标先确认具体用途，必要时临时停用第 4 组，验证之后再纳入长期规则。

## 参考

- [Little Snitch 订阅与更新](https://help.obdev.at/littlesnitch6/lsc-rulegroups)
- [Little Snitch 规则格式](https://help.obdev.at/littlesnitch6/adv-lsrules-file-format)
- [Little Snitch 规则优先级](https://help.obdev.at/littlesnitch6/concepts-rules)
- [Claude 官方网络要求](https://code.claude.com/docs/en/network-config)
- [Rex 网络用途](https://www.superlogical.com/rex/docs/reference/privacy)
- [Tailscale DERP](https://tailscale.com/kb/1232/derp-servers)
