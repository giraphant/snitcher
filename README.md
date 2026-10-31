# Snitcher

Rex、Claude Code、Codex 的 Little Snitch 规则订阅。逐项维护必要目标：核心服务允许，已确认的非必要目标拒绝，其余连接询问。

本仓库只包含规则和使用说明，不包含通信日志、个人主机名、代理配置或凭据。

## 订阅

在 Little Snitch 的规则窗口，点击 **Rule Groups → + → Remote Rule Group…**，按需要添加以下地址：

### 1. 核心白名单与询问

```text
https://raw.githubusercontent.com/giraphant/snitcher/main/rules/baseline.lsrules
```

允许 Claude 的 `api.anthropic.com`、Codex 的 `chatgpt.com`、Rex 的 Tailscale 协调及证书续期服务，均限 TCP 443。另包含 Rex Server 对 `api.anthropic.com` 和 `chatgpt.com` 的精确主机规则，以覆盖被 Little Snitch 归属为 Rex Server via claude/codex 的连接。它们按父应用匹配，也允许其他 Rex 子进程访问这两个主机，并不限定具体 helper。其他出站目标使用 Ask。个人远程主机、开发工具和学术网站按实际需要逐项确认，不做整类通配放行。

### 2. 非必要遥测拦截

```text
https://raw.githubusercontent.com/giraphant/snitcher/main/rules/deny-telemetry.lsrules
```

按程序范围拒绝 Tailscale 日志接收端、Claude Datadog 日志接收端和 Rex 归属的 Homebrew 统计。关闭这个组即可撤销该组影响。

### 3. 官方 DERP 拦截（独立可选组）

```text
https://raw.githubusercontent.com/giraphant/snitcher/main/rules/deny-derp.lsrules
```

只针对上述程序阻止官方 DERP 节点的全部协议和端口，包括 STUN。名单为 2026-10-31 的官方快照，包含 IPv4、IPv6 与域名，不包含自建 DERP 或 Peer Relay。可能影响新连接建立；停用这个组即可撤销其影响。

## 自动更新与回退

- 将更新间隔设为 **Hourly**。推送后的文件需要客户端拉取，并非实时生效。
- 检查 **Disable new allow rules**：保留勾选时，新增 Allow 需要手动审核启用；取消勾选才会自动采用维护者发布的新增 Allow。首次添加也应确认核心 Allow 没有被禁用。
- `rules/` 内规则为启用状态，只有添加并启用对应订阅组才会作用于设备。
- 出现问题时可单独停用相应规则组。不要通过关闭整个网络过滤器来回退。
- 维护时保留 Git 历史，更新规则时说明用途与原因；可以通过回退提交发布修正版。

## 范围和边界

规则通过签名身份匹配 Rex 客户端、Rex Server、Claude Code、Codex，没有 `Any Process` 全机规则。程序签名变化后需要重新核对。

发布规则采用高优先级。精确主机规则与兜底 Ask 同优先级，精确主机更具体。现有的高优先级具体规则仍可能覆盖 Ask；本地临时例外也需要核对优先级。普通 Third Party Apps 全放行不应盖过这些高优先级规则。

子进程、Surge、浏览器或后台代理可能被归为不同进程；父应用规则不保证覆盖所有委托请求。主机白名单也无法限制 URL、HTTP 方法、仓库或请求内容。共享平台的放行不等于只读权限。

连接询问窗口表示该次连接正在等待决定，不表示已经连通，也不表示它有必要。若预期拒绝的目标仍触发 Ask，应检查订阅是否更新并启用、地址是否在名单内、进程归属及本地规则优先级；不要仅凭弹窗新增 Allow。

这些是持续打磨的初始规则，不保证所有工作流免打扰。新目标先确认具体用途，必要时临时允许，验证之后再纳入长期规则。

## 参考

- [Little Snitch 订阅与更新](https://help.obdev.at/littlesnitch6/lsc-rulegroups)
- [Little Snitch 规则格式](https://help.obdev.at/littlesnitch6/adv-lsrules-file-format)
- [Little Snitch 规则优先级](https://help.obdev.at/littlesnitch6/concepts-rules)
- [Rex 网络用途](https://www.superlogical.com/rex/docs/reference/privacy)
- [Tailscale DERP](https://tailscale.com/kb/1232/derp-servers)
