# 本机残留扫描与清理（Mac 为主）

目标：把这台电脑上所有跟 Claude / Anthropic 有关的本地痕迹找出来，经用户同意后备份并删除。适用于换新号之前，或者旧号出事之后重新开始。

## 扫描清单

逐项检查存在性，存在的记录路径、大小、最后修改时间。不要输出任何文件内容。

1. `~/.claude` 目录（Claude Code 的配置、会话、凭据）
2. `~/Library/Application Support/Claude`（桌面客户端数据）
3. `~/Library/Application Support` 下名字含 anthropic 的其他目录
4. `~/Library/Preferences/com.anthropic.*.plist`
5. `~/Library/Preferences/ByHost/com.anthropic.claudefordesktop.*.plist`
6. `~/Library/Caches/` 下 com.anthropic 开头的目录
7. `~/Library/Logs/` 下 Claude 或 anthropic 相关日志
8. 钥匙串。用 `security dump-keychain | grep -i anthropic` 只列条目名，不导出内容。常见条目包括 Claude Safe Storage、Claude Code-credentials 一类
9. 浏览器的原生消息组件。`~/Library/Application Support/Google/Chrome/NativeMessagingHosts/` 和 Chromium、Edge、Brave 对应目录下名字以 com.anthropic 开头的文件
10. 最近文档记录。`~/Library/Application Support/com.apple.sharedfilelist/com.apple.LSSharedFileList.ApplicationRecentDocuments/` 下 com.anthropic 开头的 sfl 文件
11. 已安装的 Claude 桌面 App 本体（/Applications/Claude.app）

## 汇报格式

扫描完先给一张表，每一项写清楚是什么、在不在、删了会影响什么。用户确认哪些要删，再进清理阶段。

## 清理流程

1. 在桌面建备份目录 `~/Desktop/安家备份-<日期>/`
2. 把用户确认要删的每一项复制进备份目录，保持原路径结构
3. 复制完成且校验大小后，删除原位置的文件
4. 钥匙串条目用 `security delete-generic-password -s <条目名>` 逐条删，同样先记录条目名到备份目录的备忘文件
5. 清空废纸篓这一步留给用户自己决定，不代劳
6. 结束时重新跑一遍扫描清单，逐项报结果，全部干净才算完

## 浏览器侧的建议（不代做，只提醒）

- 平时常用的浏览器 Profile 不要登录 Claude
- 单独建一个干净的 Profile（比如叫 Claude 专用），以后所有 Claude 相关操作都在里面
- 装过 Claude 相关浏览器插件的，在插件管理页移除

## Windows 参考路径

`%USERPROFILE%\.claude`、`%APPDATA%\Claude`、`%LOCALAPPDATA%\AnthropicClaude`、`%LOCALAPPDATA%\` 下 com.anthropic 开头的目录、凭据管理器里的 Claude 条目。流程同上，先扫描汇报，同意后备份再删。
