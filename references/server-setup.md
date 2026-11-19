# 服务器部署流程（VLESS + Reality，单用户，无面板）

目标：把一台刚装好 Ubuntu 22.04 或 24.04 的全新服务器，配置成只服务主人一个人的代理节点。整个方案不装管理面板、不签域名证书、不开多用户，一切配置都是文件，出了问题看得见摸得着。

前置输入，向用户要，root 密码用安全输入方式收，不要让用户发在聊天里：

- 服务器 IP
- SSH 端口（默认 22）
- 服务器别名（默认 myvps）
- 时区（默认 America/Los_Angeles）

开始前先做预检，任何一条不过就停下来向用户报告，不要绕过：

- `lsb_release -a` 确认是 Ubuntu 22.04 或 24.04
- `ss -tlnp` 确认 443 端口没有被占用
- `systemctl list-units --type=service --state=running` 确认没有已经在跑的 xray、nginx、caddy、3x-ui 之类的服务。有就停，问用户这台服务器是不是全新空机

## 阶段一，基础加固

1. `apt update && apt upgrade -y`
2. `timedatectl set-timezone <时区>`
3. SSH 改密钥登录。先问用户要公钥（或者现场用 `ssh-keygen -t ed25519` 在用户本机生成一对，公钥上来，私钥留在本机）。把公钥追加到服务器 `/root/.ssh/authorized_keys`，然后用新开一个终端验证密钥能登录。验证成功之后才改 `/etc/ssh/sshd_config`，`PasswordAuthentication no`，`systemctl restart sshd`。顺序不能反，反了会把人锁在门外
4. 装防火墙。`apt install -y ufw`，`ufw allow <SSH端口>/tcp`，先别 enable，等 443 开好再一起

## 阶段二，装 Xray 内核

用 Xray 官方安装脚本，不要用第三方一键包：

```bash
bash -c "$(curl -L https://github.com/XTLS/Xray-install/raw/main/install-release.sh)" @ install
```

装完 `xray version` 确认版本号，并确认 `systemctl status xray` 存在（此时未启动是正常的）。

## 阶段三，生成节点

1. 生成密钥对。`xray x25519`，输出 Private key 和 Public key。私钥进服务器配置，公钥进客户端配置
2. 生成用户标识。`xray uuid` 得到 UUID，`openssl rand -hex 8` 得到 shortId
3. 选伪装目标。默认用 `www.microsoft.com:443`。先在服务器上 `curl -sI https://www.microsoft.com --max-time 5` 确认可达且证书正常，不通就换 `www.apple.com`、`www.amazon.com` 这一档大流量官网再验证
4. 写 `/usr/local/etc/xray/config.json`，最小可用配置，一个入站：

```json
{
  "inbounds": [
    {
      "listen": "0.0.0.0",
      "port": 443,
      "protocol": "vless",
      "settings": {
        "clients": [
          { "id": "<UUID>", "flow": "xtls-rprx-vision" }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "tcp",
        "security": "reality",
        "realitySettings": {
          "dest": "<伪装目标>:443",
          "serverNames": ["<伪装目标域名>"],
          "privateKey": "<私钥>",
          "shortIds": ["<shortId>"]
        }
      }
    }
  ],
  "outbounds": [
    { "protocol": "freedom" },
    { "protocol": "blackhole", "tag": "block" }
  ]
}
```

5. `xray -test -config /usr/local/etc/xray/config.json` 校验通过后，`systemctl enable --now xray`，然后 `systemctl status xray` 确认 active
6. `ufw allow 443/tcp`，`ufw --force enable`，`ufw status verbose` 确认 SSH 端口和 443 都在放行列表里

## 阶段四，系统调优

开 BBR：

```bash
cat >> /etc/sysctl.conf << 'EOF'
net.core.default_qdisc=fq
net.ipv4.tcp_congestion_control=bbr
EOF
sysctl -p
```

`sysctl net.ipv4.tcp_congestion_control` 确认输出 bbr。

## 阶段五，生成客户端配置并交付

1. 在用户本机（不是服务器）拼出节点链接，格式

```text
vless://<UUID>@<服务器IP>:443?encryption=none&flow=xtls-rprx-vision&security=reality&sni=<伪装域名>&fp=chrome&pbk=<公钥>&sid=<shortId>&type=tcp#<别名>-Reality
```

2. 写一份 Clash Verge 能直接导入的本地配置文件，proxies 用同一个节点（type vless，udp 开着），rules 第一组是国内域名和 GEOIP CN 直连，其余全部走节点，DNS 里国外用 1.1.1.1 和 8.8.8.8，国内用 223.5.5.5，fake-ip 模式
3. 两样东西加上服务器备忘（IP、端口、伪装域名、到期日提醒，不放 UUID 和密钥）一起写进用户桌面的「安家-<别名>-配置.md」。文件里注明哪些行是机密，截图分享前必须删掉
4. 提醒用户，Clash Verge 用「导入本地文件」方式导入配置，iPhone 的 Shadowrocket 直接粘贴节点链接

## 验收清单

部署完逐项跑，全部通过才算完成，用一句「九项全过，可以交付」收尾：

1. `systemctl is-active xray` 为 active
2. `systemctl is-enabled xray` 为 enabled
3. `ss -tlnp | grep :443` 看到xray在监听
4. `ufw status` 为 active，且放行了 SSH 端口和 443
5. `sshd -T | grep passwordauthentication` 为 no
6. 新开终端用密钥能免密登录
7. `sysctl net.ipv4.tcp_congestion_control` 为 bbr
8. 配置已备份到 `/root/anjia-backup/` 一份
9. 用户桌面存在「安家-<别名>-配置.md」

## 交付后建议

提醒用户去跑手册第 5 步的网页体检（出口 IP、DNS、WebRTC、信任评分），全部达标后再去注册账号。

## 纯手动用户

不用 AI 的用户按上面五个阶段的顺序自己敲即可，命令都在，注意三件事，改 sshd_config 前先验证密钥能登录、config.json 里的尖括号占位符全部替换干净、交付文件不要发给别人。
