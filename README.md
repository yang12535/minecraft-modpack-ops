# Minecraft Modpack Ops（中文版）

一个产品中立的运维技能指南，用于部署和维护模组 Minecraft 专用服务器。

本仓库以文档优先。不依赖 Codex 或特定模型提供商。任何自动化代理均可使用，前提是该代理能够：

- 读取 Markdown 文件
- 检查和编辑本地文件
- 运行本地 Shell 命令
- 通过 SSH 连接到已授权的主机
- 执行网络和应用层验证

代理必须对每个目标系统拥有明确的授权。

> **语言 / Language：** 默认提供中文版。For English, see [en/](en/) folder.

## 覆盖范围

本技能涵盖四个相关工作流：

- 模组包部署、保守的客户端专用模组修剪、管理和性能验证
- 世界替换（含在线/离线 UUID 迁移和模组特定玩家数据保留）
- FRP 和提供商 NAT 暴露，与 Minecraft 安全生命周期耦合
- 跨 FRP、网络和 Minecraft 层的历史稳定性分析和公平的多出口对比

`SKILL.md` 有意保持简短，仅将代理路由到 `zh/references/` 下的相关文件。

## 仓库布局

```text
.
|-- README.md          (中文，默认)
|-- SKILL.md           (中文，默认)
|-- zh/                (中文全文)
|-- en/                (English full text)
|-- references/        (保留兼容)
|-- agents/            (保留兼容)
```

## 与任何 Shell 代理配合使用

将本仓库的访问权限授予代理，并指示其从 `SKILL.md` 开始：

```text
阅读 /absolute/path/to/minecraft-modpack-ops/SKILL.md 并按照其路由
指示处理此 Minecraft 服务器任务。使用可用的 Shell 和 SSH 工具，
保留无关服务，并完成相关验证阶段。
```

## 安全与隐私

- 将 SSH 密钥、RCON 密码、FRP 令牌和代理凭证保存在已授权的主机上
- 切勿将机密放在命令参数、Shell 追踪、进程列表、日志、聊天或仓库文件中
- 将玩家名称、UUID、IP 地址、坐标、背包、世界、日志、`ops.json` 和 `usercache.json` 视为敏感数据

## 审查状态

核心文档是平台中立的；仅 `agents/openai.yaml` 是提供商特定且可选的。
