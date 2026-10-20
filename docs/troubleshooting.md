# 故障记录（Troubleshooting）

格式：`# | 日期 | 现象 → 查了哪 → 根因 → 修法 → 验证`
规则：死路也记；未验证必须标「未验证」；排查链比结论值钱。

| # | 日期 | 现象 → 查了哪 → 根因 → 修法 → 验证 |
|---|---|---|
| 1 | 2026-10-15 | `dnf` 加仓库报 `Curl error 6`（DNS 解析失败）→ 查网关：实际 `172.20.10.1`，**不是** VirtualBox 默认 `10.0.2.3`（环境已变）；`dig` 实际不存在（此记录作废）→ 根因：网关侧 DNS 不稳 → 修法：换可用 DNS（方案已给，**未敲**） | 未验证 |
| 2 | 2026-10-11 | 想停 sshd 报 `Access denied` → 查身份：服务归 root 管，chen 只是普通号 → 根因：权限不足 → 修法：`su - root` 再动 | 已验证（当天跑通 stop sshd） |
| 3 | 2026-10-11 | 敲 `systemctl is-enable` 报 `Unknown command verb`；`systemctl status nignx` 报 `could not be found` → 查：前者动词少 d（正确 `is-enabled`），后者服务名拼错（正确 `nginx`＝n-g-i-n-x）→ 根因：手滑打错，非权限/服务不存在 → 修法：复制粘贴正确写法 | 已验证 |

## 可迁移判据（lessons worth keeping）
- 解析失败先确认谁是"翻译官"（DNS），再确认翻译官问谁（网关）——别一上来怪网站。
- 命令拼错优先复制粘贴，不硬记；报 `Unit xxx not found` 先想是不是自己名字打错（见 #3）。
- 装完服务 `systemctl start xxx` 报 `Unit not found` → 先 `systemctl daemon-reload` 再试（本次 10-20 问题B 经验，已自愈）。

Key line (English): The troubleshooting chain (what you checked, dead ends included) is worth more than the conclusion — it shows how you think, not just what you fixed.
