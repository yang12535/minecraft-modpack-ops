---
name: minecraft-modpack-ops
description: 通过 SSH 部署、更新、迁移、精简、暴露、自动化、监控和排查模组 Minecraft 专用服务器。适用于 CurseForge 或 Modrinth 导出包、Forge/Fabric/NeoForge 整合包、客户端模组修剪、代理辅助下载、含在线/离线 UUID 迁移的世界替换、systemd 和全局生命周期命令、RCON、服务端 KubeJS 自动化、FRP 隧道、NAT 端口映射、多出口对比、历史稳定性分析、玩家握手失败和性能验证。
---

# Minecraft Modpack Ops

## 用途

将此技能用作模组 Minecraft 服务器运维的路由入口。它是产品中立的，任何具有适当本地 Shell、文件系统、网络和 SSH 访问权限的自动化代理均可遵循。仅读取当前任务所需的参考文件；不要默认加载所有指南。

## 任务路由

- **部署、更新、修复、精简、调优或运维模组包：** 阅读 [references/modpack-deployment.md](references/modpack-deployment.md)。
- **替换世界、更改在线/离线模式、迁移 UUID、保留玩家/模组数据或重建 OP 记录：** 阅读 [references/world-migration.md](references/world-migration.md)。
- **在不改动客户端的前提下添加服务端 KubeJS 登录任务、延迟私聊通知、会话感知调度或自定义命令：** 阅读 [references/kubejs-server-automation.md](references/kubejs-server-automation.md)。
- **通过 VPS、提供商 NAT、FRPS/FRPC 暴露 Minecraft，或将隧道耦合到启停命令：** 阅读 [references/frp-tunneling.md](references/frp-tunneling.md)。
- **调查历史稳定性、比较多出口、统计成功和失败连接、或区分服务健康与进程状态：** 阅读 [references/stability-observability.md](references/stability-observability.md)。
- **混合任务：** 阅读每个相关参考文件，然后合并其验证要求。例如，世界替换后紧接着公网暴露需要同时参考世界和 FRP 指南。

## 共享原则

- 在更改任何内容之前，先检查工件、主机、运行中的服务、端口和现有约定。
- 默认保留内容和用户数据。仅在有证据支持的情况下移除或重写。
- 使 Minecraft 和隧道进程独立于 SSH 会话运行。
- 将下载代理的作用域限制于下载命令。
- 将 RCON 密码、FRP 令牌、SSH 密钥和其他凭证保存在服务器上，不要放在聊天或工作区文件中。
- 将玩家名称、UUID、IP 地址、公共端点、日志、世界存档、`ops.json` 和 `usercache.json` 视为敏感运维数据。不要提交或在授权环境之外持久化这些数据。
- 避免将机密放在命令参数、Shell 历史、调试追踪、进程列表或日志中。
- 将启动、网络、客户端握手、生命周期和运行时验收测试分开。
- 优先使用精确版本并验证可用的哈希值。
- 在修改之前备份现有脚本，并保留无关服务或用户更改。
- 对清单、JSON、TOML、NBT 和区域数据使用结构化解析器。
- 永远不要仅凭进程存在或 TCP 连接就声称成功；验证实际的 Minecraft 行为。
- 在解释故障之前，确认每个被比较的主机或中继仍然存在且在观察窗口内本应处于活跃状态。

## 完成报告

仅报告高价值的运维事实：

- 安装或更改了什么
- 本地和公共端点状态；仅在授权请求者需要时披露确切地址，绝不在公共工件中披露
- 认证和权限模式
- 启动/停止/RCON/隧道行为
- 启动、状态握手、登录、优雅停止和重启结果
- 未解决的警告或未测试的验收阶段

绝不包含凭证、令牌哈希、玩家 IP、原始 UUID 映射或不必要的日志摘录。
