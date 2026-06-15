# Minecraft Modpack Ops（中文版）

一个产品中立的运维技能指南，用于部署和维护模组 Minecraft 专用服务器。

本仓库以文档优先。不依赖 Codex 或特定模型提供商。任何自动化代理均可使用，前提是该代理能够：

- 读取 Markdown 文件
- 检查和编辑本地文件
- 运行本地 Shell 命令
- 通过 SSH 连接到已授权的主机
- 执行网络和应用层验证

代理必须对每个目标系统拥有明确的授权。

> **语言选项：** 本仓库默认提供中文版。英文版请参见 [en/](en/) 文件夹。

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
|-- README.md
|-- SKILL.md
|-- zh/
|   |-- README.md
|   |-- SKILL.md
|   |-- references/
|   |   |-- modpack-deployment.md
|   |   |-- world-migration.md
|   |   |-- frp-tunneling.md
|   |   `-- stability-observability.md
|   `-- agents/
|       `-- openai.yaml
|-- en/
|   |-- README.md
|   |-- SKILL.md
|   |-- references/
|   |   |-- modpack-deployment.md
|   |   |-- world-migration.md
|   |   |-- frp-tunneling.md
|   |   `-- stability-observability.md
|   `-- agents/
|       `-- openai.yaml
`-- agents/
    `-- openai.yaml
```

`agents/openai.yaml` 是适用于兼容 OpenAI/Codex 环境的可选 UI 元数据。运维指南不依赖此文件。

## 与任何 Shell 代理配合使用

将本仓库的访问权限授予代理，并指示其从 `SKILL.md` 开始：

```text
阅读 /absolute/path/to/minecraft-modpack-ops/SKILL.md 并按照其路由
指示处理此 Minecraft 服务器任务。使用可用的 Shell 和 SSH 工具，
保留无关服务，并完成相关验证阶段。
```

具有技能目录约定的代理或框架可以将本仓库复制或符号链接到该目录中。否则，从代理的系统指令、项目指令或任务提示中引用 `SKILL.md`。

工作流假定代理能够根据操作系统和现有服务约定调整命令。它有意不提供通用部署脚本。

## 示例

```text
将此 CurseForge 导出文件部署为 Forge 专用服务器。仅移除已证明为
客户端专用的模组，添加优雅的全局启动/停止命令，并验证真实客户端登录。
```

```text
用此存档替换已停止服务器的世界。目标将使用离线模式，
因此请在启动前迁移所有玩家和模组的 UUID 引用。
```

```text
通过 FRP 中继暴露此私有 Minecraft 服务器。加固 FRPS，将 FRPC 
耦合到现有的 Minecraft 生命周期命令，并验证公共 Minecraft 
状态协议。
```

```text
比较这两个公共出口近期的稳定性。将被动日志与主动探测分开，
排除任何已退役的中继，并报告 Minecraft 超时、延迟、内存、
GC、交换以及出口归属的可信度。
```

## 安全与隐私

本仓库不包含生产端点、玩家身份、UUID、凭证或特定服务器路径。

使用本技能时：

- 将 SSH 密钥、RCON 密码、FRP 令牌和代理凭证保存在已授权的主机上
- 切勿将机密放在命令参数、Shell 追踪、进程列表、日志、聊天或仓库文件中
- 将玩家名称、UUID、IP 地址、坐标、背包、世界、日志、`ops.json` 和 `usercache.json` 视为敏感数据
- 保持 UUID 映射表和提取的世界为临时性
- 在存储或共享之前，对包含身份信息的日志摘录进行脱敏处理
- 仅向授权请求者披露确切的基础设施端点
- 在验证凭证相等性时，不要发布凭证或其哈希值
- 验证完成后清理临时存档、提取树和下载的二进制文件

包含的 `.gitignore` 是纵深防御，不能替代每次提交前审查暂存更改。

## 运维预期

- 在更改之前先检查。
- 保留用户内容和无关服务。
- 优先使用结构化解析器，而非原始文本或二进制替换。
- 将长时间运行的服务与代理的 Shell 会话分离。
- 在可用时使用 RCON 进行优雅保存和停止。
- 验证 Minecraft 本身，而不仅仅是进程或 TCP 监听器。
- 当真实客户端登录或其他验收阶段尚未测试时，明确说明。

## 审查状态

初始仓库已审查以下内容：

- 嵌入的 IP 地址、用户名、UUID、电子邮件和本地机器路径
- 凭证、令牌、密码和私钥引用
- 特定实例的服务名称和命令
- 核心工作流中仅限 Codex 的假设
- 意外包含的世界、日志、存档、二进制文件或玩家数据

核心文档是平台中立的；仅 `agents/openai.yaml` 是提供商特定且可选的。
