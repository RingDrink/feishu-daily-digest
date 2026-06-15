# feishu-daily-digest

飞书每日消息摘要 + 智能会议纪要 Skill。

## 这是什么

每天自动扫描你的飞书消息和会议智能纪要，生成一份结构化 briefing 发到你的飞书 DM。

- **消息扫描**：近 26h 所有群聊 + 私聊，可自定义黑名单过滤
- **会议纪要**：自动读取飞书 VC 智能会议纪要
- **结构化输出**：🔔 需要跟进 → 📌 拍板/口径 → 📝 会议纪要 → 💬 其他要点
- **多平台支持**：通过飞书 Bot 发送，支持 macOS / Windows / Linux 定时执行

## 安装

通过 Atlas Skill Hub 安装（莉莉丝内网）：

```bash
atlas-skillhub add feishu-daily-digest --non-interactive
```

或手动克隆本仓库，将 `skill/SKILL.md` 放到你的 agent 技能目录。

## 前置条件

1. **飞书自建应用（Bot）**：用于发送摘要消息（强依赖）
2. **lark-cli**：飞书 CLI 工具，用于拉取消息和会议数据
3. **飞书 OAuth 授权**：消息读取 + 会议纪要读取（只读权限）

详见 `skill/SKILL.md` 中的前置条件章节。

## 兼容性

- OpenClaw / ArkClaw / 其他 Claw 系
- Claude Code
- Codex
- Cursor
- 其他支持读取本地文件和执行 shell 命令的 agent

## 维护

- Owner: Jyun (周林寯)
- License: MIT
