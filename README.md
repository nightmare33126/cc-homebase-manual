# cc-homebase-manual（安家 skill）

「安家」是 [Claude 安家手册](https://claude-home-a4laziwf97p.qoder.zone/) 的配套 skill。手册教你在 Mac 和 iPhone 上把 Claude 稳定跑通，这个 skill 让你的 AI 助手替你干其中的技术活。

## 它能干什么

- **清理本机残留**。扫描并清理 Mac 上 Claude / Anthropic 的历史痕迹，换新号之前必做
- **部署服务器**。把一台全新的 Ubuntu 服务器配成个人代理节点，VLESS + Reality，单用户，无面板，全程由 AI 操作，你只负责点头
- **定期体检**。十八条只读检查，出报告，同意后才修

支持任何能读文件、能跑命令的 AI 助手，Qoder、Codex、Claude Code 都行。

## 怎么用

把下面这段话发给你的 AI 助手，按需改掉方括号里的内容。

部署服务器

```text
请下载 https://github.com/nightmare33126/cc-homebase-manual 的 SKILL.md 和 references 目录，读完部署流程，然后部署我的服务器
IP [你的IP]
SSH 端口 [默认22]
用户 root，密码用安全方式问我
别名 myvps
时区 America/Los_Angeles
```

定期体检

```text
请下载 https://github.com/nightmare33126/cc-homebase-manual 的体检清单，检查我的服务器，ssh myvps，只读不改，出报告
```

清理本机残留

```text
请下载 https://github.com/nightmare33126/cc-homebase-manual 的本机扫描清单，扫一遍我这台 Mac 上 Claude 的残留，先报告，我同意再删
```

## 结构

- `SKILL.md` 总原则和三套流程入口
- `references/local-scan.md` 本机残留扫描与清理
- `references/server-setup.md` 服务器部署五阶段与验收
- `references/health-check.md` 十八条体检清单与报告格式

## 安全原则

改动先说明、删除先备份、凭据不上屏、单用户不共享。详见 SKILL.md。

## 免责

本仓库内容仅供学习研究，请遵守你所在地的法律法规以及各平台用户协议。维护者完全匿名，不提供任何付费服务，遇到收费代装的都是骗子。

## 协议

MIT
