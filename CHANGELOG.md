# Changelog

## 1.0.2 (2026-06-15)

- 安全修复：Codex review 5 项问题
  - P1: 移除 `auth login --as user`（不存在的 flag），补 `--no-wait --json` 非阻塞授权流程
  - P1: 凭证安全加固：config.example.json + .gitignore + 配置路径改为用户级目录
  - P1: 新增 fail-closed 校验和最小审计日志
  - P2: 补全 docs:doc:readonly 权限到前置条件和授权命令
  - P2: 统一版本号（frontmatter / changelog / STATE.md）

## 1.0.1 (2026-06-15)

- 通用版首发
- 飞书 Bot 作为强依赖
- 跨平台定时（macOS / Windows / Linux / Claw cron）
- 兼容多 Claw（OpenClaw / ArkClaw）
- 砍掉交互确认，纯信息展示
- 通过 Atlas Skill Hub 安全扫描（MEDIUM）

## 1.0.0 (2026-06-15)

- 初始版本
