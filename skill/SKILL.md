---
name: feishu-daily-digest
description: 每日飞书全量消息摘要 + 智能会议纪要。扫描近一天所有群聊/私聊（可自定义黑名单），拉取飞书 VC 智能纪要，产出结构化日摘要（🔔跟进 → 📌拍板 → 📝会议纪要 → 💬要点），通过飞书 Bot 发到你的 DM。支持定时自动执行。
version: 1.0.2
created: 2026-06-15
author: Jyun (周林寯)
---

# 飞书每日摘要

每天自动扫描你的飞书消息和会议智能纪要，生成一份结构化 briefing 发到你的飞书 DM。

## 它能做什么

1. **扫近 26h 飞书消息**：所有群聊 + 私聊，过滤掉你不想看的群和系统 bot
2. **拉飞书 VC 智能会议纪要**：同窗口内有会议的话，自动读取智能纪要
3. **生成结构化摘要**：按「需要你跟进 → 拍板/口径 → 会议纪要 → 其他要点」分组，不是流水账
4. **通过飞书 Bot 发到你 DM**：只发给你自己

---

## 前置条件

### 1. 飞书自建应用（Bot）—— 强依赖

本 skill **必须有一个飞书自建应用（Bot）** 来发送消息。没有 Bot 无法投递摘要。

**创建步骤**：
1. 前往 [飞书开放平台](https://open.feishu.cn/) → 创建企业自建应用
2. 应用能力 → 开启「机器人」
3. 权限管理 → 申请以下权限：
   - `im:message:send_as_bot`（以机器人身份发消息）
   - `im:chat:readonly`（读取会话列表）
   - `im:message:readonly`（读取消息）
   - `im:message.reactions:read`（读取消息 reaction）
   - `vc:meeting.search:read`（搜索会议，如需会议纪要）
   - `vc:meeting.meetingevent:read`（读取会议事件，如需会议纪要）
   - `vc:note:read`（读取智能纪要，如需会议纪要）
   - `vc:record:readonly`（读取会议录制，如需会议纪要）
   - `docs:doc:readonly`（读取智能纪要文档正文，如需会议纪要）
4. 版本管理 → 创建版本 → 发布（需管理员审核）
5. 记录 **App ID** 和 **App Secret**

> **如果你已经有飞书应用**：只需确保它有上述权限即可。

### 2. lark-cli

消息和会议数据拉取依赖 `lark-cli`。

```bash
# 检查是否已安装
lark-cli --version

# 安装（莉莉丝内网）
npm config set @lilith:registry https://registry-cnpm.lilithgame.com
npm install -g @lilith/lark-cli@latest
```

### 3. 飞书 OAuth 授权

lark-cli 需要用户身份授权才能读取数据。`auth login` 采用 Device Flow，生成授权链接/二维码，浏览器打开完成即可。

**交互式授权**（人在终端前）：

```bash
# 基础授权（消息读取）
lark-cli auth login --scope "im:chat:readonly im:message:readonly im:message.reactions:read"

# 会议纪要 + 文档读取授权（推荐）
lark-cli auth login --scope "vc:meeting.search:read vc:meeting.meetingevent:read vc:note:read vc:record:readonly docs:doc:readonly"
```

**Agent 场景非阻塞授权**（agent 拿到链接后发给用户，不卡住等）：

```bash
# 第一步：生成授权链接（不等待）
lark-cli auth login --scope "im:chat:readonly im:message:readonly" --no-wait --json
# → 返回 verification_url + device_code，发给用户浏览器打开

# 第二步：用户确认后，用 device_code 完成授权
lark-cli auth login --device-code <上一步返回的device_code>
```

所有权限均为**只读**。

### 4. AI Agent

兼容多种 agent 框架，任选其一：
- OpenClaw / ArkClaw / 其他 Claw 系
- Claude Code
- Codex
- Cursor
- 其他支持读文件 + 执行 shell 命令的 agent

---

## 初始化配置

### 配置文件

复制 `config.example.json` 为 `daily-digest-config.json`，填入你的信息：

```json
{
  "version": 1,
  "updated": "2026-06-15",
  "window_hours": 26,
  "max_chats_scan": 120,
  "early_stop_empties": 15,
  "blacklist_keywords": [
    "示例：某个你不想看的群名关键词"
  ],
  "sysbot_names": [
    "日历助手", "云文档助手", "账号安全中心", "审批",
    "飞书项目", "飞书 aily", "会议预约", "飞书招聘",
    "飞书人事", "飞书绩效"
  ],
  "my_open_id": "ou_xxxxxxxxxxxxx",
  "bot": {
    "app_id": "cli_xxxxxxxxxxxxx",
    "app_secret": "***"
  }
}
```

**字段说明**：

| 字段 | 说明 | 建议 |
|------|------|------|
| `window_hours` | 扫描时间窗口 | 26（留 2h 重叠） |
| `max_chats_scan` | 最多扫多少个会话 | 120 |
| `early_stop_empties` | 连续 N 个空会话就停 | 15 |
| `blacklist_keywords` | 群名含这些词就跳过（不区分大小写） | 按需添加非工作群 |
| `sysbot_names` | 系统 bot 名，会被跳过 | 飞书功能性 bot |
| `my_open_id` | 你的飞书 open_id（ou_xxx） | 飞书管理后台查看 |
| `bot.app_id` | 飞书应用 App ID | 开放平台获取 |
| `bot.app_secret` | 飞书应用 App Secret | 开放平台获取 |

> ⚠️ **凭证安全**：`app_secret` 是敏感凭证。**不要**放在 skill 目录下、**不要**提交到 Git 仓库、**不要**发到群聊或截图。推荐放在用户级 state 目录或环境变量中。

### 配置文件放哪里？

| 环境 | 建议路径 |
|------|----------|
| OpenClaw / ArkClaw | `~/.openclaw/workspace-<agent>/state/daily-digest-config.json` |
| Claude Code | `~/.claude/config/daily-digest-config.json` |
| Codex | `~/.codex/config/daily-digest-config.json` |
| 通用 | `~/.config/feishu-daily-digest/config.json` |

**不要**把真实配置放在 skill 安装目录下——skill 目录可能被提交到 Git 或随 Atlas 分发。skill 目录只放 `config.example.json` 作为模板。

**关键原则**：配置文件是唯一的个性化来源。skill 正文不硬编码任何人的黑名单、open_id 或凭证。

---

## 消息拉取

```bash
# ① 全量会话，按活跃倒序
lark-cli im +chat-list --types group,p2p --sort-type ByActiveTimeDesc --page-size 100 --as user --format json
# → .data.chats；用 --page-token 翻页直到 has_more=false

# ② 过滤：名字含 blacklist_keywords → 跳过；名字在 sysbot_names → 跳过

# ③ 逐会话拉消息
lark-cli im +chat-messages-list --chat-id <oc_xxx> --start <ISO-26h-ago> --page-size 50 --as user --format json
# 连续 early_stop_empties 个空会话就停
```

### 解析要点（踩过的坑）

- **正文在消息顶层 `content`**，不是 `body.content`
- `sender.name` 自带；`msg_type` 非 text 标 `[image]/[file]/[post]/[interactive]`
- **reactions 内嵌**：`.reactions.details[].operator.operator_id` == 你的 open_id → 你已用 emoji 回应过
- bot 的 `[interactive]` 卡片**折叠成一行计数**，只有含「请跟进/待办」语义的才展开
- 消息默认倒序，分析前按 `create_time` 排正

---

## 会议智能纪要拉取

同窗口内如果有过会议，额外读取飞书 VC 智能纪要。

```bash
# ① 搜索历史会议
lark-cli vc +search --start <YYYY-MM-DD> --end <YYYY-MM-DD> --page-size 20 --as user --format json

# ② 批量获取会议产物
lark-cli vc +notes --meeting-ids '<id1>,<id2>' --as user --format json

# ③ 读取智能纪要正文
lark-cli docs +fetch --api-version v2 --doc <note_doc_token> --doc-format markdown --as user
```

### 选择规则

- 只处理有 `note_doc_token` 的会议；没有智能纪要的忽略
- 智能纪要是 AI 生成，输出里**必须标注「据智能纪要」**
- **每日最多展开 3 场**；更多只列标题 + "另有 N 场"
- 优先级：自己参会 > 需求宣讲 > 纪要中有待办 > 标题含项目关键词
- 权限不足时不要失败退出，在摘要里标注缺哪个 scope

---

## 摘要结构

### 输出模板

```
飞书日摘要 · M/D（近 26h）
　
🔔 需要你跟进
1. ...
　
📌 拍板 / 口径
- ...
　
📝 会议纪要
- **会议名**（时间）：据智能纪要，关键结论... 待办：...
　
💬 其他要点
- ...
　
注：会议纪要来自飞书 AI，关键结论建议回查原纪要。
```

> **飞书空行技巧**：每个大标题前和最后备注前，放一行**全角空格** `　`，飞书不会折叠全角空格行。

### 区块定义

1. **🔔 需要你跟进**：被 @你 且未回应的；别人明确在等你的；你自己立的 flag/承诺。最多 5 条。
   - 已回应 = ① 之后你发过消息回应（含跨会话）；② 消息带你的 emoji reaction
2. **📌 拍板 / 口径**：聊天或会议中达成的结论、推翻的旧口径、影响在写案子/做功能的新决定
3. **📝 会议纪要**：只放结论 / 待办 / 风险，不搬运长段。每场 1~2 行
4. **💬 其他要点**：其余值得知道的，3~6 条

### 长度与美观

- 常规 ≤ 1000 字；有会议纪要 ≤ 1300 字
- 超限先砍 💬，再压缩 📝，**不砍** 🔔
- 没内容的区块省略，不写"无"
- 每条优先写「对象 / 动作 / 影响」
- 同一系统/项目的内容合并

### 严谨纪律

- 错别字/口语/缩写按上下文理解，拿不准就标（待核）
- 智能纪要来自飞书 AI，重要结论需回查原纪要

---

## 投递前校验（fail-closed）

发送摘要前必须校验以下条件，任一不满足则**禁止发送**，只输出到终端并提示错误：

1. `my_open_id` 格式合法（`ou_` 开头）
2. `bot.app_id` 和 `bot.app_secret` 非空
3. 发送目标 == `my_open_id`（禁止发给别人）
4. 配置文件不在 skill 安装目录下（防止随仓库泄露）

## 运行日志

每次执行记录一条最小审计日志，写入配置文件同目录下的 `daily-digest-run.log`：

```json
{"ts":"2026-06-15T08:30:01+08:00","trigger":"cron","window":"26h","chats_scanned":42,"messages_readed":310,"meetings_found":2,"delivered":true,"recipient":"ou_xxx"}
```

日志只记元数据，不记消息内容。

---

## 投递：通过飞书 Bot 发送

摘要通过飞书 Bot API 发到你的 DM。

### 发送方式（按你的环境选一种）

**方式 A：Claw 系（OpenClaw / ArkClaw 等）**

使用 Claw 的 message 工具，走 bot 身份发送。不需要手动获取 tenant_access_token。

**方式 B：直接调用飞书 OpenAPI**

没有 Claw 系的 agent，可以通过飞书 OpenAPI 发送消息。核心步骤：

1. 用 App ID + App Secret 获取 `tenant_access_token`（飞书鉴权接口）
2. 用 token 调用消息发送接口，向你的 `open_id` 发 DM

具体 API 端点和参数请参考 [飞书开放平台消息发送文档](https://open.feishu.cn/document/server-docs/im-v1/message/create)。你的 agent 可以帮你生成完整的调用脚本。

> ⚠️ `app_secret` 是敏感凭证。放在配置文件或环境变量中，不要硬编码到提交公开仓库的脚本里。

> ⚠️ `app_secret` 是敏感凭证。不要硬编码到脚本里提交公开仓库。放在配置文件或环境变量中。

**方式 C：终端输出**

不想自动发送？直接在 agent 聊天窗口看摘要也行，手动复制。

### 安全铁律

- 摘要只发给你本人，不转发、不群发
- 内容是公司内部信息，不进公网、不进 HTML 报告

---

## 定时执行（可选）

想每天自动跑？按你的操作系统选一种：

### macOS（LaunchAgent）

macOS 用户可以使用系统自带的定时任务功能（LaunchAgent）实现每天定时执行：

1. 在 `~/Library/LaunchAgents/` 下创建一个 `.plist` 配置文件，指定触发时间和启动脚本
2. plist 中用 `StartCalendarInterval` 设置时间，`ProgramArguments` 指向你的脚本路径
3. 创建好后注册生效（具体命令请查阅 `man launchctl`）

> 具体 plist 写法和注册步骤请参考 [Apple 开发者文档](https://developer.apple.com/library/archive/documentation/MacOSX/Conceptual/BPSystemStartup/Chapters/CreatingLaunchdJobs.html)或搜索「macOS LaunchAgent tutorial」

### Windows（Task Scheduler）

```powershell
# 每天早上 08:30 执行
$action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-File C:\path\to\run-digest.ps1"
$trigger = New-ScheduledTaskTrigger -Daily -At 8:30AM
Register-ScheduledTask -TaskName "FeishuDailyDigest" -Action $action -Trigger $trigger
```

### Linux（cron）

Linux 用户可以使用系统自带的定时任务功能（如 cron），每天指定时间运行启动脚本。请查阅 `man cron` 或搜索「linux scheduled tasks tutorial」配置定时执行。

### Claw 系定时

OpenClaw / ArkClaw 用户可以用内置 cron 功能，不需要系统级定时：

```
schedule: "30 8 * * *" (Asia/Shanghai)
payload: agentTurn
message: "执行飞书日摘要，严格按 SKILL.md 流程"
```

### run-digest 脚本模板

无论哪种定时方式，核心都是唤起 agent 执行 skill。一个简单的 bash 示例：

```bash
#!/bin/bash
# run-digest.sh — 启动 agent 执行日摘要
# 替换为你的 agent 启动命令，例如：
# claude-code --prompt "执行飞书日摘要，按 ~/.claude/skills/feishu-daily-digest/SKILL.md"
# 或：
# openclaw message send --channel feishu --message "执行飞书日摘要"
```

---

## FAQ

**Q: 第一次用，黑名单怎么配？**
A: 先空着跑一次。看摘要里哪些群的信息你不关心，把群名关键词加进去。

**Q: 会议纪要读不到？**
A: 运行 `lark-cli auth login --scope "vc:meeting.search:read vc:meeting.meetingevent:read vc:note:read vc:record:readonly" --as user` 授权。

**Q: 没有飞书 Bot 怎么办？**
A: 前往 [飞书开放平台](https://open.feishu.cn/) 创建自建应用并开启机器人能力。这是强依赖，没有 Bot 无法自动投递。

**Q: 可以多人共用吗？**
A: 可以。每人各自一份配置文件（不同 open_id、黑名单、凭证）。agent 按当前触发者的配置执行。

**Q: 不想用会议纪要功能？**
A: 跳过 VC 相关步骤，不输出 📝 区块即可。消息摘要不受影响。

**Q: 能不能不发飞书，只看终端？**
A: 可以。让 agent 直接输出摘要到聊天窗口，跳过 Bot 发送步骤。

---

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0.0 | 2026-06-15 | 通用版首发 |
| 1.0.1 | 2026-06-15 | 安全修复：移除交互确认、凭证安全加固、fail-closed 校验、审计日志、修复 auth login 命令、补全 docs 权限 |
| 1.0.2 | 2026-06-15 | changelog 补全 |
