# 服务器定期体检（全程只读）

目标：不动任何东西，把服务器的健康状态查一遍，输出报告。有问题的列出来，用户同意后再修，修完重新体检一次。

什么时候做：每隔一两个月例行做一次；感觉变慢、连不上时做；出差前做。

## 检查清单

连接方式用 ssh 密钥（本方案部署完成后不再用密码登录）。

1. `systemctl is-active xray`，期望 active
2. `systemctl is-enabled xray`，期望 enabled
3. `xray -test -config /usr/local/etc/xray/config.json`，期望通过
4. `ss -tlnp` 里 443 端口在监听且归属 xray
5. `ufw status verbose`，期望 active 且放行 SSH 端口与 443，没有多出来的陌生放行
6. `sshd -T | grep -E 'passwordauthentication|permitrootlogin'`，期望密码登录已关、root 只能密钥
7. `/root/.ssh/authorized_keys` 里的公钥数量，期望只有用户自己的（多出来的立刻报告，这是安全事件）
8. `last -n 20` 最近登录记录，有陌生 IP 立刻报告
9. `sysctl net.ipv4.tcp_congestion_control`，期望 bbr
10. `timedatectl` 时区正确、时间同步正常
11. `apt list --upgradable 2>/dev/null | wc -l`，待更新补丁超过 30 个建议安排更新（更新需用户同意）
12. `df -h /` 根分区用量，超过 85% 报告
13. `free -m` 内存，可用低于 200M 报告
14. `/root/anjia-backup/` 配置备份存在，且和当前配置一致，不一致说明配置改过没备份，报告
15. `journalctl -u xray --since "7 days ago" | grep -ci error`，错误超过 10 次报告
16. 在服务器上 `curl -sI https://<伪装域名> --max-time 5`，伪装目标仍然可达
17. 服务器能正常解析域名，`curl -s https://www.google.com -o /dev/null -w '%{http_code}'` 出网正常
18. 提醒用户在本机浏览器复测手册第 5 步的网页体检，出口 IP、DNS、WebRTC、评分四项

## 报告格式

```text
安家体检报告

十八项逐项列结果，每项一行，通过的标「通过」，没过的标「待处理」。
全部通过的话附一句，服务器状态健康，安心用。

有待处理项的话，每项写清
- <项目名>，没过的原因，一句话
- 建议怎么修，改什么、有什么影响
等用户点头再动手。修完从头再查一遍清单。
```

## 修复类操作的安全要求

- 修配置前先把当前配置复制进 `/root/anjia-backup/`，带日期后缀
- 涉及重启 xray、改防火墙、改 SSH 的操作，一句话说清影响再执行
- 修完必须复跑整个清单，不能只查修过的那项
