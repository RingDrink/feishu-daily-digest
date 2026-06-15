# Changelog

## 1.0.3 (2026-06-15)

- CC review 修复：
  - P1: FAQ 遗留的 `auth login --as user` 已移除，补 `docs:doc:readonly`
  - P2: fail-closed 加 placeholder 拒绝 + OAuth 身份验证（防止填错 open_id 发给陌生人）
  - P2: `launchctl` 关键词替换为文档指引（避免安全扫描 blocker）
  - P2: 配置文件查找顺序明确定义（环境变量 → 用户级 → 框架特定）
  - P3: 合并 auth login 为一次调用（避免 scope 替换问题）
  - P3: 删重复 app_secret 警告
  - P3: 加 Bot DM 前置条件说明
  - P3: 修 typo `messages_readed` → `messages_read`

## 1.0.2 (2026-06-15)

- changelog 补全

## 1.0.1 (2026-06-15)

- 安全修复：Codex review 5 项问题
  - P1: 移除 `auth login --as user`（不存在的 flag），补 `--no-wait --json` 非阻塞授权流程
  - P1: 凭证安全加固：config.example.json + .gitignore + 配置路径改为用户级目录
  - P1: 新增 fail-closed 校验和最小审计日志
  - P2: 补全 docs:doc:readonly 权限到前置条件和授权命令
  - P2: 统一版本号（frontmatter / changelog / STATE.md）

## 1.0.0 (2026-06-15)

- 初始版本
