# 07｜第一个全链路实验：SSH 动态端口转发变成 SOCKS5

**本课目标**：自己把“本地客户端 → 加密通道 → 自有服务器 → 目标网站”连起来，明确每一步发生了什么。

与直接部署复杂代理软件相比，SSH 动态端口转发特别适合学习，因为你已经在第 06 课验证了 SSH 登录。它适合个人学习和授权的网络访问，不是大规模或生产环境的默认部署方案。

## 前提

你有自己管理的 Linux VPS、已通过 SSH 公钥登录，并且服务器可以访问 `https://example.com`。确保云安全组只放行你需要的 SSH 访问；**不需要在服务器放行 1080 入站端口**。

## 第一步：建立本地 SOCKS5 入口

在**你自己的 Windows PowerShell 或 Linux/macOS 终端**执行：

```bash
ssh -N -D 127.0.0.1:1080 -o ExitOnForwardFailure=yes ubuntu@你的服务器公网IP
```

- `ssh`：建立经过身份验证的加密 SSH 连接。
- `-D 127.0.0.1:1080`：让**本机**监听 SOCKS5 入口 1080。
- `-N`：只做转发，不打开远程 shell。
- `ExitOnForwardFailure=yes`：如果本地端口没监听成功，就退出而非假装成功。

命令保持运行时，另开一个终端窗口做验证。这里的“本地 1080”是你电脑的，不是云服务器需要对互联网开放的服务端口。

## 第二步：证明两条路径不同

在**本地另一个终端**执行：

```bash
curl -I https://example.com
curl --socks5-hostname 127.0.0.1:1080 -I https://example.com
```

Windows PowerShell 建议把 `curl` 写为 `curl.exe`，避免别名或版本差异。

第一个请求按当前系统/环境变量网络设置运行，第二个请求明确送往本机 SSH SOCKS5 入口。`--socks5-hostname` 允许让 SOCKS 服务器侧处理主机名；与 `--socks5` 的本地解析路径不同。返回 HTTP 状态码说明 HTTPS 请求到了某个 HTTP 服务，但不能单靠状态码证明出口 IP 必然改变。

**验证出口 IP**：可额外访问你信任的 IP 回显网站，比对直连与 SOCKS 请求；注意不同 IP 回显服务、IPv4/IPv6、CDN 与缓存可能产生差异。

## 第三步：真正画出路径

```text
本机 curl
  → 127.0.0.1:1080 (ssh 客户端的 SOCKS5 入口)
  → 本机 SSH 加密 TCP 连接
  → VPS 上的 sshd
  → VPS 向 example.com:443 建立连接
  → 网站 TLS/HTTP 响应按原路返回
```

SSH 的外层传输通常使用 TCP 22（也可改端口），SOCKS5 是本机应用与 SSH 客户端交互的协议；HTTPS 仍然可以端到端验证目标站证书。**两个独立层次的加密不要混为一谈**。

## 第四步：如何安全结束和排错

在运行 SSH 的终端按 `Ctrl+C` 即关闭转发。不要把本地监听地址改为 `0.0.0.0` 再开放给陌生人，否则可能形成未授权可用的代理入口。

| 现象 | 首先排查 |
| --- | --- |
| ssh 连接超时 | 公网地址、云安全组、TCP SSH 端口、网络 |
| Permission denied (publickey) | 用户名、公钥授权、私钥路径 |
| bind: Address already in use | 本机 1080 已被占用，换一个本地端口 |
| SOCKS 请求失败，但 SSH 已连接 | 服务端出站、目标 DNS、SSH 转发策略 |
| curl 返回证书错误 | 不要关闭校验；查目标域名、系统时间、TLS 链 |

**课后题**：在这个实验中，哪台机器真正向目标站点发起 TCP 连接？答：远端 VPS（由其 SSH 服务进程代为建立）。

下一课：[WireGuard：从代理进入 IP 层 VPN](course/08-wireguard.md)。
