# 第七课：进阶架构与毕业项目

学会了单机和服务器通信之后，再引入复杂组件。

## WireGuard VPN

WireGuard 是一种基于 UDP 的加密隧道。你需要区分 Peer、公私钥、AllowedIPs、IP 转发、NAT 与路由。客户端“连接成功”并不意味着所有网站都能访问：还要验证 DNS、服务器出口和防火墙。

## 正向代理和反向代理

正向代理代表客户端访问外部服务；反向代理如 Caddy、Nginx 位于网站服务前，为站点处理 HTTPS 和请求转发。它们面向的服务对象不同。

## Docker 与容器

Docker 便于打包和运行服务，却不能自动解决端口映射、DNS 解析、TLS 证书、安全组和数据持久化。

## 毕业实践

在本人拥有权限的设备与测试服务器上，完成：

1. 画出 Android / Windows → 本地网关 → VPS → 测试站点的链路拓扑。
2. 使用 SSH 密钥管理服务器，部署 HTTPS 测试站点。
3. 在客户端分别测试直连、系统代理和 TUN；说明区别。
4. 记录 DNS、端口、握手、HTTP 响应和日志检查结果。
5. 写出安全检查表：密钥保护、最小端口开放、更新和备份。

## 延伸阅读

- [MDN 网络基础](https://developer.mozilla.org/zh-CN/docs/Learn_web_development/Howto/Web_mechanics)
- [WireGuard 官方](https://www.wireguard.com/quickstart/)
- [Caddy 文档](https://caddyserver.com/docs/)
- [Mihomo 文档](https://wiki.metacubex.one/)

[返回课程首页](course/00-roadmap.md)
